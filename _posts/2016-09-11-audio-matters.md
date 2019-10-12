---
title: Why game audio matters
date: 2016-09-11 00:00:00 Z
tags:
- Unity3D,
- gamedev
description: 
layout: post
image: https://media.licdn.com/media/gcrc/dms/image/C4E12AQEHRsgRSlYq_A/article-cover_image-shrink_600_2000/0?e=2131315200&v=beta&t=4t81HQZJtzWGGxMZLCruKY6pCQhxUlxMffDqBrROLxc
---

There is a plentiful of situations where you game development workflow can go wrong, and not run smoothly on your target devices. maybe there are too many draw calls and materials, maybe your polycount is pretty high for the devices, or your game logic turns the CPU to an eggpan. in this article we'll focus on Audio/Music, a field most solo/indie developers put in their last stages, and why audio matters as much as graphics.

### a brief look at Audio Clips.

Most of the times when working with unity you are settling with default settings, a quick drag'n'drop and things works just fine. but hey, this rule won't apply to audio settings.

While sounds import feels similar to importing textures and sprites and while unity supports many formats, during the build phase it will convert them to it's preferred one.

![alt text](https://media.licdn.com/mpr/mpr/shrinknp_400_400/AAEAAQAAAAAAAAhbAAAAJDEyZTQxNDRiLWMzZGEtNDZmMi05ZDdiLTc2ZjYzZjkzNDliYw.png)

By default, Unity imports audio clips with "Decompress on Load" load type and "Vorbis" format. Using this format for all your clips may (and will) use shitloads of memory. as shown in the image example: Original file size is 35.9 MB and the imported size is 10.7 MB. This means that this Clip will increase your game (archive) size by 10 megabytes, but playing it will require nearly 36 megabytes of RAM! Sounds scary if you plan to deploy your game to mobile devices.

### Audio Load Types:

There are three supported Load types and you're the only one who knows when each combinaison will be used, let's dissect our possible load types:

* Compressed In Memory – Audio Clip will be stored in RAM and will be uncompressed when played. Will not use any additional memory when playing.
* Streaming – Audio Clip will be stored on a persistent path (hard drive, flash drive etc) and streamed when played. Does not require RAM for storing and playing (or not significant).
* Decompress On Load – Audio Clip will be stored in RAM uncompressed. This option requires the most memory but playing it won’t require much CPU power as the rest.

So using one or the other will depend on...

### BGM: Music/Ambient sounds:
music is basicly in form of lonnnng audio clip files, decompressing it when played may consume lots of memory, so IMHO i'll stick with:

**Streaming+Vorbis** will use the least amount of memory, but it may require some CPU and constant disk input/output.

**Compressed+Vorbis** will exchange I/O for some RAM requirement, may works great if you decrease the quality slider (i'm happy with 60% in most games)

### SFX:

sound effects in games are short (0-3sec) or medium (3-10sec) audio files that are played frequently(footsteps, bullet shots)<-->occasionally(doors opens) <-->rarely(win/loose situation), here i present you some rules(not facts, just empiric guidelines i collected from my experience of failing)

1. **frequently played, short** *Audio Clips* [bullet shots, footsteps, collectible audio feedbacks ] i'll use *Decompress On Load* and *PCM/ADPCM Compression Format*. in case of PCM format, no decompression is needed and if audio clip is short it will load very quickly. You can also use *ADPCM*. It requires decompression, but waaaay lighter than *Vorbis*.
2. **frequently played, medium** *Audio Clips* [Spell casts, frequent voices]  use *Compressed In Memory* and *ADPCM Compression Format*. ADPCM is around 3.5 times smaller than raw *PCM* and decompression algorithm will not consume as much CPU as *Vorbis*.
3. **rarely played, short** *Audio Clips* use *Compressed In Memory* and *ADPCM*. For the same reason as described in point 2.
4. **rarely played, medium** *Audio Clips* [Win situation trumpet, Loose situation short music]  use *Compressed In Memory*  and *Vorbis* Compression Format. This SFX might be too long to be stored using *ADPCM* and played too rarely, so the additional CPU power required to decompress is a nice bargain.

>To summarize, Audio is as important as graphics. Set it right, and you will get additional power for stuff that matters.
>As always, thanks for reading.
