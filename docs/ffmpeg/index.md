# FFMpeg Encoding

This section covers FFMpeg encoding tips, example commands, and common options.

## Quick start

- Example: convert input to H.264 MP4:

```sh
ffmpeg -i input.mkv -c:v libx264 -preset medium -crf 23 -c:a aac output.mp4
```

## Topics to add

- Encoding profiles and CRF vs bitrate
- Hardware acceleration (VAAPI, NVENC)
- Container formats and stream copying
- Example scripts and automation

Feel free to add notes or examples below.
