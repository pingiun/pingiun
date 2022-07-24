# Somewhat longer commands that I could use again in the future

## ffmpeg

Transcode with default video settings, but only copy first audio stream. Transcode audio to stereo ac3 (Dolby-Digital). The default video settings for mkv are x264 with a quality setting (20) that delivers small files with acceptable looks.

```bash
ffmpeg -i input.mkv -c:a ac3 -ac 2 -c:s copy -map 0:v:0 -map 0:a:0 -map 0:s? output.mkv
```

Same as above, but scale to 1080p:

```bash
ffmpeg -i input.mkv -vf scale=1920:1080 -c:a ac3 -ac 2 -c:s copy -map 0:v:0 -map 0:a:0 -map 0:s? output.mkv
```

Same as above but do this for all 4k files that can be found in the directory (requires jq):

```bash
find . -type f -print0 | xargs -0 -I{} ffprobe -v error -select_streams v:0 -show_entries stream=height:format=filename -of json {} | jq --slurp --raw-output '.[] | select( .streams[0].height | contains(2160)) | .format.filename' > 4kfiles
xargs <4kfiles -n1 -d$'\n' ~/convert.sh

cat ~/convert.sh
#! /usr/bin/env bash

input="$1"
output=$(basename "$input" .mkv).1080p.mkv

echo "Converting $input to $output"
set +x
ffmpeg -i "${input}" -c:a ac3 -ac 2 -vf scale=1920:1080 -c:s copy -map 0:v:0 -map 0:a:0 -map 0:s? "${output}"
```