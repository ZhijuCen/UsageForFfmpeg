
# Usage for ffmpeg

## Compress video with libx265 encoder

### Requirements

Make sure the installed ffmpeg supports libx265.

```sh
ffmpeg -codecs | grep hevc
```

```powershell
ffmpeg.exe -codecs | Select-String hevc
```

And you should see the output like below:

```console
DEV.L. hevc                 H.265 / HEVC (High Efficiency Video Coding) (decoders: hevc hevc_qsv hevc_cuvid )
(encoders: libx265 hevc_amf hevc_mf hevc_nvenc hevc_qsv )
```

### Example

```sh
SRC_VIDEO = "foo.mp4"
DST_VIDEO = "bar.mp4"
CRF_VALUE = 28  # default in hevc, grater value, smaller output size.
PRESET = "medium"  # choices: `veryfast` `faster` `fast` `medium` `slow` `slower` `veryslow`
ffmpeg -i $SRC_VIDEO -c:v libx265 -preset $PRESET -crf $CRF_VALUE -c:a copy -y $DST_VIDEO
```

## Change resolution of video

### Example

```sh
SRC_VIDEO = "foo.mp4"
DST_VIDEO = "bar.mp4"
WIDTH = 1920
HEIGHT = 1080
ffmpeg -i $SRC_VIDEO -s ${WIDTH}x${HEIGHT} -y $DST_VIDEO  # the character `x` is between width and height
```

## Channel mapping

[Docs of -map option](https://ffmpeg.org/ffmpeg-all.html#Advanced-options)

### Example

```sh
SRC_VIDEO_A = "foo.mp4"
SRC_VIDEO_B = "bar.mp4"
DST_VIDEO = "baz.mp4"
# A new video file contains video of video_a and audio of video_b, both of which have same duration.
ffmpeg -i $SRC_VIDEO_A -i $SRC_VIDEO_B \
-map 0:v -c:v libx265 \
-map 1:a -c:a copy \
$DST_VIDEO
```

## Concatenating video clips

[StackExchange Concatenate clips](https://video.stackexchange.com/questions/10396/how-to-concatenate-clips-from-the-same-video-with-ffmpeg)

[StackExchange Slow Fast Motion](https://video.stackexchange.com/questions/21800/ffmpeg-slow-fast-motion-part-a-video-anywhere)

```sh
# Repeat a clip multiple times with fast motion speed.
SRC_VIDEO="foo.mp4"
DST_VIDEO="bar.mp4"
BEGIN=15.8
END=16  # must be grater than $BEGIN, and `Second` is the only time unit.
N=20
motion_speed=2.  # Must be in range(0.5..100) so that atempo can process.
fc_head=""
fc_body=""
for i in `seq 0 $($N-1)`
do
fc_head="${fc_head}
[0:v] trim=${BEGIN}:${END},setpts=$(1 / $motion_speed)*(PTS-STARTPTS)   [v${i}];
[0:a] atrim=${BEGIN}:${END},asetpts=PTS-STARTPTS,atempo=${motion_speed} [a${i}];
"
fc_body="${fc_body}[v${i}][a${i}]"
done
fc="${fc_head}${fc_body}concat=${N}:v=1:a=1[out]"

ffmpeg -i $SRC_VIDEO -filter_complex=$fc -map "[out]" -y $DST_VIDEO
```
