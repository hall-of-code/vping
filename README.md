# `vping` for Vlang
This is a Ping Library for the Vlang Programming-Language (Link: [vlang/v](https://github.com/vlang/v)).
`vping`  uses the "native" `ping` and `traceroute|tracert` commands of Linux and Windows (!Currently <ins>not</ins> well tested on Windows, and not compatible with MacOS. Tested on Ubuntu 20 based system).

## Changelog
### __-- Version 0.3 --__
- Added (early stage) Support for `Traceroute`
- Traceroute (including the Docs for that) can be found in `/traceroute` as ___"submodule"___
- Import Traceroute via `import vping.traceroute`
- Traceroute has its own Structs called `ParsedTraceAnswer` and newly introducing `AbstractTraceAnswer`.
Unfortunatly `ParsedTraceAnswer` and the TraceParser isn't implemented yet.
- But it should work :)
  
#### Traceroute Docs in this [subfolder](traceroute/traceroute.md). No need to install it separately since its part of `vping`.


#### (!) Project in early stage of Development - use it with care. Best Regards _Theo.

## Quick Documentation
### Install it:
To install I'll recommend to use __vpm__.  
Which is just this one command: 
```shell
v install hall_of_code.vping
```
to install it including all dependencies.
The dependencies are `os` (Builtin) and `pcre`.  
__After installation__, you can simply use it by:
```vlang
import hall_of_code.vping
```
But now to the more 'nerdy' part - the Documentation.
### Basic Usage:

```vlang
module main 

import hall_of_code.vping

fn main() {
    result := vping.ping(ip: "google.com") //returns an vping.Answer Object
    println(result.parsed.mam_line) // "min/avg/max/mdev = 16.865/17.175/17.436/0.248 ms"
    println("The AVG Ping after ${result.parsed.pk_send} packages send, was ${result.parsed.avg}ms!")
}
```

#### Arguments:

| Argument | Description | Importance              | Example |
|----------|-------------|-------------------------|---------|
| `ip`       | The IP-Adress/Hostname/Domain of the Host                                                              | `required`              | `ping(ip: "google.com")`                           |
| `count`    | Number of Packets to send                                                                              | `(optional)`            | `ping(ip: "google.de", count: 4)`                  |
| `timeout`  | On Windows this is the time per Packet in MS - on Linux its for the whole Ping Process and in Seconds. | `(optional)`            | `ping(ip: "google.de", timeout: 10)`               |
| `size`     | Size per Packet to send in Byte                                                                        | `(optional)`            | `ping(ip: "google.de", size: 16)`                  |
| `force`    | Force to use IPv4 / IPv6                                                                               | `(optional)`            | `ping(ip: "google.de", force: 4) //forces to IPv4` |
| `ttl`      | Given TTL                                                                                              | `(optional)`            | `ping(ip: "google.de", ttl: 15)`                   |
| `iface`    | Network Interface Name (Linux-Only)                                                                    | `(optional/Linux-Only)` | `ping(ip: "google.de", iface: "eth0"`              |
| `interval` | Interval of packets in Seconds (Linux-Only)                                                            | `(optional/Linux-Only)` | `ping(ip: "google.de", interval: 1)`               |

## Functions:
Only public functions are documented here :) __//everything else -> todo for later__
### `ping(Conf) Answer`
This function takes a __Conf__-Struct as Parameter - since this is comparable with "named arguments", you also can
do:
```vlang
ping(ip: "google.de", count: 2, timeout: 5)
```
The Command takes some time - (while running the Ping Process by os.execute) and then - returns an `Answer{}`
Struct/Object.

### `(Answer) no_newline_raw() Answer`
__(Just for visualization-purpose).__
This function is callable on Answer-Object, and returns a new Answer where `<Answer>.raw` all `"\n"` replaced by `" "`.  
__Example:__
```vlang
module main

import hall_of_code.vping

fn main() {
    answ := vping.ping(ip: "google.com", timeout: 5, count: 2).no_newline_raw()
    println(answ.raw)
}
```

### `Answer{}`-Struct:
```vlang
pub struct Answer {
    mut:
        conf Conf      //given Conf (line {ip: "google.de", count: 3})
        command string //the generated ping-command
        status int     //status code (exit_status) -> 0 = everything ok, 1 = packetloss, 2 = ping-error
        raw string     //full terminal-response as string
        parsed Parsed  //parsed response, here it gets interesting ;)
}
```
### `Parsed{}`-Struct:
```vlang
pub struct Parsed {
	pub mut:
	time_tt int = -1          //in ms
	pk_send int = -1          //number of packages send
	pk_recv int = -1          //number of packages received
	pk_loss_percent int = -1  //package loss in percent%
	mam_line string = "-1"    //min/max/avg full line
	avg f32 =  -1             //ms
	min f32 =  -1             //ms
	max f32 =  -1             //ms
	mdev f32 = -1             //ms
    //vping specific error (NOT comparable with Status-code (exit code) of ping command):
	is_err int                //if parse() error [vping specific] 1 = yes, 0 = no error detected
}
```
__!! Parser should only work properly for Linux right now - so for Windows you have to parse the Answer.raw yourself !!__ //todo

## Further Examples:

Force to use IPv6 Ping:
```vlang
module main

import hall_of_code.vping

fn main() {
	answ := vping.ping(ip: "ipv6.google.com", force: 6, timeout: 10)
	if answ.status == 0 {
		println(answ.parsed.avg.str()) //print the average time it took
	} else if answ.status == 1 {
		println("Packet Loss: " + answ.parsed.pk_loss_percent.str() + "%") //print the Packet Loss in %
	} else {
		println("Error: Status-Code = " + answ.status.str()) //print the Status Code
		println(answ.raw) //print out ping response from terminal as string
	}
}
```

Use Conf-Struct as Parameter:
```vlang
module main

import hall_of_code.vping

fn main() {
	params := vping.Conf{
		ip: "google.de"
		count: 6
		timeout: 10
		interval: 2
	}

	println(vping.ping(params))
}
```

## Good to know:
When some values of __Conf__ or __Parsed__ are `== -1` there would be a parsing/general problem.  
__Edit:__ For that case its now Parsed.is_err implemented (Linux only), if nothing got parsed/changed this value is 1. So feel free
to use 
```vlang
if result.parsed.is_err == 1 { println('ping result is not valid for usage - parser cant parse the result (raw)')}
```
Since just few values __shouldn't__ end up as an Error or Failure, it's implemented like that. So when using the 
__Answer__ make sure to proof if the value is `>= 0` to "validate" it. 


