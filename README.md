# linux-rootkits

A collection of Linux kernel rootkits found across the internet taken and put together, with a short report on how they work.
The README's in each folder contain the report about the rootkit sample.

## Rootkits

|Name|Short Description|link to orignal repo|
|:-:|:-:|:-:|
|[Puszek](Puszek/)|rootkit which can log requests and prevent itself from being rmmod'd|[Eternal's repo](https://github.com/Eterna1/puszek-rootkit)|
|[Reptile](Reptile/)|rootkit which can give root privs to users and a backdoor| [f0rb1dd3n's repo](https://github.com/f0rb1dd3n/Reptile)|


## Test Environment

All rootkits are tested in a fresh Ubuntu 16.04 VM where `uname -a` returns `Linux r3x-VirtualBox 4.15.0-112-generic #113~16.04.1-Ubuntu SMP Fri Jul 10 04:37:08 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux`


## Disclaimer

The rootkits were written by their respective authors and the code belongs to them. This is just an attempt to preserve rootkits with a report on how they work and the OS they were tested on.

If your rootkit is present here and you wish it to be removed please contact me at `siddharth.muralee[AT]gmail[DOT]com`
