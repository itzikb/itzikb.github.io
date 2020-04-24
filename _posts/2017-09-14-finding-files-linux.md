---
layout: single
permalink: /finding-files-linux/
title: "Finding files in Linux"
tags: linux
comments: true
---

## Finding Files using the locate command
If you want to find files fast just by the name of the files, part of the name or some regular expression - my favorite one is the locate command.

First you need to install the mloacte package:
```shell
$ sudo dnf -y install mlocate
```
Then create the database:
```shell
$ sudo updatedb
```
**Find files by name**
```shell
$ locate which
/boot/extlinux/whichsys.c32
/etc/profile.d/which2.csh
/etc/profile.d/which2.sh
/usr/bin/which
/usr/share/doc/which
/usr/share/doc/which/AUTHORS
/usr/share/doc/which/EXAMPLES
/usr/share/doc/which/NEWS
/usr/share/doc/which/README
/usr/share/info/which.info.gz
/usr/share/licenses/which
/usr/share/licenses/which/COPYING
/usr/share/man/man1/which.1.gz
/usr/share/rubygems/rubygems/commands/which_command.rb
/usr/share/syslinux/whichsys.c32

```
The above will find all the files that include which in their names.

**Find the exact file name**
```shell
$ locate "*/which"
/usr/bin/which
/usr/share/doc/which
/usr/share/licenses/which
```

**Ignoring case**
```shell
$ locate -i authors
/usr/share/doc/cpio/AUTHORS
/usr/share/doc/cracklib/AUTHORS
/usr/share/doc/cronie/AUTHORS
/usr/share/doc/cyrus-sasl-lib/AUTHORS
/usr/share/doc/dbus/AUTHORS
...
```

**Note:** The locate command uses it's database to search for the files so in case it wasn't run for some time it may not have accurate results. Just make sure to run the updatedb command before doing the search.

## Using the find command
The find command is a very powerful. You can not only find files but also act on the results (e.g. Deleting those files).

**Find a file in the current directory**
```shell
$ find -name somefile.txt
```

**Find all the empty files**
```shell
$ find / -empty
```
**Using Regular expression**
```shell
$ find / -name *.mp4
```
**Output to a file**
```shell
find / -name .mp4 -fprint outputfile
```

**Ignoring case**
```shell
$ find / -iname '*.TXT'
```

**Find files that are just 2 levels deep**
```console
$ find ./mydir -maxdepth 2 -name "*.mp3"
./mydir/cool_songs/song1.mp3
./mydir/cool_songs/song2.mp3
./mydir/cool_songs/song3.mp3
./mydir/song4.mp3
```

**Find files by file type**
```shell
$ find -name which -type f
```
The above will find all the files named 'which'.

Find supports other types as well:
 f: regular file
 d: directory
 l: symbolic link
 c: character devices
 b: block devices


**Invert match**
```shell
$ find ./mydir -not -name "*.mp3"
./mydir
./mydir/somefile.txt
./mydir/somefile2.txt
```
The '-not' cab be replaced by '!'

**Using Or**  
Find all the mp3 and mp4 files in the current directory
```shell
$ find -name '*.mp3' -o -name '*.mp4'
./songs/song1.mp3
./songs/song2.mp3
./songs/song3.mp3
./videos/video1.mp4
./videos/video2.mp4
./videos/video3.mp4
```

**Search in multiple directories**
```shell
$ find /dir1 /dir2 -type f -name "*.txt"
```

**Finding files with permission 0755**
```shell
find / -maxdepth 2 -perm 0755 2>/dev/null
```
We ignore the permission denied errors by directing the standard error to /dev/null

**Finding file that are readable**
```shell
$ find /somedir -maxdepth 3 -perm /a=r
```

**Find files by size**

Find all the files with size greater that 1G
```shell
$ sudo find / -size +1G
```

**Ignore matches**
Ignore all the '.git' directories
```shell
$ find -type d -path '.git' -prune -o -print
```

**Putting files in a Tar archive**
```shell
$ find . -type f -name "*.mp3" | xargs tar rvf songs.tar
```

### Acting upon results
The powerfull aspect of the find command is acting upon the results. We'll see some examples:

**Show the top 10 largest files in a directory**
```shell 
$ find . -type f -exec ls -sh {} \; | sort -n -r | head -10
```
The "{}" is used as a placeholder for the files that find matches. The "\;" is used so that find knows where the command ends.

**Remove files (Be careful!)**
```shell
$ find /home/user -type f -name '*.tar.gz' -size +1G -exec rm -f {} \;
```

**Find all the log files that contain the Error string**
```shell
find . -type f -name "*.log" -exec grep -l Error {} \;    
```
We used '-l' to get just the file names and not the lines that contain them

**Copy one file to multiple directories**
```shell
$ find dir1 dir2 dir3 dir4 -type d -exec cp somefile.txt {} \;
```
