# Kismet 2018-07-BETA1

https://www.kismetwireless.net

## MAJOR KISMET UPDATE

Welcome to the new, MAJOR rewrite of Kismet!  This version changes almost everything, hopefully for the better, including:

* Web-based UI allowing for much simpler presentation of data and compatibility with mobile devices

* Standard JSON-based data export for easy scripting against Kismet instances

* Support for wireless protocols beyond Wi-Fi, like basic Bluetooth scanning, thermometer and weather station detection with the RTL-SDR hardware, and more on the way

* New remote-capture code optimized for binary size and RAM, allowing extremely low-end embedded devices to be used for packet capture

* New logging format which can encapsulate complex information about devices, system state, alerts, messages, and packets in a single file with simple tools for extracting standard formats

* Pcap-NG multi-interface logs with complete original headers, readable by Wireshark and other tools

Please remember that as a pre-release, there is still some to be done and warts to be taken care of, but the codebase has been under development and in use for several years and has been performing well.

Setting up the new Kismet is generally simpler than the older versions, keep reading for more information!

At the *very least* you will need to uninstall any old versions of Kismet, and you will need to install the new config files with `make forceconfigs`.  Read on for more info!

## TOC

[TOC]

## Quick Setup

Kismet has many configuration knobs and options; but for the quickest way to get the basics working:

1. Uninstall any existing Kismet installs.  If you installed Kismet using a package from your distribution, uninstall it the same way; if you compiled it yourself, be sure to remove it

2. Install dependencies.  Kismet needs a number of libraries and  development headers to compile; these should be available in nearly all distributions.

   *Ubuntu/Debian/Kali/Mint*

   ```bash
   $ sudo apt-get install build-essential git libmicrohttpd-dev pkg-config zlib1g-dev libnl-3-dev libnl-genl-3-dev libcap-dev libpcap-dev libncurses5-dev libnm-dev libdw-dev libsqlite3-dev libprotobuf-dev libprotobuf-c-dev protobuf-compiler protobuf-c-compiler libsensors4-dev
   ```

   On some older versions, `libprotobuf-c-dev` may be called `libprotobuf-c0-dev`.For the Python add-ons, you will also need:

   ```bash
   $ sudo apt-get install python python-setuptools python-protobuf python-sqlite python-requests
   ```

   You can also use the `pip` equivalents of the python libraries, so long as they're installed in a location your normal Python interpreter can find them.

   For rtlsdr rtl_433 support, you will also need:

   ```bash
   $ sudo apt-get install librtlsdr0
   ```
   as well as the rtl_433 tool from https://github.com/merbanan/rtl_433 if it is not otherwise provided by your distribution.

3. Clone Kismet from git.  If you haven't cloned Kismet before:
   ```bash
    $ git clone https://www.kismetwireless.net/git/kismet.git
   ```

    If you have a Kismet repo already:

    ```bash
   $ cd kismet
   $ git pull
    ```

4. Run configure.  This will find all the specifics about your system and prepare Kismet for compiling.  If you have any missing dependencies or incompatible library versions, they will show up here.
   ```bash
   $ cd kismet
   $ ./configure
   ```

   Pay attention to the summary at the end and look out for any warnings! The summary will show key features and raise warnings for missing dependencies which will drastically affect the compiled Kismet.

5. Compile Kismet.
   ```bash
   $ make
   ```

   You can accelerate the process by adding '-j #', depending on how many CPUs you have.  For instance on a quad-core system:
   ```bash
   $ make -j4
   ```
   
   C++ uses quite a bit of RAM to compile, so depending on the RAM available on your system you may need to limit the number of processes you run simultaneously.

6.  Install Kismet.  Generally, you should install Kismet as suid-root; Kismet will automatically add a group and install the capture binaries accordingly.
  
   When installed suid-root, Kismet will launch the binaries which control the channels and interfaces with the needed privileges, but will keep the packet decoding and web interface running without root privileges.
   ```bash
   $ sudo make suidinstall
   ```
   
7.  Put yourself in the Kismet group.
   ```bash
   $ sudo usermod -a -G kismet foouser
   ```
   This will add 'foouser' to the Kismet group.

8.  Log out and back in.  Linux does not update groups until you log in; if you have just added yourself to the Kismet group you will have to re-log in.
9.  
   Check that you are in the Kismet group with:
   ```bash
   $ groups
   ```
   If you are not in the `kismet` group, you should log out entirely, or reboot.

9.  You're now ready to run Kismet!  Point it at your network interface... Different distributions (and kernel versions, and distribution versions) name interfaces differently; your interface may be `wlan0` or `wlan1`, or it may be named something like `wlp0s1`, or it may be named using the MAC address of the card and look like `wlx00c0ca8d7f2e`.

   You can now start Kismet with something like:
   ```bash
   $ kismet -c wlan0
   ```
   
   *or*, you can just launch Kismet and then use the new web UI to select the card you want to use, by launching it with just:
   ```bash
   $ kismet
   ```
   
   just remember, until you add a data source, Kismet will not be capturing any packets!

   *THE FIRST TIME YOU RUN KISMET*, it will generate a new, random password for your web interface.

   This password can be found in the config file: ~/.kismet/kismet_httpd.conf which is in the *home directory of the user* starting Kismet.

   If you start Kismet as or via sudo (or via a system startup script where it runs as root), this will be in *roots* home directory.

  You will need this password to control Kismet from the web page - without it you can still view information about devices, view channel allocations, and most other actions, but you CAN NOT control Kismet data sources, view pcaps, or perform other actions.

10.  Point your browser at http://localhost:2501

   You will be prompted to do basic configuration - Kismet has many options in the web UI which can be tweaked.

   To use all the features of the Kismet web UI, put in the password found in `~/.kismet/kismet_httpd.conf`

## Debugging Kismet

Kismet (especially in beta) is in a state of rapid development - this means that bad things can creep into the code.  Sorry about that!

If you're interested in helping debug problems with Kismet, here's the most useful way to do so:

1. Compile Kismet from source (per the quick start guide above)

2. Install Kismet (typically via `sudo make suidinstall`)

3. Run Kismet, *FROM THE SOURCE DIRECTORY*, in `gdb`:

  ```bash
  $ gdb ./kismet
  ```

  This loads a copy of Kismet with all the debugging info intact; the copy of Kismet which is installed system-wide usually has this info removed; the installed version is 1/10th the size, but also lacks a lot of useful information which we need for proper debugging.

