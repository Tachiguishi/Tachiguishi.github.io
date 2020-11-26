---
layout: post
title:  Youtube-dl
date:   2018-11-14 19:30:00 +0800
categories: tools
tags: youtube-dl
---

`youtube-dl`是一个命令行视频下载工具，可以下载包括`YouTube`在内的[众多网站](https://ytdl-org.github.io/youtube-dl/supportedsites.html)，
依赖于`Python`。[官网](https://github.com/ytdl-org/youtube-dl)  

## 安装

```shell
dnf install youtube-dl
```

## 使用

只要使用视频的`URL`地址即可下载，如果地址是播放列表则会下载播放列表中的所有视频  
这样不加其它参数下载的就是所能下载的最高品质的视频

```shell
youtube-dl $(url)
```


示例

```shell
youtube-dl https://www.youtube.com/watch?v=diuexInkshA
```

### 指定格式下载

查看可以下载的所有视频格式列表，然后可以根据列表选择格式自己的特定格式下载

#### 查看可下载格式列表

```shell
youtube-dl -F $(url)
# or
youtube-dl --list-formats $(url)
```

示例

`youtube-dl -F https://www.youtube.com/watch?v=diuexInkshA`

```
[youtube] diuexInkshA: Downloading webpage
[youtube] diuexInkshA: Downloading video info webpage
[youtube] diuexInkshA: Downloading js player vflca9-f7
[info] Available formats for diuexInkshA:
format code  extension  resolution note
249          webm       audio only DASH audio   53k , opus @ 50k, 1.06MiB
250          webm       audio only DASH audio   70k , opus @ 70k, 1.40MiB
140          m4a        audio only DASH audio  130k , m4a_dash container, mp4a.40.2@128k, 2.86MiB
251          webm       audio only DASH audio  140k , opus @160k, 2.80MiB
171          webm       audio only DASH audio  140k , vorbis@128k, 2.87MiB
160          mp4        256x144    144p   14k , avc1.4d400c, 30fps, video only, 248.81KiB
133          mp4        426x240    240p   19k , avc1.4d4015, 30fps, video only, 336.77KiB
134          mp4        640x360    360p   28k , avc1.4d401e, 30fps, video only, 479.88KiB
135          mp4        854x480    480p   34k , avc1.4d401f, 30fps, video only, 572.10KiB
278          webm       256x144    144p   37k , webm container, vp9, 30fps, video only, 599.83KiB
394          mp4        256x144    144p   39k , av01.0.05M.08, 30fps, video only, 651.94KiB
242          webm       426x240    240p   47k , vp9, 30fps, video only, 758.27KiB
395          mp4        426x240    240p   48k , av01.0.05M.08, 30fps, video only, 772.96KiB
136          mp4        1280x720   720p   54k , avc1.4d401f, 30fps, video only, 983.74KiB  
243          webm       640x360    360p   65k , vp9, 30fps, video only, 1.05MiB
244          webm       854x480    480p   67k , vp9, 30fps, video only, 1.06MiB
396          mp4        640x360    360p   74k , av01.0.05M.08, 30fps, video only, 1.14MiB  
397          mp4        854x480    480p  110k , av01.0.05M.08, 30fps, video only, 1.67MiB  
247          webm       1280x720   720p  137k , vp9, 30fps, video only, 2.10MiB
137          mp4        1920x1080  1080p  175k , avc1.640028, 30fps, video only, 2.50MiB
248          webm       1920x1080  1080p  279k , vp9, 30fps, video only, 4.20MiB
18           mp4        640x360    medium  200k , avc1.42001E, mp4a.40.2@ 96k (44100Hz), 4.42MiB (best)
```

上面的列表中可以看到`audio only`和`video only`的字样，这是因为`YouTube`的高品质视频中画面与音频是分离的，可以自由选择组合下载

#### 选择格式下载

```shell
youtube-dl -f $(format-code) $(url)
```

示例：  

```shell
# 只下载音频
youtube-dl -f 171 https://www.youtube.com/watch?v=diuexInkshA
# 自由匹配音频与视频
youtube-dl -f 171+244 https://www.youtube.com/watch?v=diuexInkshA
```

也可以使用特定的名称来指定格式，这样就不用预先知道格式代码了

* best: Select the best quality format represented by a single file with video and audio.
* worst: Select the worst quality format represented by a single file with video and audio.
* bestvideo: Select the best quality video-only format (e.g. DASH video). May not be available.
* worstvideo: Select the worst quality video-only format. May not be available.
* bestaudio: Select the best quality audio only-format. May not be available.
* worstaudio: Select the worst quality audio only-format. May not be available.

### 字幕

* `--list-subs`	List all available subtitles for the video
* `--write-sub`	Write subtitle file
* `--write-auto-sub`	Write automatically generated subtitle file (YouTube only)
* `--all-subs`	Download all the available subtitles of the video
* `--sub-format FORMAT`	Subtitle format, accepts formats preference, for example: "srt" or "ass/srt/best"
* `--sub-lang LANGS`	Languages of the subtitles to download (optional) separated by commas
* `--convert-subs FORMAT`	Convert the subtitles to other format (currently supported: srt/ass/vtt/lrc)

### 代理

```shell
youtube-dl --proxy socks5://127.0.0.1:1080
youtube-dl --proxy https://127.0.0.1:1080
```

## ffmpeg

由于`YouTube`上最高品质的资源都是视频与音频分离的，所以下载后需要使用`ffmpeg`等其它工具合并。  
方便的是`youtube-dl`在下载后会自动调用`ffmpeg`来合并。但还是需要手动转换格式

```shell
dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

dnf install ffmpeg ffmpeg-devel
```

[How to Install FFmpeg on Fedora](https://tecadmin.net/install-ffmpeg-on-fedora/)

### 转换格式

```shell
ffmpeg -i inputfile outputfile
```

`ffmpeg`会根据文件后缀名来判断文件类型，然后进行相应的转换

示例

```shell
# 将`webm`转换成`mp3`
ffmpeg -i diuexInkshA.f171.webm DevilmanNoUta.mp3
```

## 总结

一般情况下在安装好`youtube-dl`与`ffmpeg`后使用最简单的方式即可

`youtube-dl https://www.youtube.com/watch?v=diuexInkshA`

```
[youtube] diuexInkshA: Downloading webpage
[youtube] diuexInkshA: Downloading video info webpage
[youtube] diuexInkshA: Downloading js player vflca9-f7
[download] Destination: Débilman No Uta (Full) - Devilman Crybaby OST-diuexInkshA.f248.webm
[download] 100% of 4.20MiB in 00:10
[download] Destination: Débilman No Uta (Full) - Devilman Crybaby OST-diuexInkshA.f171.webm
[download] 100% of 2.87MiB in 00:03
[ffmpeg] Merging formats into "Débilman No Uta (Full) - Devilman Crybaby OST-diuexInkshA.webm"
Deleting original file Débilman No Uta (Full) - Devilman Crybaby OST-diuexInkshA.f248.webm (pass -k to keep)
Deleting original file Débilman No Uta (Full) - Devilman Crybaby OST-diuexInkshA.f171.webm (pass -k to keep)
```

可以看到，`youtube-dl`先下载了两个文件，一个最好品质音频，一个最好品质视频，然后自动调用`ffmpeg`进行合并，最后删除下载的两个原始文件

## 自动下载

由于网络因素，晚上下载速度慢且不稳定，凌晨速度快且稳定，所以我制作了一个定时脚本放在`Raspberry Pi`上，让它在每天早上4点去下载我想要的视频

### 下载脚本

`/home/pi/.bin/youbutedaily`

```shell
#!/bin/sh

# video URL that I want to download
youlist=/home/pi/.youtube
# video URL that downloading failed
faillist=/tmp/youfailed

if [ -f $youlist ]; then

# remove empty line
sed -i "/^$/d" $youlist
# clear failed URL
echo > $faillist

# download
while read line
do
	# check for empty line
	if [ -z "$line" ]; then
		echo "pass"
		continue
	fi
	echo "$line"" begin"
	youtube-dl --proxy socks5://127.0.0.1:1080 -o "/home/pi/%(title)s-%(id)s.%(ext)s" $line 2>> /home/pi/.youtube.log
	if [ $? -ne 0 ]; then
		echo "$line" >> $faillist
	fi
done < $youlist

# remove success URL from list
cat $faillist > $youlist

fi
```

> 写定时任务脚本时注意`$PATH`环境与直接在终端执行不同，具体看`/etc/crontab`文件中的设置
> 也可以自己通过`crontab -e`添加必要的路径

### 定时执行

```shell
crontab -l
# 0 4 * * * /home/pi/.bin/youbutedaily
```

> 设置定时任务时需要注意系统时区

### 添加下载地址到`.youtube`文件

对于一般视频，只要直接添加视频地址即可。但有些视频我只想下载音频，所以只要在地址前添加`-f bestaudio`即可

`/home/pi/.bin/bestaudio`

```shell
#!/bin/sh

if [ $# -ne 1 ]; then
	echo "wrong parameters"
fi

echo "-f bestaudio ""$1" >> /home/pi/.youtube

cat /home/pi/.youtube
```
