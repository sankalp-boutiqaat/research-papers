# Byte ordering
---

Each memory address can store only **1 byte** at max.

So, if the data size is less than 1 byte than the data can be easily stored in a single address location.

However, if the data size exceeds 1 byte, the data needs to be stored in multiple address locations where each address location cannot hold more than 1 byte of data.

*There are two main techniques of doing this:*
 
- **Big Indian (MSB first) approach** => In this Most significant byte is stored first and then the least significant byte.

- **Little Indian (LSB first) approach** => In this Least significant byte is stored first and then the Most significant byte.

![Flow](https://en.wikipedia.org/wiki/Endianness)

> IN MSB -> 1001010 <-LSB

Since, Ipv4 addresses are **32bit(4 byte)** integers and port numbers are **16 bit(2byte)** integers lets see this with an example.

We will be using hexa(0x) notations in our example as it allows us to represent 4bits in one single digit.

Consider port number : 6100  
Binary  16bits       : 00010111.11010100
Hex                  :  1   7     D   4   = 17.D4
BigEndianStorage     : 17 -> MemX, D4 -> MemX+1
LittleIndian         : D4 ->MemX, 17 -> MemX+1

Consider IP address: 127.255.255.1
Binary             : 01111111.11111111.11111111.00000001
Hex                :  7    F    F   F   F    F    0   1 = 7F.FF.FF.01
BigEndian          : 7F -> MemX, FF -> MemX+1, FF -> MemX+2, 01 -> MemX+3
LittleEndian       : 01 -> MemX, FF -> MemX+1, FF -> MemX+2, 7F -> MemX+4


*OK fine, but why are we learning this?*

The problem is that some of the system architectures use Big endian while some uses Little endian. SO, when the multibyte data is transfered from 
one host to another it could easily get malformed if both the systems uses different ordering schemes. 
To resolve above issue, someone has done a good job and finalized that all the data passing over network should use BigEndian ordering. So, incase the host uses Little Endian ordering
it has to first convert the data to Big Endian ordering before passing it to the network. 
NOTE: BigEndian ordering is also called Network byte order as all networking protols uses it. Little endian is also called as intel order as most of the intels processors are little endian.

Following are the functions for converting byte order from host order to network order and vice versa. Incase host order == network order, these functions perform no action.
So, its advisable to always use these functions when receiving or sending data to network. This helps the application to function irrespective of the byte order used by host system.

hton() -> convert from host order to network order(used when sending data).
ntoh() -> convert from network order to host order(used when receiving data).
