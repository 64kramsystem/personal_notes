# FFmpeg/Avconv

- [FFmpeg/Avconv](#ffmpegavconv)
  - [Generic options](#generic-options)
  - [Conversion](#conversion)
    - [Convenient snippets](#convenient-snippets)
    - [libfdk-aac test](#libfdk-aac-test)
    - [libx265 test](#libx265-test)
    - [Video to animated GIF/PNG](#video-to-animated-gifpng)
  - [Stream/file operations](#streamfile-operations)
    - [Demux](#demux)
    - [Lossless split (trim) m4a](#lossless-split-trim-m4a)
    - [Extract segment](#extract-segment)
    - [Split by duration, starting on a keyframe](#split-by-duration-starting-on-a-keyframe)
    - [Concatenate videos](#concatenate-videos)
  - [Scaling](#scaling)
  - [Other operations](#other-operations)
    - [Snippets](#snippets)
    - [Build FFmpeg with libfdk-aac support](#build-ffmpeg-with-libfdk-aac-support)

## Generic options

- `-y`                            : overwrite destination
- `-c:a $codec`, `-acodec $codec` : Audio codec
  - `-an`                         : Ignore audio
- `-c:v $codec`                   : Video codec
  - `-vn`                         : Ignore video

## Conversion

MP3 (LAME: `-c:a libmp3lame`):

- `-qscale:a 4`: VBR
  - 4: ≈165 kbps, 5: ≈130 kbps
- `-b:a 320k`  : CBR

M4A (FDK-AAC `-c:a libfdk_aac`):

- `-vbr 5`: VBR
  - Ignore the warning `Note, the VBR setting is unsupported and only works with some parameter combinations` (see https://www.mail-archive.com/ffmpeg-user@ffmpeg.org/msg23605.html)
  - WATCH OUT! Not all the bitrate/channels/quality combinations work (see https://hydrogenaud.io/index.php/topic,95989.msg817833.html#msg817833)

WAV:
- `-f wav`; specify format; optional if the output has the extension.

h265 (libx265: `-c:v libx265`):

- `-crf 25 -preset slower`: VBR (CRF); use x265
  - CRF: best:0, default:28, worst:51
  - Preset: ultrafast, ..., slower, slow, medium, ...
- `-x265-params lossless=1`: Lossless (no need for `crf` param)

MP4 (libxvid (default): `-c:v mpeg4 -vtag xvid`):

- `-qscale:v 3`: VBR (CRF), 1-31
  - 3: cartoon/93'/720p ≈2.1GiB; 4: cartoon/93'/720p ≈1.6GiB
- `-b:v 1000k` : CBR

For h264/h265 options, see:

- https://trac.ffmpeg.org/wiki/Encode/H.265
- https://trac.ffmpeg.org/wiki/Encode/MPEG-4

### Convenient snippets

Quick audio spectrum/bitrate check:

```sh
rm -rf /tmp/z
mkdir /tmp/z

find *.flac | parallel 'ffmpeg -i {} -c:a libfdk_aac -vbr 2 /tmp/z/"$(basename {})".m4a'

find /tmp/z/*.m4a | parallel 'ffmpeg -i {} {}.wav && sox {}.wav -n spectrogram -o {}.wav.png'; eom /tmp/z/*.png

(
  echo 'puts ('
  for f in /tmp/z/*.m4a; do ffprobe "$f" 2>&1 | perl -lne 'print "$1+" if /bitrate: (\d+)/'; done
  echo "0) / $(ls -1 /tmp/z/*.m4a | wc -l)"
) | ruby
```

Convert unencrypted VOBs to h265:

```sh
# Inspect the audio and if it's 192kb, decide if copy or compress it (see comment).
# On a test movie, 192kb ac3 was 148 MB, aac (q3, 44 kHz) 70 MB
#
ffprobe VTS_01_1.VOB

ffmpeg \
  -i "concat:$(ls -1 *.VOB | tr $'\n' '|')" \
  -ac 2 -ar 44100 -c:a libfdk_aac -vbr 3 `# -c:a copy` \
  -c:v libx265 -crf 25 -preset slower \
  /tmp/output.mkv
```

### libfdk-aac test

Tested on 4 albums of different genre; kbps/khz ~cutoff:

- 5: 225/19
- 4: 149/16.5
- 3: 115/16
- 2:  99/13
- 1: 102/13 (!!)

Downsampling from 48Khz to 44Khz saved around 3% on a test DVD.

### libx265 test

General infos:

- It's not clear what "intra encoding" from the guide is, but it makes `slower` output size balloon!
- CRF:
  - 28: like x264's 23, at around half size
  - every 6 values, size approximately doubles (based on x264 guide)
  - visually lossless should be at around 22 (based on x264 guide)
- presets:
  - `slow` is considered very slow already; few people suggest `slower`
- Compressing a video (+audio) employed around 1000/1200% CPU time, so if there are many, best to compress 2 or 3 in parallel

Test data:

- Source (Swing 5.3 recap), h264 (Lavf58.20.100): 1080p=24M (reference), 540p=6.8M
- Based on this test, the most balanced is around 25+slower

| height |   preset    |  crf  |  fps  | size (M) | notes                                                                                             |
| :----: | :---------: | :---: | :---: | :------: | ------------------------------------------------------------------------------------------------- |
|  540p  |   medium    |  28   | 226.5 |   2.0    | size inconsistent with `slower`; didn't visually inspect                                          |
|  540p  |   slower    |  28   |  19   |   2.1    | can hardly distinguish from LL!                                                                   |
|  540p  |   slower    |  26   | 17.5  |   2.6    |                                                                                                   |
|  540p  |   slower    |  25   | 16.8  |   3.0    | very small improvements vs CRF 28 (required frame inspection); can't distinguish from source 540p |
|  540p  |   medium    |  LL   | 63.9  |   119    |                                                                                                   |
|  540p  | x264/slower |  23   |       |   3.9    | very hard to distinguish from x265/25/slower; slightly blockier on moving areas                   |

Fast presets:

- Source: 30 minutes of a documentary DVD, video only

|  preset   | speed |
| :-------: | :---: |
| ultrafast | 30.8  |
| superfast | 26.8  |
| veryfast  | 15.8  |

### Video to animated GIF/PNG

See concatenation section to use multiple input videos as source.

Note that asciinema is nice, however, more complex usages are not supported (eg. switching terminal); in such cases, recording with Virtualbox and converting with ffmpeg is an acceptable strategy.

2 FPS are the bare minimum, for resource-restricted animations. For detail-oriented ones, 4+ are probably a starting point.

GIF:

```sh
# Source: https://superuser.com/a/556031
#
# - `fps=10`: 10 FPS
# - keep the same resolution
# - `-loop -1`: don't loop
#
ffmpeg -i "$input" -vf "fps=10,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop -1 "${input%.*}.gif"
```

(A)PNG; considerably larger than GIF; may not make a noticeable difference.

```sh
# `-plays 10`: 10 loops (use `0` for infinite)
# `-r 1/2`: capture 2 frames every second (keeps the video time the same!)
#
ffmpeg -i "$input" -plays 10 -r 1/2 "${input%.*}.apng"
```

## Stream/file operations

### Demux

```sh
# `c:a copy`:    codec:audio copy
# `vn`:          no video; required!
#
ffmpeg -i $input -vn -c:a copy $output

# Demux a single stream.
# See `Stream` from ffprobe (`Stream #0:1: Audio` -> `0:a:1`)
#
ffmpeg -i $input -vn -map 0:a:1 -c copy $audio_output
```

### Lossless split (trim) m4a

See http://uber-rob.co.uk/2014/02/splittingcutting-an-m4a-file.

```sh
# Add `-vn` if there is video
# `-t` is the length
#
ffmpeg -ss 0:00:42 -i in.m4a -c copy -t 1:00:00 out.m4a
```

### Extract segment

```sh
# `ss`: start
# `t` : duration

ffmpeg -ss hh:mm:ss -i $source -t hh:mm:ss -vcodec copy -acodec copy
```

### Split by duration, starting on a keyframe

```sh
# Cute but underwhelming results (see https://unix.stackexchange.com/q/1670).
#
ffmpeg -i input.mp4 -segment_time 00:10:00 -reset_timestamps 1 -f segment output%02d.avi
```

### Concatenate videos

Source: https://stackoverflow.com/a/11175851

Concat demuxer (lossless): all files must have the same parameters.

```sh
ffmpeg -f concat -safe 0 -i /dev/stdin output.apng << LIST
file '$PWD/01.webm'
file '$PWD/02.webm'
file '$PWD/03.webm'
LIST

# Alternative approach, with wildcard (simplified; doesn't support filenames with single quotes).
#
(for f in *.webm; do echo "file '$PWD/$f'"; done) | ffmpeg -f concat -safe 0 -i /dev/stdin output.apng
```

For formats supporting it (ie. MPEG-2), can use the `concat` filter: `-i "concat:input1|input2...`.

## Scaling

Audio:

- `-sample_fmt s16`                   : Downscale to 16 bit
- `-ac (1|2)`                         : Convert to mono/stereo (space required after `-ac`); if source and filter have the same # of channels, the filter is noop.
- `-ar 44100`                         : Downsample
- `-af 'pan=stereo|c0<c0+c1|c1<c0+c1'`: Mix both audio channels into two channels (https://trac.ffmpeg.org/wiki/AudioChannelManipulation)

Video:

- `-vf scale=w=800:h=600:force_original_aspect_ratio=decrease` : Scale to fit within the specified dimensions with the same aspect ratio, eg. 1280x720 -> 800*450
                                                                 (https://trac.ffmpeg.org/wiki/Scaling).
- `-vf scale=800:600 -aspect 4:3`                              : Scale and enforce an aspect ratio
- `-filter:v scale=-1:540`                                     : Scale, keeping width proportional (works also for height)

## Other operations

### Snippets

```sh
# Check if a video file is truncated (`-sseof -60`: last 60 seconds)
#
ffmpeg -v error -sseof -60 -i video.mkv -f null -

# Check file encoding formats (metadata)
#
ffmpeg -i $filename
ffprobe $filename

# Record desktop:
#
# "-f x11grab -s 1920x1200 -r 10 -i :0.0" = input part; r=FPS
# "-vf scale=1280x800" = scaling
# "-c:v libx264 -preset slower -crf 32" = encoding; crf=constant rate factor
#
# ~2 MB/min
#
# source: https://askubuntu.com/questions/227464/record-my-desktop-in-mp4-format
#
ffmpeg -f x11grab -s 1920x1080 -r 10 -i :0.0 -vf scale=1280x800 -c:v libx264 -preset slower -crf 32 $HOME/Desktop/desktop_recording.mp4

# Check FFmpeg linked binaries
#
ldd $(which ffmpeg)
```

### Build FFmpeg with libfdk-aac support

```sh
sudo apt install \
  libfdk-aac-dev libx264-dev frei0r-plugins-dev libgnutls28-dev libaom-dev libass-dev libmp3lame-dev \
  libopenjp2-7-dev libvorbis-dev libvpx-dev libwebp-dev libx265-dev ocl-icd-opencl-dev libdrm-dev

git clone https://github.com/FFmpeg/FFmpeg.git

cd FFmpeg

latest_release=$(git branch -r | perl -lne 'print $1 if /origin\/release\/([\d.]+)/' | sort -V | tail -n 1)
git checkout release/"$latest_release"

# Compile minimal version, with some options taken from Ubuntu:
#
# `-lpthread`: include pthread library
# `-lm`: include standard C math library `libm` (see https://stackoverflow.com/q/1033898)
# --enable-gpl --enable-nonfree: required in order to include also GPL-licensed stuff
#
# In order to check how an ffmpeg binary was compiled, run `ffmpeg -version`; switches can be listed via `./configure --help`.
#
# Notes:
#
# - Not availabile anymore on 5.0: `--disable-filter=resample`.
# - Not sure if needed: `--extra-libs="-lpthread -lm"`
#
./configure \
  --enable-gpl --enable-nonfree --enable-libfdk-aac --enable-libx264 \
  --prefix=/usr --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 \
  --enable-libmp3lame --enable-libopenjpeg --enable-opencl --enable-opengl --enable-sdl2 --enable-gnutls --enable-libaom --enable-libass \
  --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx265 --enable-libdrm --enable-libx264 --enable-shared

make -j $(nproc)
```
