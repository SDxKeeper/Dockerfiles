FFmpeg is a set of open source tools for audio and video processing, such as creating, converting/transcoding, and publishing media content.

### Audio/Video Codecs

The FFmpeg docker images are compiled with the following audio and video codecs:

| Codec | Version | Codec | Version |
|-------|:-------:|-------|:-------:|
|fdk-acc|0.1.6|x265|2.9|
|mp3lame|3.100|vpx|1.7.0|
|opus|1.2.1|aom|1.0.0|
|ogg|1.3.3|SVT-HEVC|1.3.0|
|vorbis|1.3.6|SVT-AV1|custom|
|x264|stable|SVT-VP9|custom|


### Patches

The FFmpeg builds included the following patches for feature enhancement, better performance or bug fixes:

| Patch | Description |
|-------|-------------|
|[11625](https://patchwork.ffmpeg.org/patch/11625/raw)|Enhance 1:N transcoding performance.|
|[11035](https://patchwork.ffmpeg.org/patch/11035/raw)|Fix libvpx to run on Intel(R) Xeon(R) processors.|
|[H.265 FLV](https://raw.githubusercontent.com/VCDP/CDN/master/The-RTMP-protocol-extensions-for-H.265-HEVC.patch)|Support H.265 in FLV for RTMP streaming.|
|[IE_FILTERS_01](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0001-Intel-inference-engine-detection-filter.patch)|Intel inference engine detection filter.|
|[IE_FILTERS_02](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0002-New-filter-to-do-inference-classify.patch)|New filter to do inference classify.|
|[IE_FILTERS_03](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0003-iemetadata-convertor-muxer.patch)|IE metadata convertor muxer.|
|[IE_FILTERS_04](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0004-Kafka-protocol-producer.patch)|Kafka protocol producer.|
|[IE_FILTERS_05](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0005-Support-object-detection-and-featured-face-identific.patch)|Support object detection and featured face identification.|
|[IE_FILTERS_06](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0006-Send-metadata-in-a-packet-and-refine-the-json-format.patch)|Send metadata in a packet and refine the json format.|
|[IE_FILTERS_07](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0007-Refine-features-of-IE-filters.patch)|Refine features of IE filters.|
|[IE_FILTERS_08](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0008-fixed-extra-comma-in-iemetadata.patch)|Fixed extra comma in iemetadata.|
|[IE_FILTERS_09](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0009-add-source-as-option-source-url-calculate-nano-times.patch)|Add source as option source url calculate nano times.|
|[IE_FILTERS_10](https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/media-analytics/0010-fixed-buffer-overflow-issue-in-iemetadata.patch)|Fixed buffer overflow issue in iemetadata.|

### GPU Acceleration

In GPU images, the FFmpeg docker images are accelerated through vaapi and/or qsv (Intel Media SDK).

### FFmpeg Examples:

Transcode raw yuv420 content to SVT-HEVC and mp4:

```bash
ffmpeg -f rawvideo -vcodec rawvideo -s 320x240 -r 30 -pix_fmt yuv420p -i test.yuv -c:v libsvt_hevc -y test.mp4
```

1:N Transcoding:

```bash
ffmpeg -i input.h264 -vf "scale=1280:720" -pix_fmt nv12 -f null /dev/null -vf "scale=720:480" -pix_fmt nv12 -f null /dev/null -abr_pipeline
```

Encoding/decoding with vaapi:

```bash
ffmpeg -y -vaapi_device /dev/dri/renderD128 -f rawvideo -video_size 320x240 -r 30 -i test.yuv -vf 'format=nv12, hwupload' -c:v h264_vaapi -y test.mp4
ffmpeg -hwaccel vaapi -hwaccel_device /dev/dri/renderD128 -i test.mp4 -f null /dev/null
```

Encoding/decoding with qsv (Intel Media SDK):

```bash
ffmpeg -y -init_hw_device qsv=hw -filter_hw_device hw -f rawvideo -pix_fmt yuv420p -s:v 320x240 -i test.yuv -vf hwupload=extra_hw_frames=64,format=qsv -c:v h264_qsv -b:v 5M test.mp4
ffmpeg -hwaccel qsv -c:v h264_qsv -i test.mp4 -f null /dev/null
```

Face detection and emotion identification, save metadata to json format:

```bash
ffmpeg -i ~/Videos/xxx.mp4 -vf detect=model=./face-detection-adas-0001/FP32/face-detection-adas-0001.xml:name=face, \
classify=model=./emotions_recognition/emotions-recognition-retail-0003.xml:label=./emotions_recognition/emotion-labels.txt:name=emotion \
-an -f iemetadata -source_url $URL -custom_tag $TAG emotion-meta.json
```

Object Detection with labels:

```bash
ffmpeg -i ~/Videos/xxx.mp4 -vf detect=model=./mobilenet-ssd.xml:label=./object_labels.txt:name=objects -an -f null /dev/null
```

Face detection and reidentification:

```bash
ffmpeg -i ~/Videos/xxx.mp4 -vf detect=model=./face-detection-retail-0004.xml:name=face, \
classify=model=./face-reidentification-retail-0095.xml:label=./labels.txt:name=face_id:feature_file=./registered_faces.bin -an -f null /dev/nul
```

