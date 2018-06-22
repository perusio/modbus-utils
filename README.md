# modbus-utils

Modbus client and server command line tools based on [libmodbus](http://libmodbus.org/).

NOTE: Currently the RTU transport seems not to be working. It
generates a timeour when reading. Hence all
examples and documentation refers to the TCP transport. 

## Requirements

 1. [libmodbus](http://libmodbus.org/). In debian the
    [libmodbus5](https://packages.debian.org/buster/libmodbus5) 
    and
    [libmodbus-dev](https://packages.debian.org/buster/libmodbus-dev)
    packages need to be installed.
 2. To create the debian package you need also:
   + [fakeroot](https://packages.debian.org/buster/fakeroot)
   + [pkg-config](https://packages.debian.org/buster/pkg-config)
   + [build-essential](https://packages.debian.org/buster/build-essential)
   + [debhelper](https://packages.debian.org/buster/debhelper)
    
## Building

There are two options:

 1. Build the program: 

        make 

 2. Build the debian package. In the 

        fakeroot debian/rules binary
 
## Installation

 1. Default installation is under `/usr/bin`.
        make install
    To install under `/usr/local` do:
        PREFIX=/usr/local make install
 2. To install the Debian package:
        dpkg -i modbus-utils_*.deb
 
## Usage

### Server (Modbus slave)


```bash
modbus_server [--debug] -a<slave address> -m<transport> --di=<nbr> --co=<nbr> --ir=<nbr>\
    --hr=<nbr> [rtu-params]-b<baud rate> -d<data bits> -s<stop bits> -p<parity>\
    [tcp params]p<port> [<IP address>]
```
where: 
 + `--debug`: enable debugging.
 + `<slave address>`: the Modbus slave address. Any value between 1
   and 247. Default: `1`.
 + `<transport>`: `rtu` or `tcp`.
 + `--di`: discrete inputs (binary). Specifiy the number of discrete
   inputs. It can be specified in decimal or hexadecimal. Default: `100`.
 + `--co`: coils (binary). Specifiy the number of discrete
   inputs. It can be specified in decimal or hexadecimal. Default: `100`.
 + `--ir`: analog input registers. Specifiy the number of analog input
   registers. It can be specified in decimal or hexadecimal. Default:
   `100`.
 + `--ir`: analog holding registers. Specifiy the number of analog input
   registers. It can be specified in decimal or hexadecimal. Default:
   `100`.
 + the RTU parameters `[rtu-params]` should be specified for a RTU
   transport. Default: `b9600 d8 s1 peven`.
 + the TCP parameters `[tcp params]` should be specified for a TCP
   transport. Default: `p502`. In the case of using a TCP transport
   the **last** argument is the IP address of the server. 
   
#### Examples

+ Create a server (slave) with 32 holding registers, 10 coils, 0
  discrete inputs and 0 input registers using TCP on port 1502 on localhost.

```bash
   modbus_server -mtcp --di=0x0 --co=0xa --ir=0x0 --hr=0x20 p=1502 127.0.0.1
``` 

+ The same can be specified using decimal notation:

```bash
   modbus_server -mtcp --di=0 --co=10 --ir=0 --hr=32 p=1502 127.0.0.1
``` 
 
### Client (Modbus master)

```bash

modbus_client [--debug] -a<slave address> -m<transport> \
    -c<read count> -r<start address> -t<function code> -o<timeout> \
    [rtu-params]-b<baud rate> -d<data bits> -s<stop bits> -p<parity>\
    [tcp params]p<port> [<IP address>] [<write data>]
```
where: 
 + `--debug`: enable debugging.
 + `<slave address>`: the Modbus slave address. Any value between 1
   and 247. Default: `1`.
 + `<transport>`: `rtu` or `tcp`.
 + `<read count>`: how many registers/coils/inputs read or
   write. Default `1`.
 + `<function code>`: the 
 [Modbus function code](https://www.csimn.com/CSI_pages/Modbus101.html).
 + `<timeout>`: timeout of the read or write operation in
   milliseconds. Default `1000`.   
 + `<start address>`: start address of the read or write
   operation.
 + the RTU parameters `[rtu-params]` should be specified for a RTU
   transport. Default: `b9600 d8 s1 peven`.
 + the TCP parameters `[tcp params]` should be specified for a TCP
   transport. Default: `p502`. In the case of using a TCP transport
   the **last** argument is the IP address of the server.

### Examples

+ Write to the last three holding registers created above the data: 10, 23, 46.

```bash
modbus_client -mtcp -r0x18 -t0x10 -p1502 127.0.0.1 0xa 0x17 0x2e
```

+ Do the same using decimal notation:

```bash
modbus_client -mtcp -r0x18 -t0x10 -p1502 127.0.0.1 10 23 46
```

+ Read those same registers:

```bash
modbus_client -mtcp -c3 -r0x18 -t0x03 -p502 127.0.0.1
```

+ Make the first 5 coils be set to 0x1.

```bash
modbus_client --debug -mtcp -r0x0 -t0x0f -p502 127.0.0.1 0x1 0x1 0x1 0x1 0x1
```

+ Do the same using decimal notation:

```bash
modbus_client --debug -mtcp -r0x0 -t0x0f -p502 127.0.0.1 1 1 1 1 1
```

NOTE: If you try to write a value different from 0 or 1, since a
coil can only have single bit values, it will be ignored and set to 1
if bigger than 1.

+ Read those values:

```bash
modbus_client --debug -mtcp -r0x0 -c5 -t0x01 -p502 127.0.0.1 
```
 
## TODO

 + man page.
 + Fix the RTU transport.
 + systemd setup (service file) to launch the server.
 + Make it work in OS X.
