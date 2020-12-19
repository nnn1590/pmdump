# pmdump

pmdump is a simple tool that provides process memory acquisition on Linux or Android.
Pmdump dumps process memory with its header information from /proc/<pid>/maps file. Data is dumped either to the file or throughout the network.
  
## Usage

### To use pre-built binary

There are prebuilt pmdump binaries in /pmdump_prebuilt_bin folder. 
They can be used to dump a process memory.
To build, please refer to below the build instruction.

`pmdump_parser.py` is also provided, which is a useful script that parses the memory dump file.

### pmdump

pmdump is used to dump process memory. Running of pmdump may require root permission.

```bash
./pmdump [OPTION]... MODE[,MODE]... <pid>
./pmdump [OPTION]... MODE[,MODE]... <pid> <ip-address> <port>

Dumping process memory to 'output_pmdump.bin' file or network.
The dumped result contains /proc/<pid>/maps entries info and its memory contents.

Options
 --raw    Dumping only data without /proc/<pid>/maps info header
 --anon    Dumping only anonymous memory

Each MODE is of the form '[-+][rwxps]'. If no mode is given, don't care the permission

Example
 ./pmdump +r +w -x +p --anon 1928    # dump only 'rw-p' permission with no file-mapped memory.
 ./pmdump +w --raw 1928 127.0.0.1 1212    # dump only writable memory without header info.
 ```

### pmdump_parser.py

pmdump_parser is the script that parses the dump images created by pmdump.

```bash
Usage: pmdump_parser.py [--raw|-<number>] <pmdumped_file>

print maps information from the dump file if no option is given.

Option:
    --raw       export only data part without header information
    -number     export given entry number's memory region

Example:
    ./pmdump_parser.py output.bin           // show memory info like 'cat /proc/<pid>/maps
    ./pmdump_parser.py --raw output.bin     // output_raw.bin is generated
    ./pmdump_parser.py -10 output.bin       // output_10.bin is generated
```

## How to Build

### Android

Android NDK is required to build it. If Android SDK is installed, NDK-bundle that comes with Android SDK can be also used.

Run the following command.

```bash
cd pmdump_src
make -f Makefile.android (all|arm|arm64|x86|x86_64)
``` 

### Ubuntu

The build is simple. Just run gcc command or use the following Makefile

```bash
cd pmdump_src
make -f Makefile.host
```

## Example usages in Android

The following example is to show how to install pmdump on Android device and dump process memory.


1. adb root privilege requires running pmdump in Android

```bash
adb root
```

2. copy pmdump to the proper folder. /data folder is a good choice

```bash
adb push pmdump /data/pmdump
```

3. find the process id of the target process by using DDMS or ps command

```bash
adb shell ps
```

4. dump memory and copy it to the host

```bash
adb shell
$ cd data
$ ./pmdump +r +w -x +p <pid> 
$ exit
adb pull /data/output_pmdump.bin .
```

Or, dump memory and get it throughout the network

```bash
# in remote PC
nc -lvvv 1212 > dumpfile.bin

# in PC connected with Android
adb shell
$ cd data
$ ./pmdump +r +w -x +p <pid> 192.168.1.154 1212
$ exit
```

5. Play with the dump file

pmdump_parser.py provides the function of parsing the dump to show information about the dump file

```bash
python pmdump_parser.py output_pmdump.bin
```
