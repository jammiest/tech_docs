# FFmpeg用法

## 下载m3u8文件将其转化为mp4

如果有指向m3u8的链接，则有最为简单的下载方式：`ffmpeg -i [视频链接] [保存文件]`

例如，我需要保存抓包下来的网课至dst.mp4，则进行命令

```powershell
D:\> ffmpeg -i http://link.to.your/video.m3u8 dst.mp4
```

## mkv转mp4格式

原视频 `src.mp4`

转化后的视频`dst.mp4`

```powershell
D:\> ffmpeg -i src.mkv -c:v copy -c:a copy dst.mp4
```

## 添加字幕

### 为MP4文件添加ass字幕

待添加字幕`subtitle.ass`(确保格式为UTF-8)
原视频 `src.mp4`

转化后的视频`dst.mp4`

```powershell
D:\> ffmpeg -i src.mp4 -vf ass=subtitle.ass -y dst.mp4
```

## hevc编码的MP4视频转码为h264编码

原视频 `src.mp4`

转化后的视频`dst.mp4`

```powershell
D:\> fmpeg -i src.mp4 -c:a copy -c:s copy -c:v libx264 dst.mp4
```

或

```powershell
D:\> fmpeg -i src.mp4 -map 0 -c:a copy -c:s copy -c:v libx264 dst.mp4
```

顺带旋转角度也调整为0°.

## mkv转mp4

原视频 `src.mkv`

转化后的视频`dst.mp4`

```powershell
D:\> ffmpeg -i src.mkv -c:v copy -c:a copy dst.mp4
```

## 剪切视频

原视频 `src.mp4`

剪切后的视频`dst.mp4`

```powershell
D:\> ffmpeg -ss 00:00:00 -t 00:00:30 -i src.mp4 -vcodec copy -acodec copy dst.mp4
```

## 合并视频

包含合并的文件描述`list.txt`

合并后的视频`dst.mp4`

```powershell
D:\> ffmpeg -f concat -i list.txt -c copy dst.mp4
```

> 其中`list.txt`里内容为

```txt
file ./src1.mp4
file ./src2.mp4
```

## 将MP4转为m3u8文件

## 高级用法
## hls

将in.mkv转化为hls格式的文件

```ffmpeg -i in.mkv -c:v h264 -flags +cgop -g 30 -hls_time 1 out.m3u8```

> 这个例子将生成播放列表文件`out.m3u8`和分片文件`out0.ts`, `out1.ts`, `out2.ts`, 等.

另请参阅段复用器，它提供了更通用和更灵活的分段器实现，可用于执行 HLS 分段。