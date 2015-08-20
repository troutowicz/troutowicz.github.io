---
layout: post
title: Files and orphaned data
---

So here is a fun little scenario. I was alerted to a near full partition.

```sh
df -B1 /var/log/

Filesystem         1B-blocks   Used       Available Use% Mounted on
/dev/mapper/lv_log 10186764288 9679290368 0         100% /var/log
```

A common culprit is a monstrous log file that has not been rotated.

```sh
du -hs /var/log/* | sort -rh | head -20

996M    /var/log/httpd
76M     /var/log/wtmp-20150801
45M     /var/log/wtmp
30M     /var/log/audit
19M     /var/log/sa
7.1M    /var/log/messages-20150802
7.0M    /var/log/messages-20150816
7.0M    /var/log/messages-20150809
7.0M    /var/log/messages-20150726
6.5M    /var/log/secure-20150816
6.5M    /var/log/secure-20150809
6.5M    /var/log/secure-20150802
6.5M    /var/log/secure-20150726
6.4M    /var/log/rhsm
3.3M    /var/log/messages
3.0M    /var/log/secure
456K    /var/log/dracut.log-20140101
416K    /var/log/secure-20140518.gz
412K    /var/log/secure-20140504.gz
336K    /var/log/secure-20140511.gz
```
Despite the questionable size of `/var/log/httpd`, lack of log rotation did not seem to be the issue here. So what is the total file space usage for `/var/log`?

```sh
du -bs /var/log/

1277777466      /var/log/
```

So the file system for this partition is filled with `~9.68 GB` of data, but only `~1.28 GB` worth of files exist??

After some thinking, I recalled some of the more interesting details of the Linux VFS. Files are an illusion. A file is just a virtual representation of data chunks found on the physical file system. Simply put, the kernel exposes data through files. Normally when a file is deleted the related data chunks get removed from the file system. However, if a file gets deleted while a running process has the file opened, the data chunks will remain until the process has closed the file.

```sh
lsof -a +L1 /var/log/

COMMAND   PID     USER   FD    TYPE  DEVICE  SIZE/OFF     NLINK  NODE     NAME
...
...
...
httpd     30425   root   14w   REG   253,3   2075451392   0      389479   /var/log/httpd/api_log-20150719 (deleted)
httpd     30425   root   15w   REG   253,3   2075451392   0      389479   /var/log/httpd/api_log-20150719 (deleted)
httpd     30425   root   16w   REG   253,3   2075451392   0      389479   /var/log/httpd/api_log-20150719 (deleted)
httpd     30425   root   17w   REG   253,3   2075451392   0      389479   /var/log/httpd/api_log-20150719 (deleted)
httpd     30425   root   18w   REG   253,3   6302420992   0      389420   /var/log/httpd/log-20150714 (deleted)
httpd     30425   root   19w   REG   253,3   6302420992   0      389420   /var/log/httpd/log-20150714 (deleted)
httpd     30425   root   20w   REG   253,3   6302420992   0      389420   /var/log/httpd/log-20150714 (deleted)
httpd     30425   root   21w   REG   253,3   6302420992   0      389420   /var/log/httpd/log-20150714 (deleted)
```

Aha, someone deleted log files that are currently opened by a process! The above output excludes the rows pertaining to children processes to make things easier to digest. There are two opened files, one with a size of ~2.08 GB, the other with a size of ~6.30 GB.

`2.08 + 6.30 + 1.28 = 9.66 (close enough)`

A restart of the service will close all files that were opened by the process, allowing the kernel to remove any orphaned data from the file system.
