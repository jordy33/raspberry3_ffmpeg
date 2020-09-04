## How to install ffmpg in raspberry

### Install Tvheadend
Update
```
apt-get update
apt-get upgrade
```
### Adjust language English and us keyboard
```
piwiz
```


### Install FFmpeg
Now we need to install the packages that we need to compile FFmpeg and its additional libraries.

As there are quite a few, the installation process may take some time to complete.

Run the following command to install all of the required packages to your Raspberry Pi.
```
sudo apt -y install autoconf automake build-essential cmake doxygen git graphviz imagemagick libasound2-dev libass-dev libavcodec-dev libavdevice-dev libavfilter-dev libavformat-dev libavutil-dev libfreetype6-dev libgmp-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libopus-dev librtmp-dev libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-net-dev libsdl2-ttf-dev libsnappy-dev libsoxr-dev libssh-dev libssl-dev libtool libv4l-dev libva-dev libvdpau-dev libvo-amrwbenc-dev libvorbis-dev libwebp-dev libx264-dev libx265-dev libxcb-shape0-dev libxcb-shm0-dev libxcb-xfixes0-dev libxcb1-dev libxml2-dev lzma-dev meson nasm pkg-config python3-dev python3-pip texinfo wget yasm zlib1g-dev libdrm-dev
```

Before we get started, let’s create a directory where we will store the code for each of these libraries.
```
mkdir ~/ffmpeg-libraries
```

The first library that we are going to compile is the Fraunhofer FDK AAC library.

Compiling this library will allow FFmpeg to have support for the AAC sound format.

Run the following command to download and compile the source code to your Raspberry Pi.
```
git clone --depth 1 https://github.com/mstorsjo/fdk-aac.git ~/ffmpeg-libraries/fdk-aac \
  && cd ~/ffmpeg-libraries/fdk-aac \
  && autoreconf -fiv \
  && ./configure \
  && make -j$(nproc) \
  && sudo make install
```

The next library we are going to compile is the “dav1d” library.

This library will add support for decoding the AV1 video format into FFmpeg. This codec is considered the successor of the VP9 codec and as a competitor to the x265 codec.

Run the following command to compile and install the “dav1d” library to your Raspberry Pi.

```
git clone --depth 1 https://code.videolan.org/videolan/dav1d.git ~/ffmpeg-libraries/dav1d \
  && mkdir ~/ffmpeg-libraries/dav1d/build \
  && cd ~/ffmpeg-libraries/dav1d/build \
  && meson .. \
  && ninja \
  && sudo ninja install
```

This library that we are going to compile next is an HEVC encoder called “kvazaar“.

Using the following command, you can clone and compile the Kvazaar library on your Raspberry Pi.
```
git clone --depth 1 https://github.com/ultravideo/kvazaar.git ~/ffmpeg-libraries/kvazaar \
  && cd ~/ffmpeg-libraries/kvazaar \
  && ./autogen.sh \
  && ./configure \
  && make -j$(nproc) \
  && sudo make install
```

We can now compile the library that we need for FFmpeg to be able to support the VP8 and VP9 video codecs on our Raspberry Pi.

This library we are compiling is called LibVPX and is developed by Google.

The following command will clone, configure, and compile the library to our Pi.

```
git clone --depth 1 https://chromium.googlesource.com/webm/libvpx ~/ffmpeg-libraries/libvpx \
  && cd ~/ffmpeg-libraries/libvpx \
  && ./configure --disable-examples --disable-tools --disable-unit_tests --disable-docs \
  && make -j$(nproc) \
  && sudo make install
```

We now need to compile the library called “AOM“.

This library will allow us to add support for encoding to the AP1 video codec on your Raspberry Pi.

```
git clone --depth 1 https://aomedia.googlesource.com/aom ~/ffmpeg-libraries/aom \
  && mkdir ~/ffmpeg-libraries/aom/aom_build \
  && cd ~/ffmpeg-libraries/aom/aom_build \
  && cmake -G "Unix Makefiles" AOM_SRC -DENABLE_NASM=on -DPYTHON_EXECUTABLE="$(which python3)" -DCMAKE_C_FLAGS="-mfpu=vfp -mfloat-abi=hard" .. \
  && sed -i 's/ENABLE_NEON:BOOL=ON/ENABLE_NEON:BOOL=OFF/' CMakeCache.txt \
  && make -j$(nproc) \
  && sudo make install
```

The final library we need to compile is the “zimg” library.

This library implements a range of image processing features, dealing with the basics of scaling, colorspace, and depth.

Clone and compile the code by running the command below.
```
git clone https://github.com/sekrit-twc/zimg.git ~/ffmpeg-libraries/zimg \
  && cd ~/ffmpeg-libraries/zimg \
  && sh autogen.sh \
  && ./configure \
  && make \
  && sudo make install
```
Now, run the command below to update the link cache.

This command ensures we won’t run into linking issues because the compiler can’t find a library.
```
sudo ldconfig
```
We can finally compile FFmpeg on our Raspberry Pi.

During the compilation, we will be enabling all the extra libraries that we compiled and installed in the previous two sections.

We will also be enabling features that help with the Raspberry Pi, such as omx-rpi.

Run the following command to compile everything. This command is reasonably large, as there is a considerable amount of features that we need to enable.
```
it clone --depth 1 https://github.com/FFmpeg/FFmpeg.git ~/FFmpeg \
  && cd ~/FFmpeg \
  && ./configure \
    --extra-cflags="-I/usr/local/include" \
    --extra-ldflags="-L/usr/local/lib" \
    --extra-libs="-lpthread -lm -latomic" \
    --arch=armel \
    --enable-gmp \
    --enable-gpl \
    --enable-libaom \
    --enable-libass \
    --enable-libdav1d \
    --enable-libdrm \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libkvazaar \
    --enable-libmp3lame \
    --enable-libopencore-amrnb \
    --enable-libopencore-amrwb \
    --enable-libopus \
    --enable-librtmp \
    --enable-libsnappy \
    --enable-libsoxr \
    --enable-libssh \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libzimg \
    --enable-libwebp \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libxml2 \
    --enable-mmal \
    --enable-nonfree \
    --enable-omx \
    --enable-omx-rpi \
    --enable-version3 \
    --target-os=linux \
    --enable-pthreads \
    --enable-openssl \
    --enable-hardcoded-tables \
  && make -j$(nproc) \
  && sudo make install
```

