
# Linux Questions

## Kernel Modules

1. How do you verify which kernel modules are currently loaded in the system?

```
$ lsmod
Module                  Size  Used by
btrfs                1101824  0
xor                    24576  1 btrfs
raid6_pq              118784  1 btrfs
ufs                    73728  0
qnx4                   16384  0
```

2. If a kernel module does not load or fails to load, describe the process you would take to
troubleshoot the problem.

* Examine the output of `dmesg` to see if any kernel-related errors ocurred

* Try to reload the module with verbose options to see if any error jumps out.

```
$ modprobe -vvv ptp
modprobe: INFO: ../libkmod/libkmod.c:364 kmod_set_log_fn() custom logging function 0x560ab36354b0 registered
modprobe: DEBUG: ../libkmod/libkmod-index.c:755 index_mm_open() file=/lib/modules/4.13.0-38-generic/modules.dep.bin
modprobe: DEBUG: ../libkmod/libkmod-index.c:755 index_mm_open() file=/lib/modules/4.13.0-38-generic/modules.alias.bin
modprobe: DEBUG: ../libkmod/libkmod-index.c:755 index_mm_open() file=/lib/modules/4.13.0-38-generic/modules.symbols.bin
```

* Use gdb to possibly find the problem (See: http://linux-hacks.blogspot.com/2009/07/using-gdb-for-debugging-kernel-modules.html)

## Networking

1. How do you list the current IP addresses and routing table of the system?

* IP addresses:

```
$ ifconfig
eno1      Link encap:Ethernet  HWaddr 90:b1:1c:9e:df:95
          inet addr:10.226.2.205  Bcast:10.226.2.255  Mask:255.255.255.0
          inet6 addr: fe80::5643:187c:a286:9d4d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3177265 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1353836 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:597527053 (597.5 MB)  TX bytes:151246570 (151.2 MB)
          Interrupt:20 Memory:f7200000-f7220000

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:3114 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3114 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:334445 (334.4 KB)  TX bytes:334445 (334.4 KB
etc.
```

* routing table

```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.226.2.1      0.0.0.0         UG    100    0        0 eno1
10.226.2.0      *               255.255.255.0   U     100    0        0 eno1
ntp.chi.cashnet 10.226.2.1      255.255.255.255 UGH   100    0        0 eno1
link-local      *               255.255.0.0     U     1000   0        0 eno1
172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
```

2. Describe how DNS works.

* The local computer first checks any hosts file for static entries
* If this is not available, the local computer checks the local DNS cache.
* If it is not or no longer in the cache, it does a DNS query on ISP recursive DNS servers, generally checking their own DNS cache first.
* If the ISP does not have the DNS entry in its recursive server cache, it asks root nameservers (there are 13 of these throughout the world)
* If these do not have the answer, the query is sent to TLD (top-level domain) DNS servers (e.g. .com, .edu, .gov, etc.)
* If the TLD servers do not have this info, they refer to authoritative DNS servers (e.g. dyndns.com)
* Successive responses are stored in caches (above in reverse)
* All DNS records have a TTL (time to live) value.  This determines how old the DNS record can be before the server needs to request a new record.
* Once a record is retrieved by your computer, it reads the IP address from that record and continues with its networking request.


## Disk systems

1. Your system has a filesystem that is 100% full, you find the log file that is the cause and delete it. However, the system does not clear up the space. Describe the issue and how you would resolve it.

* I would suspect that the inodes are full on the machine:

```
$ df -i
Filesystem       Inodes   IUsed    IFree IUse% Mounted on
udev            2040831     575  2040256    1% /dev
tmpfs           2047412     815  2046597    1% /run
etc...
```

If this is the case, I would probably try to find directories with a lot of files in them and clean them up.
```
$ for i in /*; do echo $i; find $i |wc -l; done
/bin
160
/boot
357
etc.
```

## Other

1. How would you age out a linux user password to ensure he/she are force to change their password?

* I would use the following to expire the users passwd, where `username` is the user to force a password change on.

```
sudo passwd -e username
```
* As an alternative, you can also use the `chage` utility to fine tune expirations:
```
$ chage --help
Usage: chage [options] LOGIN

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximim number of days before password
                                change to MAX_DAYS
  -R, --root CHROOT_DIR         directory to chroot into
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
```

2. How do you make a file immutable so that an automation system like, puppet, ansible or chef cannot overwrite it?

* Use the `chattr` command:

```
$ lsattr pjs.txt
-------------e-- pjs.txt

$ sudo chattr +i pjs.txt
[sudo] password for psolomon:

$ lsattr  pjs.txt
----i--------e-- pjs.txt

$ rm pjs.txt
rm: cannot remove 'pjs.txt': Operation not permitted

$ sudo chattr -i pjs.txt

$ lsattr  pjs.txt
-------------e-- pjs.txt

$ rm pjs.txt
```

3. How do you make a linux shell script executable?

```
$ chmod +x script.sh
```

