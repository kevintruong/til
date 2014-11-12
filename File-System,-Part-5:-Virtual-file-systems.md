## How can I copy bytes from one file to another?
Use `dd` command. For example, the following command copies 1mb of data from the file `/dev/urandom` to the file `/dev/null`. The data is copied as 1024 blocks of blocksize 1024 bytes.

dd if=/dev/urandom of=/dev/null bs=1k count=1024

Both the input and output files in the example above are virtual - they don't exist on a disk. Instead they are part of the `dev` filesystem, which is virtual filesystem provided by the kernel.
The virtual file `/dev/urandom` provides an infinite stream of random bytes, while the virtal file `/dev/null` ignores all bytes written to it. A common use of `/dev/null` is to discard the output of a command,
```
myverboseexecutable > /dev/null
```

Another commonly used /dev virtual file is `/dev/zero` which provides an infinite stream of zero bytes.
For example, we can benchmark the operating system performance of reading stream zero bytes in the kernel into a process memory and writing the bytes back to the kernel without any disk I/O. Note the throughput (~20GB/s) is strongly dependent on blocksize. For small block sizes the overhead of additional `read` and `write` system calls will  dominate.

```
> dd if=/dev/zero of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 0.0539153 s, 19.9 GB/s
```


Todo - move this to a different section
## What happens when I touch a file?
The `touch` executable creates file if it does not exist and also updates the file's last modified time to be the current time. For example, we can make a new private file with the current time:
```
> umask 077       # all future new files will maskout all r,w,x bits for group and other access
> touch file123   # create a file if it does not exist, and update its modified time
> stat file123
  File: `file123'
  Size: 0         	Blocks: 0          IO Block: 65536  regular empty file
Device: 21h/33d	Inode: 226148      Links: 1
Access: (0600/-rw-------)  Uid: (395606/ angrave)   Gid: (61019/     ews)
Access: 2014-11-12 13:42:06.000000000 -0600
Modify: 2014-11-12 13:42:06.001787000 -0600
Change: 2014-11-12 13:42:06.001787000 -0600
```

An example use of touch is to force make to recompile a file that is unchanged after modifying the compiler options inside the makefile. Remeber that make is 'lazy' - it will compare the modified time of the source file with the corresponding output file to see if the file needs to be recompiled
```
touch myprogram.c   # force my source file to be recompiled
make
```

## Virtual file systems
POSIX systems, such as Linux and Mac OSX (which is based on BSD) include several virtual filesystems that are mounted (available) as part of the file-system. Files inside these virtual filesysems do not exist on the exist; they are generated dynamically by the kernel when a process requests a directory listing.
Linux provides 3 main virtual filesystems
```
/dev  - A list of physical and virtual devices (for example network card, cdrom, random number generator)
/proc - A list of resources used by each process and (by tradition) set of system information
/sys - An organized list of internal kernel entities
```
## How do I find out what filesystems are currently available (mounted)?
Use `mount`
Using mount without any options generates a list (one filesystem per line) of mounted filesystems including networked, virtual and local (spinning disk / SSD-based) filesystems. Here is a typical output of mount

``
> mount
/dev/mapper/cs241--server_sys-root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/sda1 on /boot type ext3 (rw)
/dev/mapper/cs241--server_sys-srv on /srv type ext4 (rw)
/dev/mapper/cs241--server_sys-tmp on /tmp type ext4 (rw)
/dev/mapper/cs241--server_sys-var on /var type ext4 (rw)rw,bind)
/srv/software/Mathematica-8.0 on /software/Mathematica-8.0 type none (rw,bind)
engr-ews-homes.engr.illinois.edu:/fs1-homes/angrave/linux on /home/angrave type nfs (rw,soft,intr,tcp,noacl,acregmin=30,vers=3,sec=sys,sloppy,addr=128.174.252.102)
```
Notice that each line includes the filesystem type source of the filesystem and mount point.
To reduce this output we can pipe it into `grep` and only see lines that match a regular expression. 
```
>mount | grep proc  # only see lines that contain 'proc'
proc on /proc type proc (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
```

##Todo
sudo mount /dev/cdrom /media/cdrom
mount
mount | grep proc

Examples of virtual files in /proc:

cat /proc/sys/kernel/random/entropy_avail
hexdump /dev/random
hexdump /dev/urandom

Differences between random and urandom?

cat /proc/meminfo
cat /proc/cpuinfo
cat /proc/cpuinfo | grep bogomips

cat /proc/meminfo | grep Swap

cd /proc/self
echo $$; cd /proc/12345; cat maps
