---
title: "It's Just a Matter of Time"
type: post
author: jtbouse
date: 2021-04-12T22:12:02-04:00
publishdate: 2021-04-12
draft: true
summary: |
    After several years of faithful service, I decided to take another shot at
    my stratum-1 NTP time server running off a Raspberry Pi 2 that I had originally
    built back in late '16 to keep the time on all my servers in sync. That had
    really been my first Raspbery Pi project and it had several issues that left 
    me not 100% satisfied with the results.
tags:
    - ntp
    - nmea
    - gps
    - raspberry pi
categories:
    - Projects
slut: matter-of-time
---

## Let's go back in time

{{< img src="images/20160817_192541.jpg" alt="Original Raspberry Pi NTP build" class="alignleft" >}}
Back in August of 2016 I had built my original NTP server with a Raspberry Pi 2 that I had purchased
at my local MicroCenter. I had then ordered the original [Adafruit Ultimate GPS HAT][bom-1] from
Amazon along with an RF adapter cable that I later determined was the incorrect one I needed and had
to find a way to work around.

{{< img src="images/20210411_000102.jpg" alt="New Raspberry Pi NTP build" class="alignright" >}}
This was my first time trying to solder a 2x20 header so it wasn't the best job and I think it may
have contributed to some of the performance issues I encountered with the project. There was some
general instability that would require me to routinely reboot the system to reset the board. The
other factor was that the default NTP daemon package available with that version of Raspberry Pi OS,
called Raspbian at the time but has since been renamed, did not support the GPS NMEA reference clock
needed to use the Ultimate GPS HAT.

Besides that there wasn't much thought that went into the construction of the build. I used the 
simple Official Red & White Raspberry Pi case which was not built with the external GPS antenna 
in mind so I had to just leave the RF adapter cable exit the case between the USB ports. While this
worked, it was not "pretty" and it provided no support so any tension on the adapter cable went
directly to the u.FL connector on the Ultimate GPS HAT risking potential damage. I had also made the
mistake of purchasing an RP-SMA to u.FL cable rather than a [SMA to u.FL cable][bom-3] which required me to
get an additional RP-SMA to SMA adapter between the Raspberry Pi and the [external GPS antenna][bom-2].

So I wanted to come up with something more purpose built and look more like an appliance ready for 
future use. I found that [Adafruit][1] had a great enclosure kit that had the holes perfect for the
RF adapter jack to be secured and looked how I thought an appliance box should. So I started the new
build project with the purchase of the [enclosure kit][bom-4]. Since I already had the Raspberry Pi
board and the microSD card I didn't need to purchase those but if not I could have gotten a new
[Raspberry Pi 3][bom-5]. I had been making use of an old phone USB charging adapter to power my
Raspberry Pi, with the new enclosure I went ahead and purchase a [5V power supply][bom-6] as I had
began noticing that I was getting under-voltage messages in the logs.

While I purchased a new [Ultimate GPS HAT][bom-1] due to my displeasure with the solder work and I
had found that Adafruit now had a [solderless header][bom-8] option with a [jig kit][bom-7]. The
Ultimate GPS HAT does come with a 2x20 header that requires soldering and the solderless jig kit does
include a 2x20 solderless header so I got the kit and if I build another box later I can just get the
header and reuse the jig. The final missing part to polish the finished product was getting a kit of
[nylon screws and stand-offs][bom-9]. I picked the black nylon stand-offs rather than the white just
for personal preference and availbility which ensured that the HAT and the Raspberry Pi are securely
supported more than by the header alone.

## Putting together the pieces

{{< img src="images/20210410_235419.jpg" alt="Board assembly" class="alignleft" >}}
The first step in the new build was to take the [Ultimate GPS HAT][bom-1] board and the 2x20
[solderless header][bom-7] from the kit. Using the installation jig from the kit I secured the
header and to the HAT. I then took four 12mm long M-F hex standoffs and 4 M2.5 x 4mm screws from the
[screw and stand-off kit][bom-9] to assemble the Raspberry Pi and Ultimate GPS HAT together to the
base plate of the [enclosure kit][bom-4]. With the boards secured to the base plate I just needed to
attach the [RF adapter cable][bom-3] to the u.FL connector on the HAT to complete the assembly.

