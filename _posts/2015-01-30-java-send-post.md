---
layout: post
title: Java发送Post请求
date: 2015-01-30 15:17:12
tags: Java
categories: code
---
Java可以很方便地实现发送Post请求，并在请求前转码。如下
<!-- more -->
``` java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;

public class Main {
	static String req = "test post";
	public static String sendPost(String url, String param) {
        PrintWriter out = null;
        BufferedReader in = null;
        String result = "";
        try {
            URL realUrl = new URL(url);

            URLConnection conn = realUrl.openConnection();

            conn.setRequestProperty("accept", "*/*");
            conn.setRequestProperty("connection", "Keep-Alive");

            conn.setDoOutput(true);
            conn.setDoInput(true);
            
            byte[] utf8 = param.getBytes("utf8");
            
            OutputStream os = conn.getOutputStream();
            os.write(utf8);

            in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            String line;
            while ((line = in.readLine()) != null) {
                result += line;
            }
        } catch (Exception e) {
            System.out.println("send post error!"+e);
            e.printStackTrace();
        }

        finally{
            try{
                if(out!=null){
                    out.close();
                }
                if(in!=null){
                    in.close();
                }
            }
            catch(IOException ex){
                ex.printStackTrace();
            }
        }
        return result;
    }
	
	public static void main(String[] args) throws Exception{
		System.out.println("start");
		String res = sendPost("http://localhost:8080/test", req);
		System.out.println("stop");
		System.out.println(res);
	}

}
```