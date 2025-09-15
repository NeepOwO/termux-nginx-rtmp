# termux-nginx-rtmp

This is an `nginx` build for Termux that includes `nginx-rtmp-module`.

## Pre-Requirements

```sh
pkg install root-repo
```
+ termux from f-droid repo (google play is outdated)
+ termux-services
+ openssl-1.1 (legacy)
+ gettext (to inject env variables into nginx config)
```sh
apt install termux-services openssl gettext
```
```sh
ln -s $PREFIX/lib/openssl-1.1/libssl.so.1.1 $PREFIX/lib/libssl.so.1.1
```
```sh
ln -s $PREFIX/lib/openssl-1.1/libcrypto.so.1.1 $PREFIX/lib/libcrypto.so.1.1
```

## Installation

```sh
apt remove nginx # remove any existing nginx installation.
```
```sh
echo "deb https://muxable.github.io/termux-nginx-rtmp/ termux extras" > $PREFIX/etc/apt/sources.list.d/nginx-rtmp.list
```
```sh
apt update --allow-insecure-repositories
```
```sh
apt install nginx-rtmp
```

# Tweak nginx.conf
```sh
curl https://raw.githubusercontent.com/NeepOwO/termux-nginx-rtmp/main/nginx-custom.conf > $PREFIX/etc/nginx/nginx.conf.template
```
```sh
envsubst < $PREFIX/etc/nginx/nginx.conf.template > $PREFIX/etc/nginx/nginx.conf
```
```sh
mkdir -p $PREFIX/www/static/ && curl https://raw.githubusercontent.com/NeepOwO/termux-nginx-rtmp/main/stat.xsl > $PREFIX/www/static/stat.xsl
```
# Restart phone

## Enable and Start Service
```sh
sv-enable nginx
sv up nginx
nginx
```
## test link
```sh
0.0.0.0:8080/stat 
```
# install ffmpeg
```sh
apt install ffmpeg
apt install libexpat
```
# Start stream
## check ip
```sh
ifconfig
```
## streamlink
```sh
rtmp://ip:1935/publish/live
```
## ffmpeg start string

without coding to HEVC
```sh
ffmpeg -i rtmp://localhost:1935/publish/live -c:v copy -c:a copy -f mpegts srt://yourip:yourport?mode=caller 
```

with coding to HEVC
```sh
ffmpeg -i rtmp://localhost:1935/publish/live -c:v libx265 -crf 18 -c:a copy -f mpegts srt://yourip:yourport?mode=caller 
```
# auto ffmpeg restart script

create file
```sh
nano ffmpeg.sh
```

paste script
```sh
while true; do
ffmpeg -i rtmp://localhost:1935/publish/live -c:v copy -c:a copy -f mpegts srt://ip:port?mode=caller
echo "FFmpeg завершился. Перезапуск через 5 секунд..."
sleep 5
done
```
New script
```sh
while true; do
ffmpeg -i rtmp://localhost:1935/publish/live -c:v hevc_mediacodec -b:v 2000k -c:a libopus -b:a 128k -ar 48000 -f mpegts "srt://IP:PORT?latency=2000000"
echo "FFmpeg завершился. Перезапуск через 1 секунд..."
sleep 1
done
```

New script 2
```sh
#!/bin/bash

threshold=0.9

while true; do
    echo "Запуск FFmpeg..."

ffmpeg -i rtmp://localhost:1935/publish/live -c:v hevc_mediacodec -pix_fmt nv12 -b:v 2000k -c:a libopus -b:a 128k -ar 48000 -f mpegts "srt://IP:PORT?latency=2000000&maxbw=0" 2>&1 | while read -r line; do
        echo "$line"

        if echo "$line" | grep -q "X="; then
            value=$(echo "$line" | grep -oP "X=\K[0-9.]+")
            echo "Обнаружено X=$value"
            if (( $(echo "$value < $threshold" | bc -l) )); then
                echo "X меньше порога ($value < $threshold), перезапуск..."
                pkill -f "ffmpeg.*rtmp://localhost:1935/publish/live"
                break
            fi
        fi
    done

    echo "FFmpeg завершился или был остановлен. Перезапуск через 1 секунду..."
    sleep 1
done

```

give rights
```sh
chmod +x ffmpeg.sh
```
start script
```sh
./ffmpeg.sh
```




