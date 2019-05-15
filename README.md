# Running MPD with pulseaudio on Banana Pi M1
## Use cases
- play sound locally via mpd
- play remote (stream sound from laptop to banana)


# MPD 
## Packages
root@bananapi /etc/pulse # dpkg -l | grep mpd  
ii  libmpdclient2                      2.9-1                             armhf        client library for the Music Player Daemon   
ii  mpd                                0.19.1-1.1                        armhf        Music Player Daemon  

## MPD Configuration 
```
root@bananapi /etc # cat /etc/mpd.conf | grep -vP '^#.*' | grep -vP '^\s*$'
music_directory         "/var/lib/mpd/music"
playlist_directory      "/var/lib/mpd/playlists"
db_file                 "/var/lib/mpd/tag_cache"
log_file                "/var/log/mpd/mpd.log"
pid_file                "/run/mpd/pid"
state_file              "/var/lib/mpd/state"
sticker_file            "/var/lib/mpd/sticker.sql"
user                    "mpd"
bind_to_address         "0.0.0.0"
input {
    plugin    "curl"
}
filesystem_charset      "UTF-8"
id3v1_encoding          "UTF-8"
audio_output {
    type     "pulse"
    name     "MPD Pulse Output"
    server   "127.0.0.1"
}
```

# Pulseaudio
## Packages 
root@bananapi /etc/pulse # dpkg -l | grep pulse  
ii  libpulse0:armhf                    5.0-13                            armhf        PulseAudio client libraries  
ii  libpulsedsp:armhf                  5.0-13                            armhf        PulseAudio OSS pre-load library  
ii  pulseaudio                         5.0-13                            armhf        PulseAudio sound server  
ii  pulseaudio-module-zeroconf         5.0-13                            armhf        Zeroconf module for Pulse Audio sound server  
ii  pulseaudio-utils                   5.0-13                            armhf        Command line tools for the PulseAudio sound server  

## Configuration
```
root@bananapi /etc/pulse # cat system.pa | grep -vP '^#.*' | grep -vP '^\s*$' 
.ifexists module-udev-detect.so
load-module module-udev-detect ignore_dB=1
.else
load-module module-detect 
.endif
.ifexists module-esound-protocol-unix.so
load-module module-esound-protocol-unix
.endif
load-module module-native-protocol-unix
load-module module-stream-restore
load-module module-device-restore
load-module module-default-device-restore
load-module module-rescue-streams
load-module module-always-sink
load-module module-suspend-on-idle
load-module module-position-event-sounds
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1;192.168.0.0/16
load-module module-zeroconf-publish
\# important changes:
\# - no auth from localhost + local range (tell ymd to use 127.0.0.1 and not the sockte)
\# - load-module module-udev-detect ignore_dB=1 Fixes this won't fix bug
\# 	https://bbs.archlinux.org/viewtopic.php?id=218731	
\# 	https://bugs.launchpad.net/ubuntu/+source/pulseaudio/+bug/223133
```

# Run pulseaudio in system mode
```
root@bananapi ~ # pulseaudio --system
W: [pulseaudio] main.c: Running in system mode, but --disallow-exit not set!
W: [pulseaudio] main.c: Running in system mode, but --disallow-module-loading not set!
N: [pulseaudio] main.c: Running in system mode, forcibly disabling SHM mode!
N: [pulseaudio] main.c: Running in system mode, forcibly disabling exit idle time!
W: [pulseaudio] main.c: OK, so you are running PA in system mode. Please note that you most likely shouldn't be doing that.
W: [pulseaudio] main.c: If you do it nonetheless then it's your own fault if things don't work as expected.
W: [pulseaudio] main.c: Please read http://pulseaudio.org/wiki/WhatIsWrongWithSystemMode for an explanation why system mode is usually a bad idea.
W: [pulseaudio] authkey.c: Failed to open cookie file '/var/run/pulse/.config/pulse/cookie': No such file or directory
W: [pulseaudio] authkey.c: Failed to load authorization key '/var/run/pulse/.config/pulse/cookie': No such file or directory
W: [pulseaudio] authkey.c: Failed to open cookie file '/var/run/pulse/.pulse-cookie': No such file or directory
W: [pulseaudio] authkey.c: Failed to load authorization key '/var/run/pulse/.pulse-cookie': No such file or directory
```

# Use it from remote (send laptop audio to banana pi; tested on Arch)
- pactl load-module module-tunnel-sink-new server=192.168.10.60 sink_name=banane channels=2 rate=44100
- go to pavucontrol and sink for application (Playback => Chromium => dropdown menu)
- Remove sink: pactl unload-module 32 (32 was printed running the ..sink-new command)

# Debugging tips
- PULSE_SERVER=192.168.10.60 pavucontrol

todo: systemd unit file   
