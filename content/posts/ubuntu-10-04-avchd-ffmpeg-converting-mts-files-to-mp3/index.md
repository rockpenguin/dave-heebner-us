---
title: "Ubuntu 10.04 + AVCHD + FFmpeg: Converting .MTS files to MP3"
date: 2010-08-02
categories: 
  - "techsupport"
---

Well, it's been awhile since I posted last. Been a little busy.

Where I work, we have to convert a lot of audio/video to another format for transcription purposes. Mplayer and ffmpeg have been lifesavers for me. However, getting setup to convert the newer .MTS (AVCHD) files that the latest videocams use was a little tedious as Google searches often yielded a lot of results that were not quite the solution I needed for Ubuntu 10.04. I realize this is a moving target for each release of Ubuntu, but hopefully this will help Ubuntu users (NOTE: this is more helpful for me as I will forget how to do this in 2 weeks). Here goes nothing!

OK, ready? Get a cup of coffee as this is going to take awhile...

1. <!--more--> Install ffmpeg if you haven't already
2. install "libavcodec-extra-52" (aptitude install libavcodec-extra-52)

That's it!  Here is a list of some of the packages I have installed on my PC:

> ffmpeg gstreamer0.10-ffmpeg libavcodec-extra-52 gstreamer0.10-alsa gstreamer0.10-ffmpeg gstreamer0.10-fluendo-mp3
> 
> gstreamer0.10-gnomevfs gstreamer0.10-gnonlingstreamer0.10-nice gstreamer0.10-plugins-bad gstreamer0.10-plugins-base gstreamer0.10-plugins-base-apps gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-pulseaudio gstreamer0.10-tools gstreamer0.10-x libgstreamer-plugins-base0.10-0 libgstreamer0.10-0

That's all for now!
