---
title: Reveal Embedded Subtitles with FFmpeg
date: 2023-12-10 12:00:00 -500
categories: [video,editing]
tags: [video,editing]
---

![featured_image](/assets/subtitles/featured_image.png)

### Extracting Subtitles Using FFmpeg!

"How can I overlay subtitle tracks generated by my camera onto my video clips?" I've seen this type of question pop up a at least once a month on video editing communities, espcially those focused around the usage of drones. The response is usually something like "Good luck!" or "You can't without paying for specialized software.". Thankfully there is an easy way to do this with FFmpeg.

* Step 1: Install FFmpeg
* Step 2: Use FFmpeg to extract the embedded track(s)
* Step 3: Import the subtitle track(s) into your preferred editor

---

#### Step 1: Download and install FFmpeg

##### For Windows

I prefer to manage Windows packages with [Scoop](https://scoop.sh/).

Use Scoop to install FFmpeg

```powershell
scoop install ffmpeg
```

##### For Linux

FFmpeg is widely available across many Linux distrubtions and may already be installed on your machine.

* Use your preffered package manager, I'll be using Pacman since I use Arch Linux.

```bash
pacman -S ffmpeg
```

#### Step 2: Using FFmpeg to extract subtitles

The following command will extract the subtitles from "my_video.mp4" and save them as "subtitles.srt".

```bash
ffmpeg -i my_video.mp4 subtitles.srt
```

A new SubRip Subtitle (.srt) file will be created, you can view the contents of the file with a text editor or import it into your preffered editing application.

#### Step 3: Import the SubRip Subtitle file into your video editor

For DaVinci Resolve, all you have to do is right click in your media pool and import just like you would any other form of media, and then drag to your timeline.

![davinci_timeline](/assets/subtitles/davinci_timeline.png)

For Kdenlive you need to import it from the "Project" drop down menu before you can drag it into your timeline.

![kdenlive_timeline](/assets/subtitles/kdenlive_import.png)

Now you can display all of this data however you like!