---
layout: post
title: Filtering with rsync
---

When I need to sync two directories, [rsync](http://linux.die.net/man/1/rsync) is usually my first tool of choice. With it's vast number of options, rsync can be helpful in many file syncing situations. Take for example this directory structure.

```
/
├── dir1
│   └── file4
├── dir2
│   ├── dir4
│   │   └── file1
│   ├── file1
│   ├── file2
│   └── file3
└── dir3
    ├── dir4
    │   └── file1
    ├── file1
    ├── file2
    └── file3
```

Let's say that I need to copy the entirety of `dir1`, all files with names matching `file1` or `file2`, and exclude all directories with the name `dir4`. I also want to maintain the directory structure while ignoring empty directories. Taking a look at rsync's [manual](http://linux.die.net/man/1/rsync), a few options look helpful.

```
--exclude=PATTERN           exclude files matching PATTERN
--include=PATTERN           don't exclude files matching PATTERN
-m, --prune-empty-dirs      prune empty directory chains from file-list

-a, --archive               archive mode; equals -rlptgoD (no -H,-A,-X)
-z, --compress              compress file data during the transfer
-v, --verbose               increase verbosity
```

These options provide everything needed to meet the requirements for this task.

```sh
rsync -azvm src/ dst \
--include=*/ \              # Include all first level directories
--include=/dir1/** \        # Include the contents of first level dir1/
--exclude=/*/dir4/ \        # Exclude second level dir4/
--include=/*/file1 \        # Include second level file1
--include=/*/file2 \        # Include second level file2
--exclude=*                 # Exclude everything else
```
```
building file list ... done
created directory dst
./
dir1/
dir1/file4
dir2/
dir2/file1
dir2/file2
dir3/
dir3/file1
dir3/file2

sent 474 bytes  received 154 bytes  1256.00 bytes/sec
total size is 0  speedup is 0.00
```

This method works fine if the plan is to put the command in a script for repeated use. But what if we want to interactively test the options on the command line? The above solution can be typed on the command line just fine, but the escape characters and long option names leave alot to be desired. What if we explicitly use rsync's `filter` option and provide the filters within a [here doc](http://tldp.org/LDP/abs/html/here-docs.html)?

```
-f, --filter=RULE           add a file-filtering RULE
```

```sh
rsync -azvmf '. -' src/ dst <<EOF
+ */
+ /dir1/**
- /*/dir4/
+ /*/file1
+ /*/file2
- *
EOF
```

This method allows filters on multiple lines without escape characters, and provides a more natural way to include/exclude files. I think that this solution is preferrable, in every case, with the former solution. Not only are the latter's filters more visually appeasing, but they are also easier to type and interpret.
