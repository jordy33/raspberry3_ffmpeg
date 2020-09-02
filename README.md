## How to install tvheadend in ubuntu

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

Installing Dependencies
```
sudo apt-get install gitk
sudo apt-get install libiconv
sudo apt-get install libic6
sudo apt-get install libc6
sudo apt-get install libiconv-dev
sudo apt-get install libc6-dev
sudo apt-get install clang
sudo apt-get install libdvbcsa-dev
sudo apt-get install dvb-apps
git clone https://github.com/tvheadend/dtv-scan-tables.git
cd dtv-scan-tables/
make clean
sudo make dvbv3
sudo make install
```
Installing Tvheadend
```
git clone https://github.com/tvheadend/tvheadend.git
gitk --all
git checkout 9874ab0b1d4a6752840a9a23bf7502c3e623825f
```

### Configure Options
```
 ./configure --enable-dvbcsa --enable-constcw  --enable-bundle --disable-ffmpeg_static
```
### Set library PATH
```
export LIBRARY_PATH=/opt/vc/lib
```
### Package
```
AUTOBUILD_CONFIGURE_EXTRA="--enable-dvbcsa --enable-constcw  --enable-bundle --disable-ffmpeg_static" ./Autobuild.sh -t debian
```

### Compile
```
make
```

### Test

```
./build.linux/tvheadend
```
### Install Kodi
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:team-xbmc/ppa
sudo apt-get update
sudo apt-get install kodi
```

### Install hts tvheadend client
```
sudo apt-get install kodi-pvr-hts
sudo apt-get install kodi-pvr-iptvsimple
```

### Install FFmpeg
```
sudo apt-get install ffmpeg
ffmpeg -version
```

In Kodi, create a user * with password * 
Provide all streaimg access


In tvh add an IPTV network (no automatic).  Add a mux for that network and use a URL like this....

For version 2.8.15

```
pipe:///usr/bin/ffmpeg -loglevel fatal -re -i http://localhost:9981/stream/channel/6a86ca950c6198ef2c992d4437955e18?ticket=A36A476830B5BFA62BCD75C7B6B5893512ED567 -c:v copy -filter_complex [0:1][0:2][0:3]amerge=inputs=3,pan=5.1|FL=c0|FR=c1|FC=c2|LFE=c3|BL=c4|BR=c5 -c:a eac3 -f mpegts pipe:1
```

(Get the http link openning the link to open the channel)

It'll scan in a new service and you just map that to a new channel.  

### Compile/Install OSCAM-EMU

```
mkdir oscam
cd oscam
git clone https://github.com/oscam-emu/oscam-patched
cd oscam-patched
git checkout oscam11505-emu791
```

Select all protocols and all options
```
make config
```

### Compile oscam
```
make
cd Distribution
mv oscam-1.20_svn11505-791-x86_64-linux-gnu-ssl ./oscam
sudo cp ./oscam /usr/local/bin/oscam
```

### Create Directory: 

```
/home/jordy/.oscam/tmp
```

### Create a directory in: /home/jordy/.oscam and insert the following files:
oscam.conf
```
# oscam.conf generated automatically by Streamboard OSCAM 1.20-unstable_svn SVN r11392
# Read more: http://www.streamboard.tv/svn/oscam/trunk/Distribution/doc/txt/oscam.conf.txt

[global]
logfile                       = stdout
fallbacktimeout               = 3
bindwait                      = 5
initial_debuglevel            = 64
nice                          = -1
maxlogsize                    = 200
waitforcards                  = 0
preferlocalcards              = 1
lb_mode                       = 1
lb_save                       = 100
lb_savepath                   = /home/jordy/.oscam/oscam.stat
lb_auto_timeout_t             = 26000

[cache]

[streamrelay]
stream_relay_enabled          = 0

[dvbapi]
enabled                       = 1
au                            = 1
pmt_mode                      = 4
listen_port                   = 9001
delayer                       = 60
ecminfo_type                  = 4
user                          = tvheadend
read_sdt                      = 1
extended_cw_api               = 1
boxtype                       = pc

