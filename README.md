swiftz-protocal
==========

**Swiftz** is a project aims to create a portable replacement of Edusupplicant, base on the analysis on Edusupplicant 3.6.5.

The project is **FOR STUDY ONLY**. Attacking, hacking or any other illegal purposes are **STRICTLY PROHIBITED**.

----------

## Protocal version

**The version of Edusupplicant based on:** `3.6.5`


## Representation

The protocal in this documentation is written as **procedure**.

For example:

    SendTo: 172.16.1.180:3848, UDP
    Encrypt: Crypto3848
    Action: LOGIN
    Data:
      MAC as hex(16)
      USERNAME as string
      PASSWORD as string
      IP as string
      ACCESSPOINT as string
      DHCP as boolean
      VERSION as string

- Send to remote `172.16.1.180` port `3848` in `UDP`.
- The packet should be encrypted with `Crypto3848` before sending.
- The action the packet represented is `ACTION` (see **Consts** section below).
- The packet contains these fields (see **Consts** section below):
  * **MAC**: Hexadecimal, with fixed length 16
  * **USERNAME**: String
  * **PASSWORD**: String
  * **IP**: String
  * **ACCESSPOINT**: String
  * **DHCP**: Boolean
  * **VERSION**: String


## Encoding

### Charset

All strings in the protocal is in `GB2312`.

### Packet

A naked (non-encrypted) packet is in this structure:

- 1 byte of `ACTION` represented what does the packet do
- 1 byte represented the length of whole packet
- 16 bytes `MD5` hash
- 1 byte of the `key` of the first field
- 1 byte of the `length` of the first field
- the `data` of the first field
- 1 byte of the `key` of the second field
- 1 byte of the `length` of the second field
- the `data` of the second field
- ......

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

See `examples` directory.


## Flow

1. Send a initialization packet to `1.1.1.8` without waiting for any response
2. Search for authorization server (optional)
3. Request for access points list (optional)
4. Make a **login** request, and wait for a response
5. If successed, make a **breathe** every 30 seconds

After it is logged in, you should listen at udp port `4999` for disconnecting announcement you may recieved.

Make a **logout** request to end the session.


## Packets

(To be continued)


## Consts

### Actions

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
    ACCESSPOINT = 0x07

    // return access point
    ACCESSPOINT_RET = 0x08

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

### Fields

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
    ACCESSPOINT = 0x0A

    // message (NOTE: wrong in return packet)
    MESSAGE = 0x0B

    // server ip address
    SERVER = 0x0C

    // is dhcp enabled
    DHCP = 0x0E

    // self-services website link
    WEBSITE = 0x13

    // serial no
    SN = 0x14

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