{{< img src="images/20210410_235912.jpg" alt="Enclosure assembly" class="alignright" >}}
The enclosure then just slides over the base plate assembly and the RF adapter cable jack secured
through the top left hole. The enclosure kit includes several rubber plugs to fill the other 3 holes
that were not going to be used to help keep dust out of the enclosure. Securing the top of the case
on the enclosure completes the build. With the hardware fully assembled it was time to move on to
the software phase of the build.

The first step in the software phase was to install the [Raspberry Pi OS][2] on the microSD card. 
The easiest way to perform this is with the Raspberry Pi Imager application which will automatically
download the image and write it to the microSD card then verify it. The new version 1.6 has recently
added an advanced configuration option available by hitting CTRL+Shift+X which allows you to set
the hostname, password, enable SSH and configure WiFi to assist in being able to bring the Raspberry
Pi online and ready to be configured remotely without having to have a monitor and keyboard.

## Ready for some Pi?

So there are plenty of Raspberry Pi install guides out there so I won't go deep into how to perform
the initial install. You do need to be able to access the command line so if you do not enable SSH
before bootstrapping then you will need a monitor and keyboard to perform the initial installation.
It really does not matter if you use wifi or ethernet, though an ethernet connection will most likely
be a more stable connection. As for the Raspberry Pi OS, I choose to use the `Raspberry Pi Lite`
over the Raspberry Pi Full or Desktop. The reasoning for this was simply to install the smallest
footprint possible from a security standpoint and just not wanting to install unnecessary desktop
environment.

Once the Raspberry Pi has bootstrapped and is running the first thing you want to do is edit the
`/boot/config.txt`. You want to comment out the line to disable `audio` as shown below on line 57. 
Next you want to add the dtoverlay for `pi3-disable-bt` and `pps-gpio` as shown below on lines 59-60.
The GPIO used on the Ultimate GPS HAT is Pin 4, this needs to be included as it is not the default 
for the pps-gpio overlay.

{{< highlight ini "linenostart=54,hl_lines=4 6-7" >}}
# Additional overlays and parameters are documented /boot/overlays/README

# Enable audio (loads snd_bcm2835)
# dtparam=audio=on

dtoverlay=pi3-disable-bt
dtoverlay=pps-gpio,gpiopin=4

[pi4]
# Enable DRM VC4 V3D driver on top of the dispmanx display stack
dtoverlay=vc4-fkms-v3d
max_framebuffers=2

[all]
#dtoverlay=vc4-fkms-v3d
{{< /highlight >}}

The other system level configuration to make is to edit the `/boot/cmdline.txt` and remove all
reference to the serial console. This is because the Ultimate GPS HAT will be using the serial
port to communicate. You will generally see the `/boot/cmdline.txt` will contain something that
looks similar to the following:

{{< highlight conf >}}
console=serial0,115200 console=tty1 root=PARTUUID=cead1835-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
{{< /highlight >}}

Which I then editted to look similar to:

{{< highlight conf >}}
console=tty1 root=PARTUUID=cead1835-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait plymouth.ignore-serial-consoles nhoz=off
{{< /highlight >}}

{{< img src="images/ntp-gps-udev-rules.png" alt="GPS udev rules" class="alignleft" >}}

The next thing to get in place is the udev rule to set up the symlinks for our GPS device. Using
your favorite editor you can create the `/etc/udev/rules.d/99-gps.rules` file with the following:

{{< highlight shell >}}
KERNEL=="pps0", SYMLINK+="gpspps0"
KERNEL=="ttyAMA0", SUBSYSTEM=="tty", MODE=="0777", SYMLINK+="gps0"
{{< /highlight >}}

Now we can move to installing the actual software. Thankfully this is all available through the
software repository and able to use `apt` to install it. It is generally a good idea to make sure
you have everything up to date as well so here are the commands I executed:

