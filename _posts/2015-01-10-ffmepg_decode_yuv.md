---
layout: post
title: ffmpeg解码视频流并转YUV为RGB
date: 2015-01-10 10:12:29
categories: code
tags: ffmpeg
---
FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。采用LGPL或GPL许可证。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多codec都是从头开发的。
<!--more-->
以下代码在Qt环境下用ffmpeg解码视频流并将YUV转换为RGB格式。
``` cpp
int MapThread::decode_write_frame(AVCodecContext *avctx, AVFrame *frame, AVPacket *pkt)
{
    int len, got_frame;
    len = avcodec_decode_video2(avctx, frame, &got_frame, pkt);

    if (got_frame)
    {

        if(image == 0)
            image = new QImage(img_buf, avctx->width, avctx->height, QImage::Format_RGB888);

        received_frame_width = avctx->width;
        received_frame_height = avctx->height;

        for(int h = 0; h < avctx->height; h++)
        {
            for(int w = 0; w < avctx->width; w ++)
            {
                int hh = h >> 1;
                int ww = w >> 1;
                int Y = frame->data[0][h * frame->linesize[0] + w];
                int U = frame->data[1][hh * (frame->linesize[1]) + ww];
                int V = frame->data[2][hh * (frame->linesize[2]) + ww];

                int C = Y - 16;
                int D = U - 128;
                int E = V - 128;

                int r = 298 * C           + 409 * E + 128;
                int g = 298 * C - 100 * D - 208 * E + 128;
                int b = 298 * C + 516 * D           + 128;

                r = qBound(0, r >> 8, 255);
                g = qBound(0, g >> 8, 255);
                b = qBound(0, b >> 8, 255);

                QRgb rgb = qRgb(r, g, b);
                image->setPixel(QPoint(w, h), rgb);
            }
        }
        emit frameGot(image);
    }
    if (pkt->data) {
        pkt->size -= len;
        pkt->data += len;
    }

    return 0;
}

void MapThread::newData()
{
    if(!inited)
    {
        avcodec_register_all();

        pkt = new AVPacket;
        av_init_packet(pkt);

        memset(inbuf + INBUF_SIZE, 0, FF_INPUT_BUFFER_PADDING_SIZE);

        codec = avcodec_find_decoder(AV_CODEC_ID_MPEG1VIDEO);
        if (!codec)
        {
            exit(1);
        }
        c = avcodec_alloc_context3(codec);

        if (!c) {
            exit(1);
        }

        av_opt_set(c->priv_data, "preset", "superfast", 0);
        av_opt_set(c->priv_data, "tune", "zerolatency", 0);

        c->delay = 0;

        if(codec->capabilities&CODEC_CAP_TRUNCATED)
            c->flags|= CODEC_FLAG_TRUNCATED;
        if (avcodec_open2(c, codec, NULL) < 0) {
            exit(1);
        }

        frame = av_frame_alloc();
        if (!frame) {
            exit(1);
        }

        inited = true;
    }
    while(true)
    {
        int nread = mapSocket->read((char*)inbuf, INBUF_SIZE);

        if(nread <= 0)
            break;

        av_init_packet(pkt);
        pkt->size = nread;
        pkt->data = inbuf;
        while (pkt->size > 0)
        {
            if (decode_write_frame(c, frame, pkt) < 0)
                exit(1);
        }
        av_free_packet(pkt);
    }
}
```
