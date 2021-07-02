# Stagg EKG Plus
This project uses Python to communicate with the [Fellow Stagg EKG+]() over BLE. A lot of the information used to create this program came from [this post](https://www.reddit.com/r/homeassistant/comments/ek47k2/ble_reverse_engineering_a_fellow_stagg_ekg_kettle/) and additional testing I performed.
This can be used with Homebridge and [my `homebridge-kettle` plugin](https://github.com/calvinmclean/homebridge-kettle).

## Getting Started

*Python*
- This project requires Python > 3.5
- You will need to [install `bluepy`](https://github.com/IanHarvey/bluepy) via `sudo pip install bluepy` or by compiling from source.  [This GitHub issue](https://github.com/IanHarvey/bluepy/issues/158) may be helpful if you encounter `bluepy-helper not found` errors.
- You will also need to install python-dotenv 
```sudo pip install python-dotenv```

*Determining MAC address*
- Scan for nearby BLE devices
```sudo timeout -s SIGINT 5s hcitool -i hci0 lescan```
- Find the value with Fellow and copy that for your environment setup

*Environment*
- Rename `.env.sample` to `.env` 
```mv .env.sample .env```
- Alternatively, create a new `.env` `
```touch .env```
- Change the MAC value to the MAC address for your EKG+
- Update the port value to something unique if you have a conflict in your setup

*Starting the server*
```python3 api.py```

*Systemd*
If you run this on a Raspberry Pi alongside Homebridge, clone this repo at `/root/stagg-ekg-plus` and setup this Systemd Service file to easily handle restarts and log to `syslog`.
If you clone this at another location, change the `ExecStart` line to reflect.
```
[Unit]
Description=Python 3 API for Stagg EKG+
After=syslog.target network-online.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/python3 -u /root/stagg-ekg-plus/api.py
Restart=on-failure
RestartSec=10
KillMode=process
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=multi-user.target
```

## More Info
These are some `gattool` commands that were really useful when figuring this stuff out:
```bash
gatttool -b $KETTLE_MAC -I
# connect, authenticate, and turn on then off
> connect
> char-write-cmd 0x000d efdd0b3031323334353637383930313
> char-write-cmd 0x000d efdd0a0000010100
> char-write-cmd 0x000d efdd0a0400000400

# subscribing
> char-write-req 0x000e 0100
> char-write-cmd 0x000d efdd0b3031323334353637383930313233349a6d
```