{{< highlight conf >}}
sudo apt update
sudo apt upgrade
sudo apt install pps-tools setserial ntp ntpdate
{{< /highlight >}}

We also want to help speed up the boot time so let's disable the serial console related services

{{< highlight conf >}}
sudo systemctl disable hciuart
sudo systemctl disable serial-getty@ttyAMA0.service
{{< /highlight >}}

Another item is to disable the restarting of NTP by the DHCP client. The easist way to perform
this is to simply remove those files so you want to execute the following:

{{< highlight conf >}}
sudo rm /etc/dhcp/dhclient-exit-hooks.d/ntp
sudo rm /var/lib/ntp/ntp.conf.dhcp
sudo rm /lib/dhcpcd/dhcpcd-hooks/50-ntp.conf
{{< /highlight >}}

{{< img src="images/ntp-gps-rc-local.png" alt="GPS udev rules" class="alignright" >}}

The last piece we want to setup is performing the initial serial communication settings with the
GPS so we'll open up the `/etc/rc.local` in our editor and add the highlight lines 20-28.

{{< highlight shell "linenostart=14,hl_lines=7-15" >}}
# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
fi

systemctl stop ntp.service
setserial /dev/ttyAMA0 low_latency
/bin/echo -e '$PMTK314,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0*29\r\n' > /dev/ttyAMA0 # diable all except GPMRC and GPGGA
/bin/echo -e '$PMTK320,0*26\r\n' > /dev/ttyAMA0 # power saving mode off
/bin/echo -e '$PMTK301,0*2C\r\n' > /dev/ttyAMA0 # No DGPS source
/bin/echo -e '$PMTK313,0*2F\r\n' > /dev/ttyAMA0 # disable SBAS satellite
/bin/echo -e '$PMTK251,115200*1F\r\n' > /dev/ttyAMA0 #set baud to 115200
stty -F /dev/ttyAMA0 raw 115200 cs8 clocal -cstopb -echo # set serial baud to 115200
systemctl start ntp.service

exit 0
{{< /highlight >}}

At this point it is a good idea to give your Raspberry Pi a reboot to enable it to start up with
this configuration in place. We don't yet have NTP configured to use the GPS input but restarting
now will bring everything up with the hardware configured and we should be ready to configure NTP.

## The time is nigh

When our Raspberry Pi has come back online after being restarted and we have connected back to
it, either via monitor and keyboard or over SSH, we will want to confirm things are in fact working.
The first is to make sure that PPS device is responding, we will do this using the `ppstest` utility
we installed earlier.

{{< highlight bash "linenos=false" >}}
pi@raspberrypi:~ $ sudo ppstest /dev/pps0
trying PPS source "/dev/pps0"
found PPS source "/dev/pps0"
ok, found 1 source(s), now start fetching data...
source 0 - assert 1618365982.999996770, sequence: 2337 - clear  0.000000000, sequence: 0
source 0 - assert 1618365983.999994313, sequence: 2338 - clear  0.000000000, sequence: 0
source 0 - assert 1618365984.999994148, sequence: 2339 - clear  0.000000000, sequence: 0
source 0 - assert 1618365985.999993720, sequence: 2340 - clear  0.000000000, sequence: 0
{{< /highlight >}}

You can hit CTRL+C to cancel this after confirming it works. If you get a `Time out` message then
your GPS has most likely not locked on to the satellites yet so you need to attempt repositioning
your antenna to have better access to the sky.

Next we can actually confirm we're getting the GPS data. You can do this very easily using `cat`.

{{< highlight bash "linenos=false,hl_lines=2 4 6 8" >}}
pi@raspberrypi:~ $ cat /dev/gps0
$GPGGA,021742.000,2810.6866,N,08124.9390,W,1,11,0.83,16.4,M,-30.6,M,,*65
$GPGSA,A,3,12,06,23,15,02,13,05,25,29,18,20,,1.44,0.83,1.17*0D
$GPRMC,021742.000,A,2810.6866,N,08124.9390,W,0.02,49.30,140421,,,A*49
$GPVTG,49.30,T,,M,0.02,N,0.04,K,A*05
$GPGGA,021743.000,2810.6866,N,08124.9390,W,1,11,0.83,16.4,M,-30.6,M,,*64
$GPGSA,A,3,12,06,23,15,02,13,05,25,29,18,20,,1.44,0.83,1.18*02
$GPRMC,021743.000,A,2810.6866,N,08124.9390,W,0.03,223.42,140421,,,A*72
$GPVTG,223.42,T,,M,0.03,N,0.05,K,A*3E
{{< /highlight >}}

