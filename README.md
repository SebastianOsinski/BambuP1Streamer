# Credit

Thanks goes to slynn1324 for putting this all together.

# Bambu P1 Camera Streamer

Only tested on a P1S. I would expect it to work for a p1p camera. I would not expect this to work on an X1/X1C - the codecs are different and I don't believe that local network streaming is enabled. 

Built and tested on Debian 12 / amd64. Other platforms may not work.

Derived from https://github.com/hisptoot/BambuSource2Raw.  

https://github.com/AlexxIT/go2rtc does most of the work.

# AUTOMATED BUILDING

This section is for the TL;DR approach.

## Quickstart

This is an example on how to get the stack running ASAP. In this example we use Podman (as the more secure Docker replacement), Debian / Ubuntu and Chromium.

```
sudo apt update
sudo apt install podman
git clone https://github.com/slynn1324/BambuP1Streamer.git
cd BambuP1Streamer
podman build -t bambu_p1_streamer .
podman run --name bambu_p1_streamer -p 1984:1984 -e PRINTER_ADDRESS=10.1.1.13 -e PRINTER_ACCESS_CODE=24952313 localhost/bambu_p1_streamer
open http://localhost:1984
```


# MANUAL BUILDING

This section is going through all the steps manually one by one.

## DEPENDENCIES

Bambu Studio Proprietary Plugin Library
```
wget https://public-cdn.bambulab.com/upgrade/studio/plugins/01.04.00.15/linux_01.04.00.15.zip
unzip linux_01.04.00.15.zip
```

Go2Rtc
```
wget https://github.com/AlexxIT/go2rtc/releases/download/v1.6.2/go2rtc_linux_amd64
chmod a+x go2rtc_linux_amd64
```

## BUILD
replace `podman` with `docker` if that's what you're running. 

### build binary
```
podman run --rm -v $(pwd):/work docker.io/gcc:12 gcc /work/src/BambuP1Streamer.cpp -o /work/BambuP1Streamer 
```

### build container
```
podman build -t bambu_p1_streamer .
```

## RUN
Plug in the right values for the environment variables
```
podman run -d --name bambu_p1_streamer -p 1984:1984 -e PRINTER_ADDRESS=192.168.12.34 -e PRINTER_ACCESS_CODE=12345678 bambu_p1_streamer
```

## ACCESS
### Index Page (only the MJPEG parts will work)
```
http://<host>:1984/links.html?src=p1s
```

### MJPEG url
```
http://norm:1984/api/stream.mjpeg?src=p1s
```

### WebSocket
go2rtc has a unique feature for "mjpeg-over-websocket" that may demonstrate lower latency and better control than a regular MJPEG image in a browser.  This however will require creating a custom player (TODO) to leverage, but could better emulate a video control. 

WebSocket url:
```
ws://<host>:1984/api/ws?src=p1s
```

WebSocket pseudo-code:
```
1) connect to websocket
2) send 'mjpeg'
3) receive 'mjpeg'
4) receive binary messages with each frame
	update displayed impage with received data (data/base64 url)
5) disconnect web socket to stop
```

# Troubleshooting

### Bambu_Open failed: 0xffffff9b
This error on the containers output seems to occur when the PRINTER_ADDRESS destination is somehow unreachable.

### [1] [BambuTunnel::read_sample] error [1]
This error on the containers output seems to occur when the PRINTER_ACCESS_CODE is not correct.

### Harmless errors on http://localhost:1984/stream.html?src=p1s
Sometimes a few error messages appear before the stream starts. One would just wait a few more seconds.
