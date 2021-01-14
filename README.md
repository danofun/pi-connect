# Raspberry Pi streamer for [Tidal](https://github.com/ppy2/ifi-tidal-release), [Spotify](https://github.com/dtcooper/raspotify), [Plex](https://forums.plex.tv/t/plexamp-for-raspberry-pi-release-notes/368282), and [Roon Bridge](https://help.roonlabs.com/portal/en/kb/articles/linux-install#Downloads)



# Ubuntu Server 18.04

We will be using [Ubuntu Server 18.04](http://cdimage.ubuntu.com/ubuntu/releases/bionic/release/) as the OS for our audio streaming Raspberry Pi. I am using a Raspbery Pi 2 and `ubuntu-18.04.5-preinstalled-server-armhf+raspi2.img.xz` but this tutorial should apply to Raspberry Pi's 3 and 4 using the appropriate 32-bit OS.


## Install Required Dependencies

Tidal connect requires libcurl3 and curl. By default, the Ubuntu 18.04 repositories install libcurl4 when curl is installed. The following repository provides a curl version that supports both libcurl3 and libcurl4
```
sudo add-apt-repository ppa:xapienz/curl34
```

Upgrade your server install
```
sudo apt update && sudo apt dist-upgrade -y
```

Install the required dependancies
```
sudo apt install alsa-utils libssl1.0.0 libportaudio2 libflac++6v5 avahi-daemon libavformat57 libavahi-client3 curl
```


## Sound Card Setup

If you are not using a Raspberry Pi DAC or HAT, you can skip this step. If you are, you will need make the device usable in Ubuntu.
I have a HifiBerry Digi+ HAT. In order for Unbuntu to recognize it, edit `/boot/firmware/config.txt` and add:
```
dtoverlay=hifiberry-digi
```

reboot the device
```
sudo reboot
```


## Download this Repository

```
sudo git clone https://github.com/danofun/pi-connect.git /usr/pi-connect
```



# Tidal

[ppy2](https://github.com/ppy2) had compiled the files necessary to turn a Raspberry Pi into a Tidal Connect client. This repo contains a consolidation of this effort.


## Determine Audio Device Name

Run the following command to determine the name of your audio device
```
/usr/pi-connect/tidal/pa-devs-get
```

In sample output below, the name of our devcie is `snd_rpi_hifiberry_digi: HifiBerry Digi HiFi wm8804-spdif-0 (hw:0,0)`. Determine your device name as we will use it in the next step.

```
ubuntu@ubuntu:/usr/pi-connect$ /usr/pi-connect/tidal/pa-devs-get
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.front
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.rear
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.center_lfe
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.side
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround21
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround21
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround40
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround41
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround50
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround51
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.surround71
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.iec958
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.iec958
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.iec958
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.hdmi
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.hdmi
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.modem
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.modem
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.phoneline
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM cards.pcm.phoneline
Cannot connect to server socket err = No such file or directory
Cannot connect to server request channel
jack server is not running or cannot be started
JackShmReadWritePtr::~JackShmReadWritePtr - Init not done for -1, skipping unlock
JackShmReadWritePtr::~JackShmReadWritePtr - Init not done for -1, skipping unlock
device#0=snd_rpi_hifiberry_digi: HifiBerry Digi HiFi wm8804-spdif-0 (hw:0,0)
device#1=sysdefault
device#2=default
device#3=dmix
Number of devices = 4
```


## Tidal Connect Systemd Service

The service description must be adapted to fit your needs. We must change the "--playback-device" to match your system in `/usr/pi-connect/tidal/pi-connect-tidal.service`

```
[Unit]
Description=Tidal Connect Service
[Service]
Restart=on-failure
ExecStart=/usr/pi-connect/tidal/tidal_connect \
                --tc-certificate-path "/usr/pi-connect/tidal/zenstream.dat" \
                --netif-for-deviceid eth0 \
                -f "Raspbery Pi Connect" \
                --codec-mpegh true \
                --codec-mqa false \
                --model-name "Raspberry Pi 2" \
                --disable-app-security false \
                --disable-web-security false \
                --enable-mqa-passthrough false \
                --playback-device "snd_rpi_hifiberry_digi: HifiBerry Digi HiFi wm8804-spdif-0 (hw:0,0)" \
                --log-level 3
User=root
Group=root
RestartSec=1
KillMode=control-group
[Install]
WantedBy=multi-user.target
```


Once edited, copy the file into place
```
sudo cp /usr/pi-connect/tidal/pi-connect-tidal.service /lib/systemd/system/
```

Start the Tidal Connect client 
```
sudo systemctl daemon-reload
sudo systemctl start pi-connect-tidal.service
```
    
Check the status
```
sudo systemctl status pi-connect-tidal.service
```

Optionally, you can have Tidal Connect client start on boot
```
sudo systemctl enable pi-connect-tidal.service
```



# Spotify

[David Cooper](https://github.com/dtcooper) has put together an excellent Spotify Connect client, Raspotify. Installation could not be easier.
```
curl -sL https://dtcooper.github.io/raspotify/install.sh | sh
```



# Roon Bridge

Roon privides an easy installation script. My Raspberry Pi 2 has a armv7hf processor so the below command install Roon Bridge. For the Raspberry Pi 4 (armv8) and other devices, check out the [Roon website](https://roonlabs.com).
```
wget http://download.roonlabs.com/builds/roonbridge-installer-linuxarmv7hf.sh
chmod +x roonbridge-installer-linuxarmv7hf.sh
sudo ./roonbridge-installer-linuxarmv7hf.sh
rm roonbridge-installer-linuxarmv7hf.sh
```



# Plexamp

## server.json file

We need to acquire a server.json file from an existing Plexamp 1.0 install. 
1. Download Plexamp 1.0 for your OS flavor: [Windows](http://web.archive.org/web/20180109153237/https://plexamp.plex.tv/plexamp.plex.tv/Plexamp%20Setup%201.0.0.exe), [MacOS](http://web.archive.org/web/20180109153237/https://plexamp.plex.tv/plexamp.plex.tv/Plexamp-1.0.0.dmg), [Linux](http://web.archive.org/web/20180829181913/https://plexamp.plex.tv/plexamp.plex.tv/plexamp-1.0.5-x86_64.AppImage)
1. Once Plexamp is installed, sign-in and acquire the server.json file that was just created. MacOS location is `~/Library/Application Support/Plexamp/server.json`
1. Place the server.json file on the Raspberry Pi in folder `/home/ubuntu/.config/Plexamp/`
1. On the device from which you acquired the server.json file, sign out and back into your existing Plexamp install in order to generate a new identifier/token.


## Install Node 9

```
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt install -y nodejs
```


## Plexamp Connect Systemd Service

Copy the service file into place
```
sudo cp /usr/pi-connect/plexamp/pi-connect-plexamp.service /lib/systemd/system/
```

Start the Plexamp client 
```
sudo systemctl daemon-reload
sudo systemctl start pi-connect-plexamp.service
```
    
Check the status
```
sudo systemctl status pi-connect-plexamp.service
```

Optionally, you can have Plexamp client start on boot
```
sudo systemctl enable pi-connect-plexamp.service
```