# Disk Image Acquisition Using dd

Image acquisition is the process of extracting and copying the data found on a hard disk which can then be used in a forensics analysis. This allows us to keep a pristine copy of the original drive and perform the actual analysis on the copy in order to preserve the integrity of the evidence.

In this example we will use `dd` to perform the extraction. This tool is a Linux tool that is used for copy or *duplicating* drives.

## First Steps

Before we begin working with duplicating drives we can get a taste of what `dd` does by creating a file and extracting data from `/dev/urandom` and place the output into the file:

```
root@localhost:~# touch randomData.txt

root@localhost:~# dd if=/dev/urandom of=/root/randomData.txt bs=1M count=5
5+0 records in
5+0 records out
5242880 bytes (5.2 MB, 5.0 MiB) copied, 0.141211 s, 37.1 MB/s

root@localhost:~# head randomData.txt 
b?B9%^o|ݾ
8NF[,oHS.%:eQ}
              /ר/w
                  9yr26Ժ]K*^8
%?NGh(<J8qyA2,xu
 cL     gLv
qXF ?@J<ܗbU9WtjEkcTl''K2#nb{k7Q6ܞ
                                 ;%K8b\ceΉkhGoe#Ujb8"ZY2>
                                                         <jogXN,xr&Y    G]9Q3>R R+w8(f  ݰL(rZܖ+b՟Wx 3)+j⫖?ʞ'MR  #C&ڟ1׸0Oh(]ѝA9P'⍽!cN9k[Znz}  T]k1&/W#`p^!+A!0q2?lƊ2;Qu̲G%~ÝϿIqїSj>EUQ$z^KAT,uzY*8b=7μ\X
=+O9..2ǉlА1N,&Otu[-Q    ?erXH.>*FoY)c=x\9JF-޹t7OL6u9S
;ӮChΞ~NO^E"7@[׼r `L;m.MN&-""A"eKI.;v,SgFI,ivxZ'
H_T鈲
     B  S"Ub`   ?`WAf(wE
#!w
   f2suLrI繱@jVv;K7#Bd#K¾CE9%JiO?E
                                  DLEbθWBD-ܠ4}m
$kWw04ŀT+#yQ)/LzQTZy
~#t@=1V2TuXI;s1#8pCx3~8[|G</w,󬚻9H(؏aޤ!_
/                       ˞ρOt5L!ʠ6&<p@:^$mT8     3\W\=ZЫ:f^ۚ;69
 jJNNGK]UUHKmY54ǟt~';ǁ,t\Y(]vi6 !Um)9*gq^*8/okJMeLQTPu#Iݻ89PDͧ>-[-%7oݓ3?r56UTd
                                                                             ?FqDTD(jM(;v|ZY
#m!cB7yC?u+s5<X6OPb/ͤkfMnu8>-Sso9;uƩWp                                                       &uicAMdAI>")g8Q{Q6%hNd/q3НzQѭSP9$

```

In the above example we used a few parameters the `dd` command is capable of, such as defining the `bs` (block size) as 1*M* (for megabytes, or in this case 1 megabyte) and a `count` of 5, which means `dd` extracted 1MB of data from `/dev/urandom` a total of 5 times. `if=` is what I used to define the source data and `of=` to define the destination of this data. There is also a display at the end of the command giving information on how long the process took and how much data was moved as well as how quickly.

## Duplicating a Drive

Knowing that we can define a source and destination for the data, this tool can be used to duplicate a drive. This will allow us to make and exact copy of a drive for forensic analysis or even making and reverting from back up drives:

```
root@localhost:~# df -h 
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       2.0G  1.6G  212M  89% /
devtmpfs        1.5G     0  1.5G   0% /dev
tmpfs           1.5G     0  1.5G   0% /dev/shm
tmpfs           1.5G  448K  1.5G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.5G     0  1.5G   0% /sys/fs/cgroup
/dev/sdb        976M  7.6M  902M   1% /root
/dev/sdc        240M   95M  129M  43% /mnt/evidence
tmpfs           300M     0  300M   0% /run/user/0

root@localhost:~# umount /mnt/evidence/

root@localhost:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       2.0G  1.6G  212M  89% /
devtmpfs        1.5G     0  1.5G   0% /dev
tmpfs           1.5G     0  1.5G   0% /dev/shm
tmpfs           1.5G  448K  1.5G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.5G     0  1.5G   0% /sys/fs/cgroup
/dev/sdb        976M  264M  646M  29% /root
tmpfs           300M     0  300M   0% /run/user/0

root@localhost:~# dd if=/dev/sdc of=evidence.img
524288+0 records in
524288+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 16.1871 s, 16.6 MB/s

root@localhost:~# ls
evidence.img  randomData.txt
```

Using `df -h` we get a listing of all drives currently mounted on the system. The following line is our target drive: `/dev/sdc        240M   95M  129M  43% /mnt/evidence`. We need to first unmount the drive from the system using `umount /mnt/evidence`. We will next issue the `dd` command by specifying `/dev/sdc` as the source and the file name `evidence.img` as the destination. After copying the data you can take gather information on the file-system by using the `file` command:

```
root@localhost:~# file evidence.img 
evidence.img: Linux rev 1.0 ext4 filesystem data, UUID=6b832003-6bf1-4786-a3c1-e252e8f99090 (extents) (64bit) (large files) (huge files)
```

### Hashing the Image

You may need to create a hash of this `img` file for later use. By creating a hash of the file we can ensure the integrity of the data is good by making sure they match. If the hashes do not match, then the file/filesystem has been altered:

```
root@localhost:~# sha512sum evidence.img 
4596547098f9e126c2d4a1be301154fe7444c9db20adf4de524ec3c9c48a6de75c57089404bc406d04fa3ae81bbaaba556a2a3ae0fdb8bec7c5fc94f41ed6880  evidence.img
root@localhost:~# md5sum evidence.img 
cb8e1c51f51ea46bb4b5c2bb4998c987  evidence.img
```

The commands above show the both the `md5` and `sha512` of the `evidence.img` file. If at a later time we wanted to verify the data we could run this command again, and as stated, if the hash has changed then the composition of the data has also changed.
