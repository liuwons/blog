---
layout: post
title: 使用OpenCV Android SDK从摄像头帧实时检测人脸
date: 2016-06-08 10:22:56
categories: code
tags: [Android]
---

在配置好 **OpenCV Android SDK** 之后(具体见前一篇文章[Android Studio中使用OpenCV Android SDK](http://blog.lwons.com/archieve/use_opencv_in_android.html))，可以使用 **OpenCV** 封装的接口很方便地进行各种图像处理操作。

这里简单介绍如何直接使用 **OpenCV** 训练的人脸模型直接从摄像头帧检测人脸。

### 1. 新建Android Project

这里可以直接使用默认的 **Android Studio** 项目模板， **Activity** 选择 ```Empty Activity``` 。

### 2. 配置OpenCV Android SDK

参考前一篇文章: [Android Studio中使用OpenCV Android SDK](http://blog.lwons.com/archieve/use_opencv_in_android.html)

### 3. 向 ```AndroidManifest.xml``` 中添加 **Camera** 相关的 Permission

在 ```AndroidManifest.xml``` 文件 ```<application>``` 节点前添加如下代码：

```
<uses-permission android:name="android.permission.CAMERA"/>

<uses-feature android:name="android.hardware.camera" android:required="false"/>
<uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
<uses-feature android:name="android.hardware.camera.front" android:required="false"/>
<uses-feature android:name="android.hardware.camera.front.autofocus" android:required="false"/>
```

### 4. 添加OpenCV训练的人脸模型

将 **OpenCV Android SDK** 中 ```sdk/etc``` 目录下的 ```lbpcascade_frontalface.xml``` 文件复制到项目 ```app/src/main/res/raw``` 目录下。

### 5. 修改 ```MainActivity.java``` 的代码

修改 ```MainActivity.java``` 的代码为:

```
public class MainActivity extends AppCompatActivity
        implements CameraBridgeViewBase.CvCameraViewListener {


    private CameraBridgeViewBase openCvCameraView;
    private CascadeClassifier cascadeClassifier;
    private Mat grayscaleImage;
    private int absoluteFaceSize;


    private BaseLoaderCallback mLoaderCallback = new BaseLoaderCallback(this) {
        @Override
        public void onManagerConnected(int status) {
            switch (status) {
                case LoaderCallbackInterface.SUCCESS:
                    initializeOpenCVDependencies();
                    break;
                default:
                    super.onManagerConnected(status);
                    break;
            }
        }
    };


    private void initializeOpenCVDependencies() {


        try {
            // Copy the resource into a temp file so OpenCV can load it
            InputStream is = getResources().openRawResource(R.raw.lbpcascade_frontalface);
            File cascadeDir = getDir("cascade", Context.MODE_PRIVATE);
            File mCascadeFile = new File(cascadeDir, "lbpcascade_frontalface.xml");
            FileOutputStream os = new FileOutputStream(mCascadeFile);


            byte[] buffer = new byte[4096];
            int bytesRead;
            while ((bytesRead = is.read(buffer)) != -1) {
                os.write(buffer, 0, bytesRead);
            }
            is.close();
            os.close();


            // Load the cascade classifier
            cascadeClassifier = new CascadeClassifier(mCascadeFile.getAbsolutePath());
        } catch (Exception e) {
            Log.e("OpenCVActivity", "Error loading cascade", e);
        }


        // And we are ready to go
        openCvCameraView.enableView();
    }


    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);


        getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);


        openCvCameraView = new JavaCameraView(this, -1);
        setContentView(openCvCameraView);
        openCvCameraView.setCvCameraViewListener(this);
    }


    @Override
    public void onCameraViewStarted(int width, int height) {
        grayscaleImage = new Mat(height, width, CvType.CV_8UC4);


        // The faces will be a 20% of the height of the screen
        absoluteFaceSize = (int) (height * 0.2);
    }


    @Override
    public void onCameraViewStopped() {
    }


    @Override
    public Mat onCameraFrame(Mat aInputFrame) {
        // Create a grayscale image
        Imgproc.cvtColor(aInputFrame, grayscaleImage, Imgproc.COLOR_RGBA2RGB);


        MatOfRect faces = new MatOfRect();


        // Use the classifier to detect faces
        if (cascadeClassifier != null) {
            cascadeClassifier.detectMultiScale(grayscaleImage, faces, 1.1, 2, 2,
                    new Size(absoluteFaceSize, absoluteFaceSize), new Size());
        }


        // If there are any faces found, draw a rectangle around it
        Rect[] facesArray = faces.toArray();
        for (int i = 0; i <facesArray.length; i++)
            Core.rectangle(aInputFrame, facesArray[i].tl(), facesArray[i].br(), new Scalar(0, 255, 0, 255), 3);


        return aInputFrame;
    }


    @Override
    public void onResume() {
        super.onResume();

        if (!OpenCVLoader.initDebug()) {
            Log.e("log_wons", "OpenCV init error");
            // Handle initialization error
        }
        initializeOpenCVDependencies();
        //OpenCVLoader.initAsync(OpenCVLoader.OPENCV_VERSION_2_4_6, this, mLoaderCallback);
    }
}
```

### 6. 编译程序，运行

编译之后运行程序，程序会获取手机上的默认相机(一般为后置摄像头)并进行人脸检测。

效果：

![人脸检测效果](http://img.blog.csdn.net/20160608220911203)


### 参考资料

[知乎日报: Android相机开发那些坑](http://daily.zhihu.com/story/7989477)
[Android Developers: Camera](https://developer.android.com/guide/topics/media/camera.html)
[OpenCV Documentation: OpenCV4Android SDK](http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/O4A_SDK.html)
