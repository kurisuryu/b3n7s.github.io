---
layout: post
title:  "How to install Intel Pin on Fedora"
date:   2018-08-08 10:30:00 +0100
categories: 
---

# How to install Intel Pin on Fedora

Install dependencies:

```
$ sudo yum install gcc-c++
```

Download latest version from official website: https://software.intel.com/en-us/articles/pin-a-binary-instrumentation-tool-downloads

E.g.:

```
$ wget https://software.intel.com/sites/landingpage/pintool/downloads/pin-3.7-97619-g0d0c92f4f-gcc-linux.tar.gz
```

Extract the archive:

```
$ tar xzvf pin-3.7-97619-g0d0c92f4f-gcc-linux.tar.gz
$ cd pin-3.7-97619-g0d0c92f4f-gcc-linux
```

We will use the pintool `Insmix` to do a test. We now have to compile this pintool.

Go to its directory:

```
$ cd source/tools/Insmix
```

We have to change a compilation flag so compilation does not fail on warnings: 

```
$ sed -i -e 's/-Werror//g' ../Config/makefile.unix.config
```

Now compile the pintool:

```
$ make
```

The compiled pintool will be in a subdirectory named `obj-intel64`.

Now let's go back to the root of the extracted folder:

```
$ cd ../../..
```

and instrument `ls` as an example:

```
$ ./pin -t source/tools/Insmix/obj-intel64/insmix.so -- ls
```

This will create 2 new files: `bblcnt.out` and `insmix.out` with the instrumentation results.








