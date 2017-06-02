# broadcom-wl Driver for Linux > 4.6*

This is my own version of broadcom driver for the wifi card factory shipped on Acer V5-122P-0869. 

Building and installing is pretty straightforward: make && make_install. Note that you might have to blacklist kernel's standard broadcom's drivers (google blacklist bcma).

While installing standard archlinux with kernel 4.6.x + broadcom driver, wireless was working fine, I had some truble after updating to 4.7.x.

## Motivation
Arch had released kernel 4.7.1 (as of Aug 2016) and with that my driver stopped working. So I tried to recompile broadcom driver again, this time with no success.

Digging into gcc's error log, the variables `IEEE80211_BAND_{2,5,60}GHZ` on broadcom's file at https://github.com/chaws/broadcom-wl/blob/master/src/wl/sys/wl_cfg80211_hybrid.c had no references anymore, neither Kernel's nor broadcom's sources. I then jumped into kernel's differences (4.7.x - 4.6.x) for `include/net/cfg80211.h`, and got surprised that those variables were removed after this commit: https://github.com/torvalds/linux/commit/57fbcce37be7c1d2622b56587c10ade00e96afa3 (changing +200 files in the linux kernel). I mean, out of +30k files in kernel, they had to change the one needed for my wireless card driver...

That change removed the enum `ieee80211_band`, which was sort of an alias to `nl80211_band` at https://github.com/torvalds/linux/blob/master/include/uapi/linux/nl80211.h.

## Changes
This repository consists on changing broadcom's driver in order to conform with Kernel's new[old] `enum nl80211`, which `sed -i 's/IEEE80211_BAND_\(2\|5\|60\)GHZ/NL80211_BAND_\1GHZ/g' src/wl/sys/wl_cfg80211_hybrid.c` should do the job.

## * = to be continued
The driver apparently worked fine after the powerful sed script aforementioned. ping worked fine. nslookup too, now if something's using tcp, kernel panics, so do I!

Right now I'm studying kdump and kexec so I can get kernel's core dump file and see if I'm smart enough to figure out what's going on with broadcom's driver for my wireless.

## References
broadcom's sources originally downloaded from: https://www.broadcom.com/support/802.11 (6.30.223.271) 64bit STA driver as of August 2016.

There is a small discussion on bugzilla: https://bugzilla.kernel.org/show_bug.cgi?id=194903

# Colaborate
Comments and tips are welcome!