The lines we are looking for most are the `$GPGGA` and `$GPRMC` prefixed ones that I have
highlighted. The lines are comma-delimited and the `$GPGGA` line will indicate how many
satellites you are locked onto in the 8th column. In this example we are locked on 11 
satellites and I typically find my system is locked on anywhere from 6-13 satellites with
9-11 as an average. If you are curious to understand these lines I found a [great site][3]
that you can paste the line into and see what it is actually telling you.

{{< img src="images/ntp-gps-ntp-conf.png" alt="GPS udev rules" class="alignleft" >}}

If you're getting positive results to this point it is finally time to tie it all together
and get NTP using this data. At this point by default your NTP server will be running and
using external NTP pool servers to get peer with.

At this point we need to edit the `/etc/ntp.conf` file. Below you will find the configuration
I have in place and I have highlighted the important lines you will need to modify or add
if missing from your own.

While getting things initially setup you may wish to uncomment line 9 to allow NTP to record
the statistics. In particular the `clockstats` will be very helpful to confirm that NTP is
receiving the GPS data. I would however probably turn this off once confident in the setup
as this would add additional writes to the microSD card which can lower the usefulness of the
drive and fill up the space if you do not monitor it.

{{< highlight shell "hl_lines=9 46 62-65">}}
# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help

driftfile /var/lib/ntp/ntp.drift

# Leap seconds definition provided by tzdata
leapfile /usr/share/zoneinfo/leap-seconds.list

# Enable this if you want statistics to be logged.
#statsdir /var/log/ntpstats/

statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable


# You do need to talk to an NTP server or two (or three).
#server ntp.your-provider.example

# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
# pick a different set every time it starts up.  Please consider joining the
# pool: <http://www.pool.ntp.org/join.html>
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst


# Access control configuration; see /usr/share/doc/ntp-doc/html/accopt.html for
# details.  The web page <http://support.ntp.org/bin/view/Support/AccessRestrictions>
# might also be helpful.
#
# Note that "restrict" applies to both servers and clients, so a configuration
# that might be intended to block requests from certain clients could also end
# up blocking replies from your own upstream servers.

# By default, exchange time with everybody, but don't allow configuration.
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited

# Local users may interrogate the ntp server more closely.
restrict 127.0.0.1
restrict ::1

# Needed for adding pool entries
restrict source  limited kodnotrap nomodify noquery

# Clients from this (example!) subnet have unlimited access, but only if
# cryptographically authenticated.
#restrict 192.168.123.0 mask 255.255.255.0 notrust


# If you want to provide time to your local subnet, change the next line.
# (Again, the address is an example only.)
#broadcast 192.168.123.255

# If you want to listen to time broadcasts on your local subnet, de-comment the
# next lines.  Please do this only if you trust everybody on the network!
#disable auth
#broadcastclient

server 127.127.20.0 mode 83 minpoll 4 maxpoll 4 prefer
fudge 127.127.20.0 flag1 1 flag4 1 time2 0.350 refid GPS

tos mindist 0.002
{{< /highlight >}}

On line 46 I include the `limited kod` options to add more security to the configuration and
limit the changes that can be made from the peer servers.

