---
title: Using your Android device as a server over WiFi
updated: 2015-12-07
---

## NMap commands
get ip addr from your Android: WiFi Settings -> Advanced -> Scroll all the way down to find IP Address

`nmap xx.x.x.x/24` scans the whole ip range of xx.x.x.0 to xx.x.x.255.  Make sure that your Android device is listed.

once android program runs:

`sudo nmap -PN -p 9009 -sV xx.x.x.x` to verify port 9009 is open on whatever ip addr your Android ahs


