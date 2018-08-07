# Avoid transcoding by converting videos automatically

In many cases (for example, when using an Apple TV to watch videos) I faced a heavy transcoding, that resulted in frequent buffering - around every 3 minutes while watching a 720p TV show.

A way to work around that is to convert videos in a more friendly format, using a tool like Handbrake.

Since our target is to have a fully autonomous Torrent / Plex machine, we will rely on a script to run conversions automatically when a new file is downloaded.

## Pre-requisites

The automation is based on [Handbrake](https://handbrake.fr) and [MediaInfo](https://mediaarea.net/en/MediaInfo), so we will need to install both.

### Handbrake

Compiling Handbrake on Raspbian 9 requires some small changes to the standard process. For starters, we install all required packages with:

```
$ sudo apt-get install autoconf automake build-essential cmake git libass-dev libbz2-dev libfontconfig1-dev libfreetype6-dev libfribidi-dev libharfbuzz-dev libjansson-dev liblzma-dev libmp3lame-dev libogg-dev libopus-dev libsamplerate-dev libspeex-dev libtheora-dev libtool libtool-bin libvorbis-dev libx264-dev libxml2-dev m4 make patch pkg-config python tar yasm zlib1g-dev
```

Then download the source code for HandBrake with:

```
$ git clone https://github.com/HandBrake/HandBrake.git && cd HandBrake
```

#### Patches

Before we can build HandBrake we need to apply two patches. First we download them (feel free to check the content, to confirm I am not trying anything fancy):

```
$ wget -O ffmpeg_arm_patch.diff "https://www.dropbox.com/s/dl/ocahrhwsftxaccy/ffmpeg_arm_patch.diff"
$ wget -O x264_arm_patch.diff "https://www.dropbox.com/s/dl/11bd39fpqa32hci/x264_arm_patch.diff"
```

And then we apply them with:

```
$ patch -p0 < ffmpeg_arm_patch.diff
$ patch -p0 < x264_arm_patch.diff
```

#### Compiling

Now we can actually configure and build HandBrake with:

```
$ ./configure --disable-x265 --disable-gtk
$ cd build/
$ make
$ make install
```

If the configuration returns any error (for example for missing dependency) you have to run `make clean` before trying again, or

```
$ cd ..
$ rm -rf build
```

### Mediainfo

Installing Mediainfo is as easy as executing the following:

```
$ sudo apt-get install mediainfo
```