4. Tell GDB to ignore the PIPE signal 

   ```
   (gdb) handle SIGPIPE nostop noprint pass
   ```

   This tells GDB not to intercept the SIGPIPE signal (which can be generated, among other times, when a data source has a problem)

5. Configure GDB to log to a file

  ```
  (gdb) set logging on
  ```

  This saves all the output to `gdb.txt`

5. Run Kismet - *in debug mode*

  ```
  (gdb) run --debug [any other options]
  ```

  This turns off the internal error handlers in Kismet; they'd block gdb from seeing what happened.  You can specify any other command line options after --debug; for instance:

  ````
  (gdb) run --debug -n -c wlan1
  ````

6.  Wait for Kismet to crash

7.  Collect a backtrace
   ```
   (gdb) bt
   ```

   This shows where Kismet crashed.

8.  Collect thread info
   ```
   (gdb) info threads
   ```

   This shows what other threads were doing, which is often critical for debugging.

9.  Collect per-thread backtraces
   ```
   (gdb) thread apply all bt full
   ```

   This generates a dump of all the thread states

10. Send us the gdb log and any info you have about when the crash occurred; dragorn@kismetwireless.net or swing by IRC or the Discord channel (info available about these on the website, https://www.kismetwireless.net)

#### Advanced debugging

If you're familiar with C++ development and want to help debug even further, Kismet can be compiled using the ASAN memory analyzer; to rebuild it with the analyser options:

```
    $ make clean
    $ CC=clang CXX=clang++ ./configure --enable-asan
```

ASAN has a performance impact and uses significantly more RAM, but if you are able to recreate a memory error inside an ASAN instrumented Kismet, that will be very helpful.

## Upgrading & Using Kismet Git-Master (or beta)

The safest route is to remove any old Kismet version you have installed - by uninstalling the package if you installed it via your distribution, or by removing it manually if you installed it from source (specifically, be sure to remove the binaries `kismet_server`, `kismet_client`,  and `kismet_capture`, by default found in `/usr/local/bin/` and the config file `kismet.conf`, by default in `/usr/local/etc/`.

You can then configure, and install, the new Kismet per the quickstart guide above.

While heavy development is underway, the config file may change; generally breaking changes will be mentioned on Twitter and in the git commit logs.

Sometimes the changes cause problems with Git - such as when temporary files are replaced with permanent files, or when the Makefile removes files that are now needed.  If there are problems compiling, the easiest first step is to remove the checkout of directory and clone a new copy (simply do a `rm -rf` of the copy you checked out, and `git clone` a new copy)

## Installing Kismet - Suid vs Normal

It is **strongly** recommended that Kismet never be run as root; instead use the Kismet suid-root installation method; when compiling from source it can be installed via:
```
$ ./configure
$ make
$ sudo make suidinstall
```

Nearly all packages of Kismet *should* support the suid-root install method as well.

This will create a new group, 'kismet', and install capture tools which need root access as suid-root but only runnable by users in the 'kismet' group.

This will allow anyone in the Kismet group to change the configuration of wireless interfaces on the system, but will prevent Kismet from running as root.

#### Why does Kismet need root?

Controlling network interfaces on most systems requires root, or super-user access.

While written with security strongly in mind, Kismet is a large and complex program, which handles possibly hostile data from the world.  This makes it a very bad choice to run as root.

To mitigate this, Kismet uses separate processes to control the network interfaces and capture packets.  These capture programs are much smaller than Kismet itself, and do minimal (or no) processing on the contents of the packets they receive.

## Configuring Kismet

Kismet is primarily configured through a set of configuration text files.  By default these are installed into `/usr/local/etc/'`. The config files are broken into several smaller files for readability:

* `kismet.conf`
  The master config file which loads all other configuration files, and contains most of the system-wide options

* `kismet_alerts.conf`
  Alert / WIDS configuration, which includes rules for alert matching, rate limits on alerts, and other IDS/problem detection options

* `kismet_httpd.conf`
  Webserver configuration

* `kismet_memory.conf`
  Memory consumption and system tuning options.  Typically unneeded, but if you have a massive number of devices or an extremely resource-limited system, how Kismet uses memory can be tuned here.

* `kismet_storage.conf`
  Kismet persistent storage configuration

* `kismet_logging.conf`
  Log file configuration

* `kismet_uav.conf`
   Parsing rules for detecting UAV / Drones or similar devices; compiled from the `kismet_uav.yaml` file

* `kismet_site.conf`
   Optional configuration override; Kismet will load any options in the `kismet_site.conf` file last and they will take precedence over all other configs.

#### Configuration Format

Configuration files are plain text.  Lines beginning with a '#' are comments, and are ignored.
    
Configuration options all take the form of:
   `option=value`

Some configuration options support repeated definitions, such as the 
`source` option which defines a Kismet datasource:
   `source=wlan0`
   `source=wlan1`
    
Kismet supports importing config files.  This is used by Kismet itself to
split the config files into more readable versions, but can also be used
for including custom options.

* `include=/path/to/file`
   Include a config file; this file is parsed immediately, and the file **must** exist or Kismet will exit with an error.
* `opt_include=/path/to/file`
   Include an **optional** config file.  If this file does not exist, Kismet will generate a warning, but continue working.
* `opt_override=/path/to/file`
   Include an **optional OVERRIDE** config file.  This is a special file which is loaded at the **end** of all other configuration.  Any configuration options found in an override file **replace all other instances of those configurations**.  This is a very powerful mechanism for provisioning multiple Kismet servers or making a config which survives an upgrade and update to the newest configuration files when running from git.
   
#### Configuration Override Files - `kismet_site.conf`

Most users installing Kismet will likely edit the configuration files
directly.  This file is not needed by most users, and can be ignored, however if you are configuring Kismet on multiple systems, this may be useful.
    
When Kismet frequently from source (for instance, testing Git) or preparing Kismet server deployments across multiple systems presents other challenges.
    
By default, Kismet will look for an optional override file in the default
configuration directory (/usr/local/etc by default) named `kismet_site.conf`.
    
This file is specified as an OVERRIDE FILE.  Any options placed in kismet_site.conf will REPLACE ANY OPTIONS OF THE SAME NAME.
    
This mechanism allows a site configuration to override any default config
options, while not making changes to any configuration file installed by
Kismet.  This allows new installations of Kismet to replace the config files with impunity while preserving a custom configuration.
    
Typical uses of this file might include changing the http data directory,
defining sources and memory options, forcing or disabling logging, and so on; a `kismet_site.conf` file might look like:
```
server_name=Some server
server_description=Building 2 floor 3

gps=serial:device=/dev/ttyACM0,name=laptop

remote_capture_listen=0.0.0.0
remote_capture_port=3501

source=wlan1
source=wlan2
```

## Starting Kismet

Kismet can be started normally from the command line, and will run in a small ncurses-based wrapper which will show the most recent server output, and a redirect to the web-based interface.
    
Kismet can also be started as a service; typically in this usage you should also pass `--no-ncurses` to prevent the ncurses wrapper from loading.
    
An example systemd script is in the `packaging/systemd/` directory of the Kismet source.

## Kismet Data Sources

Kismet gets data (which can be packets, devices, or other information) from "data sources".
    
Data sources can be created several ways:
* `source=foo` in kismet.conf
* `-c foo` on the command line when starting Kismet
* via the web interface
* scriptable via the REST api
  
Source definitions look like:
```
source=[interface]:[option, option, option]
```

    For example to capture from a Linux Wi-Fi device on 'wlan1' with no special
    options:
    
        source=wlan1
    
    To capture from a Linux Wi-Fi device on wlan1 while setting some special
    options, like telling it to not change channels and to go to channel 6
    to start with:
    
        source=wlan1:channel_hop=false,channel=6
        source=wlan1:channel_hop=false,channel=11HT-
    
    Different data sources have different options, read on for more information
    about the different capture sources Kismet supports.
    
    When no options are provided for a data source, the defaults are controlled 
    by settings in kismet.conf:
    
    channel_hop=true | false
    
        Controls if new sources enable channel hopping.  Because radios can only
        look at one channel at a time (typically), channel hopping jumps around
        the known channels.
    
        Typically, channel hopping should be turned on.  It can be turned off on
        individual data sources.
    
    channel_hop_speed=channels/sec | channels/min
    
        Channel hopping can happen either X times a second, or X times a minute.
        Slower channel hopping may capture more information on a busy channel, but
        will miss brief bursts of traffic on other channels; faster channel hopping
        may see more momentary traffic but will fail to capture complete records.
    
        By default, Kismet hops at 5 channels a second.
    
        Examples:
            channel_hop_speed=5/sec
            channel_hop_speed=10/min
    
    split_source_hopping=true | false
    
        Kismet can run with multiple interfaces for the same protocol - for instance,
        two, three, or even more Wi-Fi cards.  Typically it does not make sense to
        have multiple sources of the same type hopping to the same channel at the
        same time.  With split-hopping, Kismet will take the channel list for devices
        of the same type, and start each source at a different part of the channel
        list, maximizing coverage.
    
        Generally there is no reason to turn this off.
    
    randomized_hopping=true | false
    
        Generally, data sources retrieve the list of channels in sequential order.
        On some source types (like Wi-Fi), channels can overlap; hopping in a 
        semi-random order increases channel coverage by using overlap to spy on
        nearby channels when possible.
    
        Generally, there is no reason to turn this off.
    
    retry_on_source_error=true | false
    
        If true, Kismet will try to re-open a source which is in an error state
        after five seconds.
    
    timestamp=true | false
    
        If true, Kismet will override the timestamp of the packet with the 
        local timestamp of the server; this is the default behavior for
        remote capture sources but can be turned off either on a per-source
        basis or by turning it off in kismet.conf



Naming and describing data sources

    Datasources allow for annotations; these have no role in how Kismet operates,
    but the information is stored alongside the source definition and is available
    in the Kismet logs and in the web interface.
    
    The following information can be set in a source, but is not required:
    
    name=arbtitrary name
    
        Gives the source a human-readable name.  This name shows up in the
        web UI and logs.  This can be extremely useful when running remote
        captures where multiple interfaces might be enumerated as 'wlan0'
        for instance.
    
        source=wlan0:name=foobar_main_card
    
    info_antenna_type=arbitrary string type
    
        Denotes the antenna type; this is a string which should have some
        sort of meaning to you, for example:
    
        source=wlan0:info_antenna_type=omni
    
    info_antenna_gain=value in dB
    
        Antenna gain in dB, for example:
    
        source=wlan0:info_antenna_type=omni,info_antenna_gain=5.5
    
    info_antenna_orientation=degrees
    
        Antenna orientation, if a fixed antenna, in degrees:
    
        source=wlan0:info_antenna_orientation=180
    
    info_antenna_beamwidth=width in degrees
    
        Antenna beamwidth
    
        source=wlan0:info_antenna_type=yagi,info_antenna_beamwidth=30
    
    info_amp_type=amplifier type
    
        Arbitrary string type of amplifier, if one is present:
    
        source=wlan0:info_amp_type=custom_duplex
    
    info_amp_gain=gain in dB
    
        Amplifier gain in dB:
    
        source=wlan0:info_amp_gain=20


    Kismet will attempt to open all the sources defined on the command line 
    (with the '-c' option), or if no sources are defined on the command line, 
    all the sources defined in the Kismet config files.
    
    If a source has no functional type and encounters an error on startup,
    it will be ignored - for instance if a source is defined as:
        source=wlx4494fcf30eb3
    
    and that device is not connected when Kismet is started, it will be ignored.
    
    However, if the type is known, for instance:
        source=wlx4494fcf30eb3:type=linuxwifi
    
    then Kismet will know how to re-start it in the future, and will detect when
    the device is plugged in.


​    
Datasource: Linux Wi-Fi

    Most likely this will be the main data source most people use when capturing
    with Kismet.
    
    The Linux Wi-Fi data source handles capturing from Wi-Fi interfaces using the
    two most recent Linux standards:  The new netlink/mac80211 standard present
    since approximately 2007, and the legacy ioctl-based IW extensions system
    present since approximately 2002.
    
    Packet capture on Wi-Fi is accomplished via "monitor mode", a special mode 
    where the card is told to report all packets seen, and to report them at
    the 802.11 link layer instead of emulating an Ethernet device.
    
    The Linux Wi-Fi source will auto-detect supported interfaces by querying the
    network interface list and checking for wireless configuration APIs.  It
    can be manually specified with 'type=linuxwifi':
    
        source=wlan1:type=linuxwifi
    
    The Linux Wi-Fi capture uses the 'kismet_cap_linux_wifi' tool, and should
    typically be installed suid-root:  Linux requires root to manipulate the
    network interfaces and create new ones.
    
    Example source definitions:
        source=wlan0
        source=wlan1:name=some_meaningful_name
    
    Supported Hardware
    
    Not all hardware and drivers support monitor mode, but the majority do.  
    Typically any driver shipped with the Linux kernel supports monitor mode,
    and does so in a standard way Kismet understands.  If a specific piece of 
    hardware does not have a Linux driver yet, or does not have a standard driver 
    with monitor mode support, Kismet will not be able to use it.
    
    The Linux Wi-Fi source is known to support, among others:
        - All Atheros-based cards (ath5k, ath9k, ath10k with some restrictions, 
          USB-based atheros cards like the AR9271) (* Some issues)
        - Modern Intel-based cards (all supported by the iwlwifi driver including
          the 3945, 4965, 7265, 8265 and similar) (* Some issues)
        - Realtek USB devices (rtl8180 and rtl8187, such as the Alfa AWUS036H)
        - Realtek USB 802.11AC (rtl8812au), with some restrictions
        - Realtek USB 802.11AC (rtl8814), with some restrictions
        - RALink rt2x00 based devices
        - ZyDAS cards
        - Almost all drivers shipped with the Linux kernel
    
    Devices known to have issues:
        - ath9k Atheros 802.11abgn cards are typically the most reliable, however
          they appear to return false packets with valid checksums on very small
          packets such as phy/control and powersave control packets.  This may
          lead Kismet to detect spurious devices not actually present.
        - ath10k Atheros 802.11AC cards have many problems, including floods of
          spurious packets in monitor mode.  These packets carry 'valid' checksum
          flags, making it impossible to programmatically filter them.  Expect
          large numbers of false devices.
        - iwlwifi Intel cards appear to have errors when tuning to HT40 and VHT
          channels, leading to microcode/firmware crashes and resets of the card.
          Kismet works around this by disabling HT and VHT channels and only
          tuning to stock channels.
        - rtl8812au Realtek 802.11AC cards, such as the Alfa 802.11AC USB cards,
          have no in-kernel drivers.  There are many variants of the out-of-kernel
          driver, however most do NOT support monitor mode.
          A variant of the one driver currently known to work in monitor mode,
          with patches to compile on modern kernels, is available at:
          https://github.com/aircrack-ng/rtl8812au.git
    
    It will NOT work with:
        - Raspberry Pi 3 or ZeroW built-in Wi-Fi using standard drivers.  The Broadcom
          embedded firmware does not support monitor mode.  It may be possible
          to get it working with the Nexmon driver project, available at: 
              https://github.com/seemoo-lab/nexmon
          The Kali distribution for the Raspberry Pi3 and Raspberry Pi 0W includes
          the nexmon patches and should work fine.
        - Most out-of-kernel drivers installed by a distribution outside of
          the normal kernel driver set.  Some distributions (raspbian for instance)
          package custom drivers for many of the cheaper USB Wi-Fi adapters, and
          these drivers do not support monitor mode.
    
    Many more devices should be supported - if yours isn't listed and works, let
    us know via Twitter (@kismetwireless).  


​    
    Wi-Fi Channels
    
    Wi-Fi channels in Kismet define both the basic channel number, and extra channel
    attributes such as 802.11N 40MHz channels, 802.11AC 80MHz and 160MHz channels,
    and non-standard half and quarter rate channels at 10MHz and 5MHz.
    
    Kismet will auto-detect the supported channels on most Wi-Fi cards.  Monitoring
    on HT40, VHT80, and VHT160 requires support from your card.
    
    Channels can be defined by number or by frequency.
    
        XX          20MHz channel, such as '6' or '153'
        XXXX        20MHz frequency, such as '2412'
        XXHT40+     40MHz, 802.11N/AC, upper secondary, such as '6HT40+' or '2412HT40+'
        XXHT40-     40MHz, 802.11N/AC, lower secondary, such as '53HT40-'
        XXVHT80     80MHz, 802.11AC, 80MHz wide AC channel, such as '116VHT80'
        XXVHT160    160MHz, 802.11AC, 160MHz wide AC channel, '36VHT160'
        XXW10       10MHz half-channel, supported on some Atheros cards and few others.
                    Not auto-detected and must be manually added.
        XXW5        5MHz quarter-channel, supported on some Atheros cards and few others.
                    Not auto-detected and must be manually added.
    
    Wi-Fi Source Parameters
    
    Linux Wi-Fi sources accept several options in the source definition:
    
    add_channels="channel list"
    
        A comma-separated list of channels to be added to the detected channels on
        this source.  Kismet will auto-detect channels, then append any channels
        from this list.
    
        This list MUST BE IN QUOTES, for example:
    
            source=wlan0:name=foo,add_channels="1W5,6W5",hop_rate=1/sec
    
        If you are providing this list via the command line '-c' option,
        you will need to escape the quotes to keep your shell from hiding them,
        for example:
    
            $ kismet -c wlan0:name=foo,add_channels=\"1W5,6W5\",hop_rate=1/sec
    
        This option is most useful for including non-standard channels supported
        by some interfaces, such as 5 and 10MHz wide channels on Atheros cards.
    
    channel=channel-def
    
        When channel hopping is disabled, set a specific channel.
    
    channels="channel list"
    
        A comma-separated list of channels to be used on this source, instead of
        the default channels automatically detected. 
    
        This list MUST BE IN QUOTES, for example:
    
            source=wlan0:name=foo,channels="1,2,6HT40+,53,57",hop_rate=1/sec
    
        If you are providing this source via the command line '-c' option, you
        will need to escape the quotes to keep your shell from hiding them,
        for example:
    
            $ kismet -c wlan0:name=foo,channels=\"1,2,6HT40+,53\",hop_rate=1/sec
    
        You may specify channels which are not detected by the source probe, however
        if the source cannot tune to them, there will be errors.
    
    channel_hop=true | false
    
        Enable channel hopping on this source.  If this is omitted, the source
        will use the global hopping option.
    
    channel_hoprate=channels/sec | channels/min
    
        Like the global channel_hop_rate configuration option, this sets the 
        speed of channel hopping on this source only.  If this is omitted,
        the source will use the global hop rate.
    
    fcsfail=true | false
    
        Mac80211-based drivers sometimes have the option to report packets 
        which do not pass the frame checksum, or FCS.  Generally these packets
        are garbage - they are packets which, due to in-air corruption due to
        collisions with other packets, have become corrupt.
    
        Usually there is no good reason to turn this on unless you are doing
        research on non-standard packets and hope to glean some sort of
        information from them.
    
    ht_channels=true | false
    
        Kismet will tune to HT40 channels when available; to disable this 
        behavior, set ht_channels to false and Kismet will no longer index
        HT40+ and HT40- variants in the autodetected channel list.
    
        This option may also be used to override Kismet disabling HT channels
        on some drivers where there have been known problems, such as 
        Intel iwlwifi; to FORCE Kismet to use HT40 channels, set ht_channels=true.
        WARNING: This causes firmware resets on all currently known Intel cards!
    
        See the vht_channels option for similar control over 80 and 160MHz channels.
    
    ignoreprimary=true | false
    
        mac80211-based drivers use multiple virtual interfaces to control
        behavior:  A single Wi-Fi interface might have 'wlan0' as the 
        "normal" Wi-Fi interface, and Kismet would make 'wlan0mon' as the
        capture interface.
    
        Typically, all non-monitor interfaces must be placed into a "down"
        state for capture to work reliably, or for channel hopping to work.
    
        In the rare case where you are attempting to run Kismet on the same
        interface as another mode (such as access point or client), you 
        may want to leave the base interface running.  If you set 
        "ignoreprimary=true" on the source, Kismet will not bring down the
        primary interface.
    
        This *almost always* must be combined with "hop=false" or setting
        channels will fail.
    
    plcpfail=true | false
    
        mac80211-based drivers sometimes have the ability to report events
        that may have looked like packets, but which have invalid low-level
        packet headers (PLCP).  Generally these events have no meaning, and
        very few drivers are able to report them.
    
        Usually there is no good reason to turn this on, unless you are 
        doing research attempting to capture Wi-Fi-like encoded data which
        is not actually Wi-Fi.
    
    uuid=AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE
    
        Assign a custom UUID to this source.  If no custom UUID is provided,
        a UUID is computed from the MAC address and the name of the datasource
        capture engine; the auto-generated UUID will be consistent as long as
        the MAC address of the capture interface remains the same.
    
        If you are assigning custom UUIDs, you *must ensure* that every UUID
        is *unique*.  Each data source must have its own unique identifier.
    
    vif=foo
    
        mac80211-based drivers use multiple virtual interfaces to control 
        behavior.  Kismet will make a monitor mode virtual interface (vif)
        automatically, named after some simple rules:
            - If the interface given to Kismet on the source definition is
              already in monitor mode, Kismet will use that interface and
              not create a VIF
            - If the interface name is too long, such as when some 
              distributions use the entire MAC address as the interface name,
              Kismet will make a new interface named 'kismonX'
            - Otherwise, Kismet will add 'mon' to the interface; ie given an
              interface 'wlan0', Kismet will create 'wlan0mon'
    
        The 'vif=' option allows setting a custom name which will be used
        instead of creating a name.
    
    vht_channels=true | false
    
        Kismet will tune to VHT80 and VHT160 channels when available; to disable 
        this behavior, set vht_channels to false and Kismet will no longer index
        VHT80 and VHT160 variants in the autodetected channel list.
    
        This option may also be used to override Kismet disabling VHT channels
        on some drivers where there have been known problems, such as 
        Intel iwlwifi; to FORCE Kismet to use VHT channels, set vht_channels=true.
        WARNING: This causes firmware resets on all currently known Intel cards!
    
        See the ht_channels option for similar control over HT40 channels.
    
    retry=true | false
        
        Automatically try to re-open this interface if an error occurs.  If the
        capture source encounters a fatal error, Kismet will try to re-open it in
        five seconds.  If this is omitted, the source will use the global retry
        option.


    Special Drivers
    
    Some drivers require special behavior - whenever possible, Kismet will detect
    these drivers and "do the right thing".
    
    - The rtl8812au driver (available at https://github.com/astsam/rtl8812au)
      supports monitor mode on these interfaces, however it appears to be 
      very timing sensitive.  Additionally, it supports creating mac80211 VIFs,
      but does NOT support capturing using them!  It will only support capturing
      from the base interface, which must be placed in monitor mode using
      the legacy ioctls.
    
      Additionally, the rtl8812au will sometimes refuse to tune to channels it
      reports as supported - other times it works as expected.  Kismet will continue
      despite intermittent errors.



Datasource: Linux Bluetooth

    Bluetooth uses a frequency-hopping system with dynamic MAC addresses and other
    oddities - this makes sniffing it not as straightforward as capturing Wi-Fi.
    
    The Linux Bluetooth source will auto-detect supported interfaces by querying the
    bluetooth interface list.  It can be manually specified with 'type=linuxbluetooth'.
    
    The Linux Bluetooth capture uses the 'kismet_cap_linux_bluetooth' tool, and should
    typically be installed suid-root:  Linux requires root to manipulate the
    'rfkill' state and the management socket of the Bluetooth interface.
    
    Example source
    
        source=hci0:name=linuxbt
    
    Supported Hardware
    
    For simply identifying Bluetooth (and BTLE) devices, the Linux Bluetooth 
    datasource can use any standard Bluetooth interface supported by Linux.
    
    This includes almost any built-in Bluetooth interface, as well as external
    USB interfaces such as the Sena UD100.
    
    Service Scanning
    
    By default, the Kismet Linux Bluetooth data source turns on the Bluetooth
    interface and enables scanning mode.  This allows it to see broadcasting
    Bluetooth (and BTLE) devices and some basic information such as the device
    name, but does not allow it to index services on the device.
    
    Bluetooth Source Parameters
    
    uuid=AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE
    
        Assign a custom UUID to this source.  If no custom UUID is provided,
        a purely random UUID is generated.



Data source: OSX Wifi

    Kismet can use the built-in Wi-Fi on a Mac, but ONLY the built-in Wi-Fi;
    Unfortunately currently there appear to be no drivers for OSX for USB
    devices which support monitor mode.
    
    Kismet uses the kismet_cap_osx_corewlan_wifi tool for capturing on OSX.
    
    OSX Wifi Parameters
    
    channel=channel-def
    
        When channel hopping is disabled, set a specific channel.
    
    channels="channel list"
    
        A comma-separated list of channels to be used on this source, instead of
        the default channels automatically detected. 
    
        This list MUST BE IN QUOTES, for example:
    
            source=en1:name=foo,channels="1,2,6HT40+,53,57",hop_rate=1/sec
    
        If you are providing this source via the command line '-c' option, you
        will need to escape the quotes to keep your shell from hiding them,
        for example:
    
            $ kismet -c en1:name=foo,channels=\"1,2,6HT40+,53\",hop_rate=1/sec
    
        You may specify channels which are not detected by the source probe, however
        if the source cannot tune to them, there will be errors.
    
    channel_hop=true | false
    
        Enable channel hopping on this source.  If this is omitted, the source
        will use the global hopping option.
    
    channel_hoprate=channels/sec | channels/min
    
        Like the global channel_hop_rate configuration option, this sets the 
        speed of channel hopping on this source only.  If this is omitted,
        the source will use the global hop rate.



Data source: Pcapfile

    Pcap files are a standard format generated by libpcap, most commonly in
    conjunction with a tool like tcpdump, wireshark, or Kismet itself.
    
    Kismet can replay a pcapfile for testing, debugging, demo, or re-processing.
    
    The Pcapfile datasource will auto-detect pcap files and paths to files:
        $ kismet -c /tmp/foo.pcap
    
    It can be manually specified with 'type=pcapfile'
    
    The pcapfile capture uses the 'kismet_cap_pcapfile' tool which does not need
    special privileges.


    Pcapfile Options
    
    pps=rate
    
        Normally pcapfiles are replayed as quickly as possible, on very large pcaps
        this can lead to processing and memory problems.  Specifying a packet-per-second
        rate throttles the pcap replay rate.  
    
        This option cannot be combined with the realtime option.
    
    realtime=true | false
    
        Normally pcapfiles are replayed as quickly as possible.  Specifying the
        realtime=true option will slow the pcap file playback to match the original
        capture rate.
    
    retry=true | false
        
        Automatically try to re-open this interface if an error occurs.  If the
        capture source encounters a fatal error, Kismet will try to re-open it in
        five seconds.  If this is omitted, the source will use the global retry
        option.
    
        Pcap files will (obviously) contain the same content each time, so replaying
        typically will not cause devices to update.
    
    uuid=AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE
    
        Assign a custom UUID to this source.  If no custom UUID is provided,
        a purely random UUID is generated.



Data source: SDR RTL433

    The rtl-sdr radio is an extremely cheap USB SDR (software defined radio).  While
    very limited, it is still capable of performing some useful monitoring.
    
    Kismet is able to take data from the rtl_433 tool, which can read the broadcasts
    of an multitude of wireless thermometer, weather, electrical, tire pressure,
    and other sensors.
    
    More information about the rtl-sdr is available at:
        https://www.rtl-sdr.com
    
    The rtl_433 tool can be downloaded from:
        https://github.com/merbanan/rtl_433
    or as a package in your distribution.
    
    The Kismet rtl_433 interface uses librtlsdr, rtl_433, and Python; rtl433 sources
    will show up as normal Kismet sources using the rtl433-X naming.
    
    For more information about the rtl433 support, see the README in the 
    capture_sdr_rtl433 directory.



Remote Packet Capture

    Kismet can capture from a remote source over a TCP connection.
    
    Kismet remote packet feeds are initiated by the same tools that Kismet uses to
    configure a local source; for example if Kismet is running on a host on IP 
    192.168.1.2, to capture from a Linux Wi-Fi device on another device you could
    use:
    
        $ /usr/local/bin/kismet_cap_linux_wifi \
            --connect 192.168.1.2:3501 --source=wlan1
    
    Specifically, this uses the kismet_cap_linux_wifi tool, which is by default 
    installed in `/usr/local/bin/`, to connect to the IP 192.168.1.2 port 3501.
    
    The --source=... parameter is the same as you would use in a `source=' Kismet
    configuration file entry, or as `-c' to Kismet itself. 
    
    You can set names for remote sources the same way you set the name for a 
    local interface:
    
        $ /usr/local/bin/kismet_cap_linux_wifi \
            --connect 192.168.1.2:3501 --source=wlan1:name=sensor_foo_wlan1


    By default, Kismet only allows remote packet connections from the localhost IP; 
    you must either:
    
    1.  Set up a tunnel, for example using SSH port forwarding, to connect the remote 
        device to the host Kismet is running on.  This is very simple to do, and adds
        security to the remote packet connection:
    
        $ ssh someuser@192.168.1.2 -L 3501:localhost:3501
    
        Then in another terminal:
    
        $ /usr/local/bin/kismet_cap_linux_wifi \
            --connect localhost:3501 --source=wlan1
    
        The `ssh' command places SSH in the background (using `-f'), connects to 
        the host Kismet is running on, and tunnels port 3501.
    
        The kismet_cap_linux_wifi command is the same as the first example, but
        connects to localhost:3501 to use the SSH port forwarding.
    
        Other, more elegant solutions exist for building the SSH tunnel, such 
        as `autossh'.
    
    2.  Kismet can be configured to accept connections on a specific interface,
        or from all IP addresses, by changing the `remote_capture_listen=' line in
        kismet.conf:
    
        remote_capture_listen=0.0.0.0
    
        would enable listening on all interfaces, while
    
        remote_capture_listen=192.168.1.2
    
        would enable listening only on the given IP (again using the above example 
        of Kismet running on 192.168.1.2).
    
        Remote capture *should only be enabled on interfaces on a protected LAN*.
    
    Additional remote capture arguments
    
    Kismet capture tools supporting remote capture also support the following options:
    
    --connect=[host]:[port]
    
        Connects to a remote Kismet server on [host] and port [port].  When using
        `--connect=...' you MUST specify a `--source=...' options
    
    --source=[source definition]
    
        Define a source; this is used only in remote connection mode.  The source
        definition is the same as defining a local source for Kismet via `-c' or
        the `source=' config file option.
    
    --disable-retry
    
        By default, a remote source will attempt to reconnect if the connection
        to the Kismet server is lost.
    
    --daemonize
    
        Places the capture tool in the background and daemonizes it.
    
    --fixed-gps [lat,lon,alt] or [lat,lon]
    
        Set the GPS location of the remote capture source; this will tag any 
        packets from this source with a static, fixed GPS location.
    
    --gps-name [name]
    
        Rename the virtual GPS reported on this source; otherwise the capture
        name "fixed-remote" is used.



Compiling Only the Capture Tools

    Typically, you will want to compile all of Kismet.  If you're doing specific
    remote-capture installs, however, you may wish to compile only the 
    binaries used by Kismet to enable capture mode and pass packets to a 
    full Kismet server.
    
    To compile ONLY the capture tools:
    
    Tell configure to only configure the capture tools; this will allow you
    to configure without installing the server dependencies such as C++,
    microhttpd, protobuf-C++, etc.  You will still need to install dependencies
    such as protobuf-c, build-essentials, and similar.
    $ ./configure --enable-capture-tools-only
    
    Once configure has completed, compile the capture tools:
    $ make datasources
    
    You can now copy the compiled datasources to your target.



Tuning Kismet Packet Capture

    Kismet has a number of tuning options to handle quirks in different types 
    packet captures.  These options can be set in the kismet.conf config file
    to control how Kismet behaves in some situations:
    
    dot11_process_phy=[true|false]
    
        802.11 Wi-Fi networks have three basic packet classes - Management,
        Phy, and Data.  The Phy packet type is the shortest, and caries the
        least amount of information - it is used to acknowledge packet reception
        and controls the packet collision detection CTS/RTS system.  These packets
        can be useful, however they are also the most likely to become corrupted
        and still pass checksum.
    
        Kismet turns off processing of Phy packets by default because they can lead
        to spurious device detection, especially in high-data captures.  For 
        complete tracking and possible detection of hidden-node devices, it can
        be set to 'true'.



Kismet Webserver

    Kismet now integrates a webserver which serves the web-based UI and data
    to external clients.
    
    THE FIRST TIME YOU RUN KISMET, it will generate a RANDOM password.  This
    password is stored in:
    ~/.kismet/kismet_httpd.conf
    
    which is in the home directory of the user which ran Kismet.
    
    You will need this password to log into Kismet for the first time.
    
    The webserver is configured via the kismet_httpd.conf file.  These options
    may be included in the base kismet.conf file, but are broken out for
    clarity.
    
    By default, Kismet does not run in SSL mode.  If you provide a certificate
    and key file in PEM format, Kismet supports standard SSL / HTTPS.  For more
    information on creating a SSL certificate, look at:
        README.SSL
    
    HTTP configuration options:
    
    httpd_username=username
    
        Set the username.  This is required for any actions which can change
        configuration (adding / removing data sources, changing server-side
        configuration data, downloading packet captures, etc).
    
        The default user is 'kismet', and by default, the httpd_username= and
        httpd_password= configuration options are stored in the 
        users home directory, in ~/.kismet/kismet_httpd.conf.
    
    httpd_password=password
    
        Set the password.  The first time you run Kismet, it will auto-generate
        a random password and store it in ~/.kismet/kismet_httpd.conf .
    
        It is generally preferred to keep the username and password in the
        per-user configuration file, however they may also be set here in 
        the global config.
    
        If httpd_username or httpd_password is found in the global config, it is
        used instead of the per-user config value.
    
    httpd_port=port
    
        Sets the port for the webserver to listen to.  By default, this is
        port 2501, the port traditionally used by the Kismet client/server
        protocol.
    
    httpd_ssl=true|false
    
        Turn on SSL.  If this is turned on, you must provide a SSL certificate
        and key in PEM format with the httpd_ssl_cert and httpd_ssl_key 
        configuration options.
    
        See README.SSL for more information about SSL certificates.
    
    httpd_ssl_cert=/path/to/cert.pem
    
        Path to a PEM-format SSL certificate.  
        
        This option is ignored if Kismet is not running in SSL mode.
    
        Logformat escapes can be used in this.  Specifically, "%S" 
        will automatically expand to the system install data directory,
        and "%h" will expand to the home directory of the user running
        Kismet.
    
        Example:
            httpd_ssl_cert=%h/.kismet/kismet.pem
    
    httpd_ssl_key=/path/to/key.pem
    
        Path to a PEM-format SSL key file.  This file should not have a
        password set.  
        
        This option is ignored if Kismet is not running in SSL mode.
    
        Logformat escapes can be used in this.  Specifically, "%S" 
        will automatically expand to the system install data directory,
        and "%h" will expand to the home directory of the user running
        Kismet.
    
        Example:
            httpd_ssl_key=%h/.kismet/kismet.key
       
    httpd_home=/path/to/httpd/data
    
        Path to static content web data to be served by Kismet.  This is
        typically set automatically to the directory installed by Kismet 
        in the installation prefix.
    
        Logformat escapes can be used in this.  Specifically, "%S" will 
        automatically expand to the system install data directory.  By
        default this should be:
            httpd_home=%S/kismet/httpd/
    
        Typically the only reason to change this directory is to replace
        the Kismet web UI with alternate code.
    
    httpd_user_home=/path/to/user/httpd/data
    
        Path to static content stored in the home directory of the 
        user running Kismet.  This is typically set to the httpd directory
        inside the users .kismet directory.
    
        This allows plugins installed to the user directory to install
        web UI components.
    
        Logformat escapes can be used in this.  Specifically, "%h" will
        expand to the current users home directory.  By default this should
        be:
            httpd_user_home=%h/.kismet/httpd/
    
        Typically there is no reason to change this directory.
    
        If you wish to disable serving content from the user directory 
        entirely, comment this configuration option out.
    
    httpd_session_db=/path/to/session/db
    
        Path to save HTTP sessions to.  This allows Kismet to remember valid
        browser login sessions over restarts of kismet_server. 
    
        If you want to refresh the logins (and require browsers to log in 
        again after each restart), comment this option.
    
        Typically there is no reason to change this option.
    
        Logformat escapes can be used in this.  Specifically, "%h" will 
        expand to the current users home directory.  By default this
        should be:
            httpd_session_db=%h/.kismet/session.db
    
    httpd_mime=extension:mimetype
    
        Kismet supports MIME types for most standard file formats, however if
        you are serving custom content with a MIME type not correctly set,
        additional MIME types can be defined here.
    
        Multiple httpd_mime lines may be used to add multiple mime types.
    
        Example:
            httpd_mime=html:text/html
            httpd_mime=svg:image/svg+xml
    
        Typically, MIME types do not need to be added.



GPS

    Kismet can integrate with a GPS device to provide geolocation coordinates
    for devices.
    
    GPS data is included in the log files, in PPI pcap files, and exported
    over the REST interface.
    
    Kismet can not use GPS to determine the absolute location of the device;
    it can only use it to determine the location of the receiver.  The 
    location estimate of a device can be improved by circling the suspected
    location.
    
    In addition to logging GPS data on a per-packet basis, Kismet maintains a
    running average of device locations which are exported as the average
    location in the Kismet UI and in device summaries.  Because the running
    average can be heavily influenced by the sensors position, this running
    average may not be very accurate.


    Multiple GPS devices can be defined at once, however only the highest 
    priority active device is used.
    
    GPS is configured via the 'gps=' configuration option.  GPS options are
    passed on the configuration line:
        gps=type:option1=val1,option2=val2
    
    Supported GPS types are:
    
    serial (High priority)
        Locally-connected serial NMEA GPS device.  This supports most
        USB and Bluetooth (rfcomm/spp) connected GPS devices.  This does
        not support the few GPS devices which output proprietary binary
    
        Options:
    
        name=foo
            Arbitrary name to identify this GPS device.
    
        device=/path/to/device
            Path to the serial device.  The user Kismet is running as must
            have access to this device.
    
        reconnect=true|false
            Automatically re-open the serial port if there is a problem with
            the GPS or if it is disconnected.
    
        baud=rate
            Specify a baud rate for the serial port.  Most serial GPS devices
            operate at 4800, which Kismet uses by default.  If your device
            is special, set the baud rate here.
    
        Example:
    
        gps=serial:device=/dev/ttyACM0,reconnect=true,name=LaptopSerial
    
    tcp (High priority)
        Network-connected raw NMEA stream.  Typically this is served by a
        smartphone app like "BlueNMEA" on Android or "NMEA GPS" on iPhone.
        For GPSD-based network GPS connections, use the "gpsd" GPS in
        Kismet.
    
        Options:
    
        name=foo
            Arbitrary name to identify this GPS device.
    
        host=ip-or-name
            IP address or hostname of NMEA network server
    
        port=port
            Port the NMEA network server is listening on
    
        reconnect=true|false
            Automatically reconnect to the NMEA TCP server if there is a 
            network error.
    
    gpsd (High priority)
        A GPSD server.  GPSD (http://www.catb.org/gpsd/) parses GPS
        data from multiple GPS vendors (including proprietary binary)
        and makes it available over a standard TCP/IP connection.
    
        There are multiple GPSD versions with various levels of support
        and incompatible protocols.  Kismet supports the older-style GPSD
        text protocol as well as the new GPSD3 JSON protocol.
    
        Options:
    
        name=foo
            Arbitrary name to identify this GPS device.
    
        host=hostname-or-ip
            Hostname or IP of GPSD host.
    
        port=port
            GPSD port.  GPSD listens on port 2947 by default.
    
        reconnect=true|false
            Automatically reconnect to the GPSD server if the connection
            is lost.
    
        Example:
    
        gps=gpsd:host=localhost,port=2947,reconnect=true
    
    web (Medium priority)
        A web-based client with a modern web browser and location hardware 
        (such as a phone) can supply their GPS location.  This is only 
        available to logged-in users on the Kismet web UI, but can turn a
        generic phone and web browser into a location source.
    
        Typically browsers cannot supply speed or other options, and the
        precision of this GPS source will be reduced because it may not
        be updated as frequently as a locally connected GPS.
    
        Options:
    
        name=foo
            Arbitrary name to identify this GPS device.
    
        Example:
    
        gps=web:name=web
        -or-
        gps=web
    
    virtual (lowest priority)
        A virtual GPS always reports a static location.  The virtual gps
        injects location information on stationary sensor or drone.
    
        Options:
    
        name=foo
            Arbitrary name to identify this GPS device.
    
        lat=coordinate
            Latitude coordinate.
    
        lon=coordinate
            Longitude coordinate.
    
        alt=altitude
            Altitude, in meters.
    
        Example:
    
        gps=virtual:lat=123.4566,lon=40.002,alt=23.45



Kismet Memory and Processor Tuning

    Kismet has several options which control how much memory and processing it 
    uses.  These are found in `kismet_memory.conf`:
    
    tracker_device_timeout=seconds
    
        Kismet will forget devices which have been idle for more than the 
        specified time, in seconds.
    
        Kismet will also forget links between devices (such as access points and
        clients) when the device has been idle for more than the specified time.
    
        This is primarily useful on long-running fixed Kismet installs.
    
    tracker_max_devices=devices
    
        Kismet will start forgetting the oldest devices when more than the 
        specified number of devices are seen.
    
        There is no terribly efficient way to handle this, so typically, leaving
        this option unset is the right idea.  Memory use can be tuned over time
        using the `tracker_device_timeout` option.
    
    keep_location_cloud_history=true|false
    
        Kismet can track a 'cloud' style history of locations around a device;
        Similar to a RRD (round robin database), the precision of the records
        decreases over time.  
    
        The location cloud can be useful for plotting devices on a map, but
        also takes more memory per dvice.
    
    keep_datasource_signal_history=true|false
    
        Kismet can keep a record of the signal levels of each device, as seen
        by each data source.  This is used for tracking signal levels across
        many sensors, but uses more memory.
    
    alertbacklog=number
    
        The number of alerts Kismet saves for displaying to new clients; setting
        this too low can prevent clients from seeing alerts but saves memory.
    
        Alerts will still be logged.
    
    packet_dedup_size=packets
    
        When using multiple datasources, Kismet keeps a list of the checksums
        of previous packets; this prevents multiple copies of the same packet
        triggering alerts.
    
    packet_backlog_warning=packets
        
        Kismet will start raising warnings when the number of packets waiting
        to be processed is over this number; no action will be taken, but
        an alert will be generated.
    
        This can be set to zero to disable these warnings; Kismet defaults to
        zero.  Disabling these warnings will NOT disable the backlog limit
        warnings.
    
    packet_backlog_limit=packets
    
        This is a *hard limit*.  If the packet processing thread is not able
        to process packets fast enough, new packets will be dropped over this
        limit.
    
        This can be set to 0; Kismet will never drop packets.  This may 
        lead to a runaway memory situation, however.
    
    ulimit_mbytes=ram_in_megabytes
    
        Kismet can hard-limit the amount of memory it is allowed to use via the 
        'ulimit' system; this could be set via a launch/setup script using the
        at startup.  
    
        If Kismet runs out of ram, it *will exit immediately* as if the system 
        had encountered an out-of-memory error.
    
        This setting should ONLY be combined with a restart script that relaunches
        Kismet, and typically should only be used on long-running WIDS-style 
        installs of Kismet.
    
        If this value is set too low, Kismet may fail to start the webserver 
        correctly or perform other startup tasks.  This value should typically
        only be used to control unbounded growth on long-running installs.
    
        The memory value is specified in *megabytes of ram*



SIEM support

    Kismet is natively compatible with the Prelude SIEM event management system
    (https://www.prelude-siem.org) and can send Kismet alerts to Prelude.
    
    To enable communication with a Prelude SIEM sensor, support must be enabled 
    at compile time by adding --enable-prelude to any other options passed to
    the configure script:
        $ ./configure --enable-prelude



Integrated libraries

    Kismet uses several header-only libraries which are part of the Kismet source
    code:
    
    - fmtlib (https://github.com/fmtlib/fmt): C++ string formatting for faster
    message generation with fewer temporary variables.
    
    - kaitai (https://kaitai.io): Binary parser generator and stream library
    
    - jsoncpp (http://jsoncpp.sourceforge.net/): JSON parser
    
    - radiotap (http://radiotap.org): Radiotap header definitions
    
    - microhttpd (https://www.gnu.org/software/libmicrohttpd/): Webserver

