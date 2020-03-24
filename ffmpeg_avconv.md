## Table of contents

- [Conversion](#conversion)
  - [Audio file](#audio-file)
  - [Video+audio](#videoaudio)
  - [Video to animated GIF](#video-to-animated-gif)
- [Stream operations](#stream-operations)
  - [Copy/demux](#copydemux)
  - [Lossless split (trim) m4a](#lossless-split-trim-m4a)
  - [Split by duration, starting on a keyframe](#split-by-duration-starting-on-a-keyframe)
- [Scaling](#scaling)
  - [Downscale audio file](#downscale-audio-file)
  - [Video scaling](#video-scaling)
- [Other operations](#other-operations)
  - [Record desktop](#record-desktop)
  - [Build FFmpeg with libfdk-aac support](#build-ffmpeg-with-libfdk-aac-support)

## Conversion

### Audio file

```sh
ffmpeg -i <input> <output.wav>
ffmpeg -i <input> -ab 320k output.mp3
```

### Video+audio

See https://trac.ffmpeg.org/wiki/Encode/MPEG-4.

```sh
# - Default video encoder: libxvid
# - qscale:v = constant quantization, 1-31 (3 -> cartoon/93'/720p ≈ 2.1GiB; 4 -> cartoon/93'/720p ≈ 1.6GiB)
# - qscale:a = VBR, 0-9, (4 ≈ 165 kbps, 5 ≈ 130 kbps)
#
ffmpeg -i input.mp4 -c:v mpeg4 -vtag xvid -qscale:v 3 -c:a libmp3lame -qscale:a 4 output.avi

# CBR Video/Audio (1000k/93' = 838 MiB), and mix both audio channels into two channels (https://trac.ffmpeg.org/wiki/AudioChannelManipulation)
#
ffmpeg -i input.mp4 -c:v mpeg4 -vtag xvid -b:v 1000k -c:a libmp3lame -b:a 96k -af 'pan=stereo|c0<c0+c1|c1<c0+c1' output.avi
```

### Video to animated GIF

```sh
# Source: https://superuser.com/a/556031
#
# - 10 FPS (`fps=10`)
# - keep the same resolution
# - don't loop (`-loop -1`)
#
ffmpeg -i "$1" -vf "fps=10,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop -1 "${1%.*}.gif"
```

## Stream operations

### Copy/demux

```sh
# `c:a copy`:    codec:audio copy
# `-acodec copy: same as above`
# `vn`:          no video; required!
#
ffmpeg -i <input.mp4> -vn -acodec copy <output.aac>
ffmpeg -i <input.webm> -vn -acodec copy <output.ogg>
```

### Lossless split (trim) m4a

See http://uber-rob.co.uk/2014/02/splittingcutting-an-m4a-file.

```sh
# Add `-vn` if there is video
# `-t` is the length
#
ffmpeg -ss 0:00:42 -i in.m4a -c copy -t 1:00:00 out.m4a
```

### Split by duration, starting on a keyframe

```sh
# Cute but underwhelming results (see https://unix.stackexchange.com/q/1670).
#
ffmpeg -i input.mp4 -segment_time 00:10:00 -reset_timestamps 1 -f segment output%02d.avi
```

## Scaling

### Downscale audio file

```sh
ffmpeg -i <input.ext> -sample_fmt s16 -ar 44100 <output.ext>
```

### Video scaling

```sh
# Scale to fit within the specified dimensions with the same aspect ratio, eg. 1280x720 -> 800*450
# (https://trac.ffmpeg.org/wiki/Scaling).
# .
#
ffmpeg -i input.mp4 -vf scale=w=800:h=600:force_original_aspect_ratio=decrease output.avi

# Scale and enforce an aspect ratio
#
ffmpeg -i input.mp4 -vf scale=800:600 -aspect 4:3 output.avi
```

## Other operations

### Record desktop

```sh
# "-f x11grab -s 1920x1200 -r 10 -i :0.0" = input part; r=FPS
# "-vf scale=1280x800" = scaling
# "-c:v libx264 -preset slower -crf 32" = encoding; crf=constant rate factor
#
# ~2 MB/min
#
# source: https://askubuntu.com/questions/227464/record-my-desktop-in-mp4-format
#
ffmpeg -f x11grab -s 1920x1200 -r 10 -i :0.0 -vf scale=1280x800 -c:v libx264 -preset slower -crf 32 $HOME/Desktop/desktop_recording.mp4
```

### Build FFmpeg with libfdk-aac support

```sh
git clone https://github.com/FFmpeg/FFmpeg.git

cd FFmpeg

sudo apt install libfdk-aac-dev

# --enable-gpl --enable-nonfree: required in order to include also GPL-licensed stuff
# disable-debug: make small
#
./configure \
  --extra-libs="-lpthread -lm" \
  --enable-gpl --enable-nonfree --enable-libfdk-aac

make -j $(nproc)

ls -lh ffmpeg
```
