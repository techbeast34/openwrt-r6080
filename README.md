## This repo will not be maintained

Turns out [Alex Lewontin](https://github.com/alexclewontin) had already authored support for the R6080 in February 2020, but the changes weren't committed until June 26th, 2020. That commit can be [seen here](https://github.com/openwrt/openwrt/commit/bd49f2c9848ec10c7c7b41eaa14ac6c26e2bc977). Guess all I had to do was wait a month ¯\\\_(ツ)_/¯

Regardless, it was a fun project. Also, after opening up my R6080, I found that the board still has the contacts necessary to add a USB port, but a lot of SMT components would need to be soldered on. If I manage to track down the components and get my hands on a hot air station, I might attempt to do this down the line.

---
# NETGEAR R6080 Router OpenWRT Firmware

added support for NETGEAR R6080 (in testing). Must be installed via [nmrpflash](https://github.com/jclehner/nmrpflash)

this firmware may work on NETGEAR R6020 routers as well, since they share the same hardware according to [WikiDevi](https://wikidevi.wi-cat.ru/Netgear_R6120). If it doesn't work, you can always unbrick your router with nmrpflash. Interestingly, they also share the same factory firmware.

To make this firmware I modified the R6120 firmware to work on the R6080 models. [Changes can be seen here](https://github.com/techbeast34/openwrt-r6080/commit/65c7164853df43e00831e8263b86f60771a4c910)

---
## What I Did
- Partitioned OpenWRT Firmware for the R6080
- Added LED Support for R6080
- Added Network Support for R6080

## How I Did it
Just to try it, I wanted to see if the R6120 firmware would work on my R6080. Using [nmrpflash](https://github.com/jclehner/nmrpflash) you can attempt to flash just about any firmware, but of course it will fail because there are checks in place.

As such, this did not work. WikiDevi lied a little when it said the hardware was identical to the Netgear R6120. The flash storage is 8MiB on the R6080, whereas the R6120 has 16MiB. So, off I went to resize the partitions for the firmware.

In the midst of trying to figure out the partition sizes I grabbed some [useful debug info](https://raw.githubusercontent.com/techbeast34/openwrt-r6080/master/r6080) while using the debug telnet shell.

After figuring out the [partition sizes and removing USB support](https://raw.githubusercontent.com/techbeast34/openwrt-r6080/65c7164853df43e00831e8263b86f60771a4c910/target/linux/ramips/dts/mt7628an_netgear_r6080.dts) (which might be able to be hacked in if there are solder pads on the board, I'll check as soon as I find my screwdriver kit), I thought that was the end of it. So I used `make menuconfig` to make a .config file for the build, typed in `make`, crossed my fingers and hit enter.

I started to flash the .img file from the build via nmrp, and...

`Ignoring extra upload request`

`Ignoring extra upload request`

Well, it got rejected. Some other safeguard is in place, it doesn't have to do with partition sizes alone. So I took another look at the factory firmware and my build. I looked at both firmwares with my hex editor and noticed something slightly different in the heading: 

**Factory**

![Factory Image Header](https://github.com/techbeast34/openwrt-r6080/blob/master/r6080%20factory%20header.png?)

**First Unworking Build**

![Header For OpenWRT Build](https://github.com/techbeast34/openwrt-r6080/blob/master/r6080%20first%20unworking%20build.png)

Turns out that header describes some Sercomm hardware inside the router, which turns out to also be different between the R6120 and R6080. So I had to change [mt76x8.mk](https://raw.githubusercontent.com/techbeast34/openwrt-r6080/65c7164853df43e00831e8263b86f60771a4c910/target/linux/ramips/image/mt76x8.mk) to match the info in the factory firmware header. `SERCOMM_HWID := CFR` and `SERCOMM_SWVER := 0x0042` (My hex editor rendered that as a unicode "B")

Typed in `make`, and finally tried to flash again.

`Received keep-alive request`

`Remote finished. Closing connection.`

`Reboot your device now`

Success! A reboot and I was able to SSH into my router.

![SSH](https://github.com/techbeast34/openwrt-r6080/blob/master/ssh%20shell.png?raw=true)

After that I used `make menuconfig` to add in LuCI, a web interface for OpenWRT, and rebuilt.
