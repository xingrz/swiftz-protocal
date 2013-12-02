Swiftz Protocal
==========

**Swiftz** is a project aims to create a portable replacement of Edusupplicant, base on the analysis on Edusupplicant 3.6.5.

The project is **FOR STUDY ONLY**. Attacking, hacking or any other illegal purposes are **STRICTLY PROHIBITED**.

----------

## Protocal version

The original analysis is based on **Edusupplicant 3.6.5**.

Tested compatible with **Edusupplicant 3.7.8**.


## Implementations based on this analysis

* [SwiftzMac](https://github.com/xingrz/SwiftzMac) (for Mac OS X >= 10.8)
* [swiftz.js](https://github.com/xingrz/swiftzjs) (for Linux/Mac, command-line, STILL IN PROGRESS)
* [Swiftoid](https://github.com/xingrz/swiftz-android) (for Android >= 4.0.3, STILL IN PROGRESS)
* OpenSupplicant (for Windows, stopped development for a long time)

## Representation

The protocal in this documentation is written as **procedure**.

For example:

```
local:3848 -> server:3848 crypto3848
LOGIN:
  MAC as data(16)
  USERNAME as string
  PASSWORD as string
  IP as string
  ENTRY as string
  DHCP as boolean
  VERSION as string
```

1. Send to remote server port `3848` from local port `3848` (all packets send via `UDP`). 
2. The packet is encrypted with `crypto3848`.
3. The packet represents an `LOGIN` action (see [Consts / Actions](#actions) section below).
4. The packet contains these fields (see [Consts / Fields](#fields) section below):
  1. **MAC**: Binary data, with fixed length 16 bytes
  2. **USERNAME**: String
  3. **PASSWORD**: String
  4. **IP**: String
  5. **ENTRY**: String
  6. **DHCP**: Boolean
  7. **VERSION**: String


## Encoding

### Charset

All strings in the protocal is in `GB2312` charset.

### Types

- **String**: strings transport in `GB2312`.
- **Char**: number in 1 byte. (i.e. `0xFF` for `255`)
- **Integer**: number in 4 bytes. (i.e. `0x00 0x00 0x00 0xFF` for `255`)
- **Boolean**: `0x00` for `false`, `0x01` for `true`.
- **Data**: as it is. (i.e. `0x4A 0x21 0x39 0xC0` for `4A2139C0`)

### Packet

A naked (non-encrypted) packet is in this structure:

1. 1 byte of `ACTION` represented what does the packet do
2. 1 byte represented the length of whole packet
3. 16 bytes `MD5` hash
4. 1 byte of the `key` of the first field
5. 1 byte of the `length` of the first field (including the `key` and the `length` itself)
6. the `data` of the first field
7. 1 byte of the `key` of the second field (including the `key` and the `length` itself)
8. 1 byte of the `length` of the second field
9. the `data` of the second field
10. ......

Note that fields' `length` is 2 bytes shorter than the field is.

### Generate a packet

1. Construct a packet as it's descripted above, of course, leaving those 16 bytes of `MD5` hash be 16 bytes of `0x00`
2. Generate a `MD5` hash of the whole packet
3. Fill those 16 bytes with the hash you got
4. Encrypt it of course :)

### Parse a packet you recieved

1. Decrypt it
2. Check the `MD5` hash (optional but recommanded for security)
3. You know how to do :)

### Encryption/Decryption

#### "crypto3848"

The `crypto3848` algorithm is a very simple encryption algorithm that changes the order of every byte from `ABCDEFGH` to `HDEFCBAG`.

Here's a piece of code in JavaScript for reference:

```js
function encrypt (buffer) {
  for (var i = 0; i < buffer.length; i++) {
    buffer[i] = (buffer[i] & 0x80) >> 6
              | (buffer[i] & 0x40) >> 4
              | (buffer[i] & 0x20) >> 2
              | (buffer[i] & 0x10) << 2
              | (buffer[i] & 0x08) << 2
              | (buffer[i] & 0x04) << 2
              | (buffer[i] & 0x02) >> 1
              | (buffer[i] & 0x01) << 7
  }
}

function decrypt (buffer) {
  for (var i = 0; i < buffer.length; i++) {
    buffer[i] = (buffer[i] & 0x80) >> 7
              | (buffer[i] & 0x40) >> 2
              | (buffer[i] & 0x20) >> 2
              | (buffer[i] & 0x10) >> 2
              | (buffer[i] & 0x08) << 2
              | (buffer[i] & 0x04) << 4
              | (buffer[i] & 0x02) << 6
              | (buffer[i] & 0x01) << 1
  }
}
```

#### "crypto3849"

Like `crypto3848` but different order that from `ABCDEFGH` to `ECBHAFDG`.

```js
function encrypt (buffer) {
  for (var i = 0; i < buffer.length; i++) {
    buffer[i] = (buffer[i] & 0x80) >> 4
              | (buffer[i] & 0x40) >> 1
              | (buffer[i] & 0x20) << 1
              | (buffer[i] & 0x10) >> 3
              | (buffer[i] & 0x08) << 4
              | (buffer[i] & 0x04)
              | (buffer[i] & 0x02) >> 1
              | (buffer[i] & 0x01) << 4
  }
}

function decrypt (buffer) {
  for (var i = 0; i < buffer.length; i++) {
    buffer[i] = (buffer[i] & 0x80) >> 4
              | (buffer[i] & 0x40) >> 1
              | (buffer[i] & 0x20) << 1
              | (buffer[i] & 0x10) >> 4
              | (buffer[i] & 0x08) << 4
              | (buffer[i] & 0x04)
              | (buffer[i] & 0x02) << 3
              | (buffer[i] & 0x01) << 1
  }
}
```


## Flow

### Startup
Just send a [**initialization**](#initialize) packet to `1.1.1.8` without waiting for any response

### First-run initialization
1. Search for [**authorization server**](#search-for-authorization-server)
2. Get [**entries list**](#get-entries)

### Get online

1. Make a [**online**](#online) request, and wait for a response
2. If successed, some servers may also need a [**confirm**](#confirm)

### During you're online

1. Make a [**breathe**](#breathe) every 30 seconds
2. Also, listen for [**disconnecting**](#being-disconnected) notification you may recieved.

### Get offline
Make a [**offline**](#offline) request to end the session.


## Packets

### Initialize

Just send this plain text to `1.1.1.8` port `3850` via UDP.

```
info sock ini
```

* No decryption is required.
* No response would be make by the server.

### Search for authorization server

#### Send

```
local:3848 -> 1.1.1.8:3850 crypto3848
SERVER:
  SESSION as string
  IP as string(16)
  MAC as data(16)
```

#### Receive

```
1.1.1.8:3850 -> local:3848 crypto3848
SERVER_RET:
  SERVER as data(4)
  UNKNOWN0D as data(4)
```

* `SERVER` is received as 4-byte data that each byte indicates a part of an IPv4 address, e.g. `AC1001B4` for `172.16.1.180`.

### Get entries

#### Send

```
local:3848 -> server:3848 crypto3848
ENTRIES:
  SESSION as string
  MAC as binary(16)
```

#### Receive

```
server:3848 -> local:3848 crypto3848
ENTRIES_RET:
  ENTRY as string
  ENTRY as string
  ...
```

* It may be more than one `ENTRY` field.

### Online

#### Send

```
local:3848 -> server:3848 crypto3848
LOGIN:
  MAC as data(6)
  USERNAME as string
  PASSWORD as string
  IP as string
  ENTRY as string
  DHCP as boolean
  VERSION as string
```

#### Receive

```
server:3848 -> local:3848 crypto3848
LOGIN_RET:
  SUCCESS as boolean
  SESSION as string
  UNKNOWN05 as char
  UNKNOWN06 as char
  MESSAGE as string
  UNKNOWN95 as data(24)
  UNKNOWN06 as char
  BLOCK34 as data(4)
  BLOCK35 as data(4)
  BLOCK36 as data(4)
  BLOCK37 as data(4)
  BLOCK38 as data(4)
  WEBSITE as string
  UNKNOWN23 as char
  UNKNOWN20 as char
```

* Only contains `SUCCESS`, `SESSION` and `MESSAGE` if not successed (i.e. wrong password)
* Will contains an unknown field `UNKNOWN95` with 24 bytes of `0x00` and an unknown field `UNKNOWN06` if it is in low-speed mode.
* The `length` of field `SESSION` received is 2 bytes shorter than the field is.
* The `length` of field `MESSAGE` received is 2 bytes shorter than the field is.

### Confirm

#### Send

```
local:random -> server:3849 crypto3849
CONFIRM:
  USERNAME as string
  MAC as data(6)
  IP as string
  ENTRY as string
```

#### Receive

```
server:3849 -> local:random crypto3849
CONFIRM_RET
  UNKNOWN30 as data(4)
  UNKNOWN31 as data(4)
  UNKNOWN32 as char
```

* The packet is sent and receive via the same random port
* 3 bytes of `0x00` appears before the md5 checksum of the packet possibly by a BUG. The length of the whole packet is calculated with those 3 bytes included.

### Breathe

#### Send

```
local:3848 -> server:3848 crypto3848
BREATHE:
  SESSION as string
  IP as string(16)
  MAC as data(16)
  INDEX as integer
  BLOCK2A as data(4)
  BLOCK2B as data(4)
  BLOCK2C as data(4)
  BLOCK2D as data(4)
  BLOCK2E as data(4)
  BLOCK2F as data(4)
```

* The initial value of `INDEX` is `0x01000000` and increases 3 after every breathe.
* `INDEX` should not be increased unless a valid `BREATHE_RET` is recevied, because you may lost packets in low-speed mode.

#### Receive

```
server:3848 -> local:3848 crypto3848
BREATHE_RET:
  SUCCESS as boolean
  SESSION as string
  INDEX as integer
```

* The `length` of field `SESSION` received is 2 bytes shorter than the field is.

### Offline

#### Send

```
local:3848 -> server:3848 crypto3848
LOGOUT:
  SESSION as string
  IP as string(16)
  MAC as data(16)
  INDEX as integer
  BLOCK2A as data(4)
  BLOCK2B as data(4)
  BLOCK2C as data(4)
  BLOCK2D as data(4)
  BLOCK2E as data(4)
  BLOCK2F as data(4)
```

* The initial value of `INDEX` is `0x01000000` and increases 3 after every breathe.
* `INDEX` should not be increased unless a valid `BREATHE_RET` is recevied, because you may lost packets in low-speed mode.
* It's very like a `BREATHE` packet.

#### Receive

```
server:3848 -> local:3848 crypto3848
LOGOUT_RET:
  SUCCESS as boolean
  SESSION as string
  INDEX as integer
```

* The `length` of field `SESSION` received is 2 bytes shorter than the field is.

### Being disconnected

Once you have bean disconnected, you'll receive a packet from the server:

```
server:2001 -> local:4999 crypto3848
DISCONNECT:
  SESSION as string
  REASON as char
```

* Possible values of `REASON`:
  * `0x00` - No valid breathe is made for a long time.
  * `0x01` - You're disconnected forcibly (by administrator, maybe).
  * `0x02` - You've reached your network traffic limit.
* The `length` of field `SESSION` received is 2 bytes shorter than the field is.


## Consts

### Actions

```
// login
LOGIN = 0x01

// login result
LOGIN_RET = 0x02

// breathe
BREATHE = 0x03

// breathe result
BREATHE_RET = 0x04

// logout
LOGOUT = 0x05

// logout result
LOGOUT_RET = 0x06

// get access point
ENTRIES = 0x07

// return access point
ENTRIES_RET = 0x08

// disconnect
DISCONNECT = 0x09

// confirm login
CONFIRM = 0x0A

// confirm login result
CONFIRM_RET = 0x0B

// get server
SERVER = 0X0C

// return server
SERVER_RET = 0x0D
```

### Fields

```
// username
USERNAME = 0x01

// password
PASSWORD = 0x02

// whether it is success
SUCCESS = 0x03

// unknown, appears while login successfully
UNKNOWN05 = 0x05
UNKNOWN06 = 0x06

// mac address
MAC = 0x07

// session (NOTE: wrong in return packet)
SESSION = 0x08

// ip address
IP = 0x09

// access point
ENTRY = 0x0A

// message (NOTE: wrong in return packet)
MESSAGE = 0x0B

// server ip address
SERVER = 0x0C

// unknown, appears while received server ip address
UNKNOWN0D = 0x0D

// is dhcp enabled
DHCP = 0x0E

// self-services website link
WEBSITE = 0x13

// serial no
INDEX = 0x14

// version
VERSION = 0x1F

// unknown, appears while login successfully
UNKNOWN20 = 0x20
UNKNOWN23 = 0x23

// disconnect reason
REASON = 0x24

// 4 bytes blocks, send in breathe and logout
BLOCK2A = 0x2A
BLOCK2B = 0x2B
BLOCK2C = 0x2C
BLOCK2D = 0x2D
BLOCK2E = 0x2E
BLOCK2F = 0x2F

// unknown 4 bytes blocks, appears while confirmed
BLOCK30 = 0x30
BLOCK31 = 0x31

// unknown
UNKOWN32 = 0x32

// 4 bytes blocks, appears while login successfully
BLOCK34 = 0x34
BLOCK35 = 0x35
BLOCK36 = 0x36
BLOCK37 = 0x37
BLOCK38 = 0x38
```
