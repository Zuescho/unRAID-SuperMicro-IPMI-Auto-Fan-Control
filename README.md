# unRAID SuperMicro IPMI Auto Fan Control
A pair of dirty little bash scripts to automate control of the fan speeds for a SuperMicro board using ipmitool. 

## What does it do?
- Gather all of the CPU package temps, determine max
- Gather all of the drive temps, determine max
- Set speeds of the fan zones based configurable temperature thresholds (5 speed settings per fan zone)


## Update 27.09.2023
Just updated my UnRaid server to 6.12.4 long story short, NerdPack was retired and gets uninstalled when you upgrade beyond 6.10.
This was a pain in the ass, so this means [ipmitool](https://github.com/ipmitool/ipmitool) will be missing.

**The offered Community Apps in UnRaid IPMI Tools (bySimonF) and IPMI-Tools(Forum-Layman) did not help me, IPMI Tools uses FreeIPMI under the hood  which is not the same as ipmitool and IPMI is overkill by coming with a Web-GUI etc.**

### How to FIX
- Create folder **extra** under ``/boot/``
- Download last known version ``wget https://github.com/Zuescho/unRAID-SuperMicro-IPMI-Auto-Fan-Control/raw/main/ipmitool-1.8.18-x86_64-1.txz``  
- Reboot (alternatively, you could ``installpkg ipmitool-1.8.18-x86_64-1.txz``)

You still need to use   
``modprobe ipmi_devintf``  
``modprobe ipmi_si``  
or else it won't find any sensors

### Why folder extra?
UnRaid installs packages that are in these folders a boot, so you will need it to be there.


### Why do i get an error when i try to install it, after downloading the .txz via wget.
```
installpkg ipmitool-1.8.18-x86_64-1.txz
Verifying package ipmitool-1.8.18-x86_64-1.txz.
xz: (stdin): File format not recognized
Unable to install ipmitool-1.8.18-x86_64-1.txz:  tar archive is corrupt (tar returned error code 2)
```
Well after hitting my head against the wall, if u use the ``file`` command you will notice that it is not an  ``ipmitool-1.8.18-x86_64-1.txz: XZ compressed data, checksum CRC64``  
but  ``ipmitool-1.8.18-x86_64-1.txz: JSON text data``  
so yeah lucky us...  
to be honest we are just dumb you need to use the raw location to get the file like this
``wget https://github.com/Zuescho/unRAID-SuperMicro-IPMI-Auto-Fan-Control/raw/main/ipmitool-1.8.18-x86_64-1.txz``  
with this you can, just download it directly into the ``/boot/extra folder``



## How to run:
This pair of scripts is specifically designed for unRAID's 'User Scripts' plugin. 
- Install the **User Scripts** plugin

- Add impi_fans_startup.sh to User Scripts
  - Set to run 'At Startup of Array'
    - _NOTE: If do not want to restart your server/array after adding this script, you must also click the "Run Script" button on ipmi_fans_startup.sh one time to force the required fan modes_
- Add ipmi_fans_auto.sh to User Scripts
  - Edit parameters within the script (read script comments for instructions) to suit your needs
  - Set to run 'Custom' and define a cron schedule in the field to run regularly (e.g. `*/5 * * * *` for every 5 minutes)

Example of how User Scripts should look once scheduled:
![image](https://user-images.githubusercontent.com/34625175/174487053-fbc9afcd-d289-44c6-ae86-fcc3e336601d.png)


### Other Notes:
- The server I used these scripts with had an Supermicro X11SSM-F motherboard.
- I've used/tested these up to unRAID v6.12.4.



###  ipmitool - can't find /dev/ipmi0 or /dev/ipmidev/0

You probably need to load the IPMI kernel modules:
````
modprobe ipmi_devintf
modprobe ipmi_si
````
Now there are many ways to call them after a reboot, UnRaid uses slackware and the ``/etc/modules`` file seems to be under  
``/etc/rc.d`` which you can open with nano ``/etc/rc.d/modules.local``  
just list the module names:  
````
ipmi_devintf
ipmi_si
````

Now option 2 is to put them in the **go** file under ``nano /boot/config/go``  
for example my go file right now
```
#!/bin/bash
# Start the Management Utility
/usr/local/sbin/emhttp &


modprobe ipmi_devintf
modprobe ipmi_si

```

Sadly i could not find any documentation on the go file or my googling is just becoming worse, well to keep it simple, after rebooting this file will be executed so if you put the commands here unRaid will execute them itself.

Now if there is a race condition between installing the ipmitool package and executed this command i do not know, but i hope not.

# Thanks to @dalkain for first creating this.