Lines 62-63 are where we actually add the GPS as our reference clock source. The `127.127.20.0`
address is the internal address for the [GPS_NMEA][4] reference clock and will point to the
/dev/gps0 and /dev/gpspps0 devices from our udev rules. The `mode` we set on line 62 tells NTP
that we are looking to process `$GPRMC` and `$GPGGA` data lines only and we are communicating
at 115200 baud over the serial port. The `flag1` setting on line 63 signals that we want to
enable PPS signal processing. The `flag4` setting is to obscure the GPS location which would be
stored in the `clockstats` statistics file and isn't really needed to operate. The `time2` setting
is the serial EOL offset and seems to be fine for Raspberry Pi serial communications. The `tos`
command on line 65 is to set the minmimum distance used but the selection and anticlockhop
algorithm. Setting this to `0.002` is just slightly more than the default of `0.001` which seems
to work excellent with the PPS signal on the Ultimate GPS HAT.

{{< img src="images/ntp-gps-stratum-1.png" alt="GPS udev rules" class="alignright" >}}

With the `ntp.conf` configuration saved you should be ready to restart the NTP service and after
given a bit of time to stablize you should be able to query the NTP server and see that it is
using the GPS data.

{{< highlight conf >}}
sudo systemctl restart ntp.service
ntpq -crv -pn
{{< /highlight >}}

If everything is working properly you are expecting to see `refid=GPS` and `stratum=1` in the 
`ntpq` output. As well you are looking to see an `o` to the left of the `127.127.20.0` peer
entry with the `delay`, `offset` and `jitter` column values as close to `0.000` as possible.
This will indicate that your time is stable. A `reach` column value of `377` indicates that 
there have been no  polling attempt failures.

## That's a wrap

At this point you will have successfully setup your stratum-1 NTP time server and can begin to
point all your servers and workstations to it in order to receive time updates. This is where
I would give one last `reboot test` and bounce the server to ensure that everything comes back
up as expected. I always say it is not done unless it has been rebooted. So give it one more 
reboot and double check that everything comes up right.

All in all, I think that total cost for this NTP server project build was around $150-160, which
includes the extras that can be used with future projects. While the case and everything makes
mention of a Raspberry Pi 3, I did reuse my existing Raspberry Pi 2 which does not include wifi
and bluetooth. My existing Raspberry Pi 3 board is already in use with another build project
which is why I did not use it and I did not think it worth buying a new one when this worked fine.
The original Ultimate GPS HAT I bought did have a reported issue with the Raspberry Pi 3 but that
appears to have been resolved and a new revision of the board released.

Overall I am quite happy with how this project turned out and the results have been extremely
more stable than my original build. I expect this device to see a long lifetime of use as I do
like to keep my timestamps accurate.

[1]: https://www.adafruit.com/ "Adafruit Industries"
[2]: https://www.raspberrypi.org/software/ "Raspberry Pi Software"
[3]: https://rl.se/gprmc "GPRMC & GPGGA decode"
[4]: https://www.eecis.udel.edu/~mills/ntp/html/drivers/driver20.html "NTP Generic NMEA GPS Receiver"

[bom-1]: https://www.adafruit.com/product/2324 "Adafruit Ultimate GPS HAT for Raspberry Pi A+/B+/Pi 2/3/Pi 4 - Mini kit"
[bom-2]: https://www.adafruit.com/product/960 "GPS Antenna - External Active Antenna 28dB 5 Meter SMA"
[bom-3]: https://www.adafruit.com/product/851 "SMA to uFL/u.FL/IPX/IPEX RF Adapter Cable"
[bom-4]: https://www.adafruit.com/product/4283 "Pilot Gateway Pro LoRa Enclosure Kit for Raspberry Pi 3 - RAK7243"

[bom-5]: https://www.adafruit.com/product/3055 "Raspberry Pi 3 - Model B - ARMv8 with 1G RAM"
[bom-6]: https://www.adafruit.com/product/1995 "5V 2.5A Switching Power Supply with 20AWG MicroUSB Cable"
[bom-7]: https://www.adafruit.com/product/3413 "GPIO Hammer Headers - Solderless Raspberry Pi Connectors - Male + Female + Installation Jig"
[bom-8]: https://www.adafruit.com/product/3663 "Hammer Header Female - Solderless Raspberry Pi Connector"
[bom-9]: https://www.adafruit.com/product/3299 "Black Nylon Screw and Stand-off Set - M2.5 Thread"