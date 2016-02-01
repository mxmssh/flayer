# What is . . . #

## Flayer ##

Flayer is a Valgrind plugin which traces (tainted) input through a program. In addition, it allows for conditional branches and function calls on the way to be skipped or altered.

For more detail, check out the [paper](http://www.google.com/search?hl=en&lr=&q=%22Flayer%3A+Exposing+Application+Internals%22).

## MKF ##

MKF is a fun little hack. It is used to apply the conditional branch alterations and function call site skipping that flayer does using only ptrace. It takes the same arguments, but means that none of the valgrind/flayer overhead is imposed!

# Installing #

Currently, Flayer is available for [download](http://code.google.com/p/flayer/downloads/list) as an [Encap](http://encap.org/) package.  The [source](http://code.google.com/p/flayer/source) is also available.

# Prerequisites #

The packages have been tested on modern 32-bit Linux installations.  At the time of writing, Flayer only supports x86 architecture system calls, but this may be fixed in the future.

To use the Encap package, epkg must be installed.  epkg and directions for installing it can be found at the [encap site](http://encap.org/epkg/#Source).

## Installing Flayer and MKF ##

Using epkg, it is as easy as just running these commands:
```
  sudo epkg /path/to/flayer-WOOT-encap-ix86-linux2.6.tar.gz
  sudo epkg /path/to/mkf-0.0.1-encap-ix86-linux2.6.tar.gz
```


# Running #

Flayer and MKF are command-line tools only. Flayer is based on the valgrind tool memcheck and its output is very similar. Any tools or scripts you may have for manipulating that output may be useful with flayer as well.

## Flayer ##

To run flayer without tainting any input, you can invoke it as follows:
```
  valgrind --tool=flayer /usr/bin/md5sum /etc/passwd
```

If you'd like to taint file input, you can add the following:
```
  valgrind --tool=flayer --taint-file=yes /usr/bin/md5sum /etc/passwd
```

You may want to filter out linked library loads, etc:
```
  valgrind --tool=flayer --taint-file=yes --file-filter=/etc/passwd /usr/bin/md5sum /etc/passwd
```

Perhaps you'd like to make md5sum not print the hash by skipping printf:
```
  valgrind --tool=flayer --taint-file=yes --file-filter=/etc/passwd --alter-fn=0x8049FB1:0 /usr/bin/md5sum /etc/passwd
```

Just as well, conditional branch behavior can be changed:
```
  valgrind --tool=flayer --taint-file=yes --file-filter=/etc/passwd --alter-branch=0x44702062:1 /usr/bin/md5sum /etc/passwd
```

In addition to file input, Flayer can also taint network input:
```
  valgrind --tool=flayer --taint-network=yes /usr/bin/curl -i http://www.google.com -o /tmp/curl.out
```

There is no filtering support on network tainting so behavior such as hostname resolution will turn up even if it is not the intended target of the trace.


## MKF ##

MKF supports the same arguments flayer does. For example, you can skip the printf function the same way:
```
  mkf --alter-fn=0x8049FB1:0 /usr/bin/md5sum /etc/passwd
```

Or, alter the conditional jump:
```
  mkf --alter-branch=0x44702062:0 /usr/bin/md5sum /etc/passwd
```

In the above example, it is critical to note that the branch behavior value was 0 instead of 1 as it was with Flayer. Since Flayer relies on Valgrind's intermediate representation of binary instructions, the jump behavior does not always match the behavior MKF enforces when it patches the x86 binary instructions. When using mkf, be sure to check that you are getting the desired behavior.


## Other components ##

More documentation for using these tools and the included Python library and interactive shell will be coming in the future.  User submissions are welcome!