[webif]
httpport                      = 8888
httprefresh                   = 2
httppollrefresh               = 2
httpallowed                   = 127.0.0.1,192.168.0.0-192.168.254.254
```

oscam.server

```
[reader]
label                         = emulator
protocol                      = emu
device                        = emulator
caid                          = 090F,0500,1801,0604,2600,FFFF,0E00,4AE1,1010
detect                        = cd
disablecrccws_only_for        = 0E00:000000
ident                         = 0D00:0000C0;0D02:0000A0;0500:023800,021110,007400;0E00:000000;1801:000000;1010:000000
group                         = 1
emmcache                      = 2,0,2,0
emu_auproviders               = 0E00:000000
auprovid                      = 000E00

[reader]
label                         = SoftCam.Key
protocol                      = constcw
device                        = /home/jordy/.oscam/SoftCam.Key
caid                          = 0D00,0D02,090F,0500,1801,0604,2600,FFFF,0E00,4AE1,1010
ident                         = 1801:000000,001101;0604:000000;2600:000000;FFFF:000000;0E00:000000;4AE1:000000;1010:000000
group                         = 1
```
oscam.services

```
# oscam.services generated automatically by Streamboard OSCAM 1.20_svn SVN r11441
# Read more: http://www.streamboard.tv/svn/oscam/trunk/Distribution/doc/txt/oscam.services.txt

[afn]
caid                          = 0E00
provid                        = 000000
srvid                         = 0004,0005,0006,0007,0008,0009,012C,0098,0096,04B1,03E9,04B2,00AD,00B7,00B9,00AF,019E,0193,019D,0192,0191,004A,004C,009D,00A0,004B,0097,000F,0047,0048,0003
```
oscam.user

```
[account]
user                          = tvheadend
pwd                           = tvheadend
monlevel                      = 4
au                            = 1
group                         = 1,2,3,4,5,6,7,8,9,11,12,13,14,15,16,17,18,19,21,22,23,24,25,26,27,28,29,31,32,33,34,35,36,37,38
```

Execute to test:

```
oscam 
```

Create Deamon at run boot
```
sudo apt-get install vim
sudo vim /etc/init.d/oscam
```
Insert the following code

```
#!/bin/sh
### BEGIN INIT INFO
# Provides: oscam
# Required-Start: $local_fs $network $remote_fs
# Required-Stop: $local_fs $network $remote_fs
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop service oscam
# Description: oscam
### END INIT INFO

DAEMON=/usr/local/bin/oscam
PIDFILE=/var/run/oscam.pid
DAEMON_OPTS="-p 4096 -r 2 -B ${PIDFILE} -c /home/jordy/.oscam -t /home/jordy/.oscam/tmp"

test -x ${DAEMON} || exit 0

. /lib/lsb/init-functions

case "$1" in
  start)
    log_daemon_msg "Starting OScam..."
    /sbin/start-stop-daemon --start --quiet --background --name oscam --exec ${DAEMON} -- ${DAEMON_OPTS}
    log_end_msg $?
    ;;
  stop)
    log_daemon_msg "Stopping OScam..."
    /sbin/start-stop-daemon -R 5 --stop --name oscam --exec ${DAEMON}
    log_end_msg $?
    ;;
  restart)
    $0 stop
    $0 start
    ;;
  force-reload)
    $0 stop
    /bin/kill -9 `pidof oscam`
    /usr/bin/killall -9 oscam
    $0 start
    ;;
  *)
    echo "Usage: /etc/init.d/oscam {start|stop|restart|force-reload}"
    exit 1
    ;;
esac
```
To “enable” the script and the starting up of oscam on reboot, use the following commands on Ubuntu/Debian:
```
sudo chmod +x /etc/init.d/oscam
sudo update-rc.d oscam defaults
```
To force detect service
```
sudo systemctl daemon-reload
sudo systemctl enable oscam
sudo systemctl start oscam
```

Enter Tvheadend http://192.168.1.91:9981
```
Configuration/General/Base:User interfase level: Expert
(Cas) Menu should appear
```
