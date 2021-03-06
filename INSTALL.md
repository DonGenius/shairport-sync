Simple Installation Instructions
==
Here are simple instructions for building and installing Shairport Sync on a Raspberry Pi B, 2B, 3B or 3B+. It is assumed that the Pi is running Raspbian Stretch Lite – a GUI isn't needed, since Shairport Sync runs as a daemon program. For a more thorough treatment, please go to the [README.md](https://github.com/mikebrady/shairport-sync/blob/master/README.md#building-and-installing) page.

In the commands below, note the convention that a `#` prompt means you are in superuser mode and a `$` prompt means you are in a regular non-priviliged user mode. You can use `sudo` *("SUperuser DO")* to temporarily promote yourself from user to superuser, if permitted. For example, if you want to execute `apt-get update` in superuser mode and you are in user mode, enter `sudo apt-get update`.

### Configure and Update
Do the usual update and upgrade:
```
# apt update
# apt upgrade
# rpi-update
``` 
**Note:** At this time of writing (September 2018), it is a good idea to update the Pi's firmware using `rpi-update` because a [significant improvement](https://github.com/raspberrypi/firmware/commit/200c2f4dd54b2048b5dcb8661ea3f232beb7d81e) has been made to the [timing software](https://github.com/raspberrypi/firmware/issues/1026) of the built-in audio DAC's drivers. It should be incorporated in firmware from Raspbian 4.14.66-v7 onwards.

(Separately, if you haven't done so already, consider using the `raspi-config` tool to expand the file system to use the entire card.)

### Activate the Improved Audio Driver
Check the file `/boot/config.txt` and, if it's not there already, add the following line and reboot afterwards:
```
audio_pwm_mode=2

(Note that this isn't needed in the most recent versions of Raspbian as it will enable this driver mode by default)
```

### Turn Off WiFi Power Management
If you are using WiFi, you should turn off WiFi Power Management:
```
# iwconfig wlan0 power off
```
WiFi Power Management will put the WiFi system in low-power mode when the WiFi system is considered inactive, and in this mode it may not respond to events initiated from the network, such as AirPlay requests. Hence, WiFi Power Management should be turned off. (See [TROUBLESHOOTING.md](https://github.com/mikebrady/shairport-sync/blob/master/TROUBLESHOOTING.md#wifi-adapter-running-in-power-saving--low-power-mode) for more details.)

Reboot the Pi.

### Remove Old Copies and Old Startup Scripts
Before you begin building Shairport Sync, it's best to search for and remove any existing copies of the application, called `shairport-sync`. Use the command `$ which shairport-sync` to find them. For example, if `shairport-sync` has been installed previously, this might happen:
```
$ which shairport-sync
/usr/local/bin/shairport-sync
```
Remove it as follows:
```
# rm /usr/local/bin/shairport-sync
```
Do this until no more copies of `shairport-sync` are found.

You should also remove the initialisation script files `/etc/systemd/system/shairport-sync.service` and `/etc/init.d/shairport-sync` if they exist – new ones will be installed in necessary.

### Build and Install
Okay, now let's get the tools and sources for building and installing Shairport Sync.

First, install the packages needed by Shairport Sync:
```
# apt install build-essential git xmltoman autoconf automake libtool libdaemon-dev \
    libpopt-dev libconfig-dev libasound2-dev avahi-daemon libavahi-client-dev libssl-dev
```
Next, download Shairport Sync, configure it, compile and install it:
```
$ git clone https://github.com/mikebrady/shairport-sync.git
$ cd shairport-sync
$ autoreconf -fi
$ ./configure --sysconfdir=/etc --with-alsa --with-avahi --with-ssl=openssl --with-systemd
$ make
$ sudo make install
```
By the way, the `autoreconf` step may take quite a while on a Raspberry Pi -- be patient!

Now to configure Shairport Sync. Here are the important options for the Shairport Sync configuration file at `/etc/shairport-sync.conf`:
```
// Sample Configuration File for Shairport Sync on a Raspberry Pi using the built-in audio DAC
general =
{
  // drift_tolerance_in_seconds = 0.010; // this is no longer necessary if you have updated the Pi's firmware using rpi-update (see above).
  volume_range_db = 50;
};

alsa =
{
  output_device = "hw:0";
  mixer_control_name = "PCM";
};

```
The next step is to enable Shairport Sync to start automatically on boot up:
```
# systemctl enable shairport-sync
```
Finally, either reboot the Pi or start the `shairport-sync` service:
```
# systemctl start shairport-sync
```
The Shairport Sync AirPlay service should now appear on the network with a service name made from the Pi's hostname with the first letter capitalised, e.g. hostname `raspberrypi` gives a service name `Raspberrypi`. You can change the service name and set a password in the configuration file.

Connect and enjoy...
