# linux-rootkits

A collection of Linux kernel rootkits found across the internet taken and put together, with a short report on how they work.
The README's in each folder contain the report about the rootkit sample.

## Rootkits 

|Name|Short Description|link to orignal repo|
|:-:|:-:|:-:|
|[Puszek](Puszek/)|rootkit which can log requests and prevent itself from being rmmod'd|[Eternal's repo](https://github.com/Eterna1/puszek-rootkit)|
|[Reptile](Reptile/)|rootkit which can give root privs to users and a backdoor| [f0rb1dd3n's repo](https://github.com/f0rb1dd3n/Reptile)|
|[Khook](Khook/)| (Not a rootkit) but an engine that can be used to hook functions, also mentioned here since it's used by rootkits to hook functions| [milabs's repo](https://github.com/milabs/khook))|

If you plan to download the latest version of these rootkits please download them from their original repo, as it would be the latest version.

## Features Descriptions

|Name|Short Description|Rootkits|links to code samples|
|:-:|:-:|:-:|:-:|
|Finding Syscall Table address (1)| Search memory for the pointer table! using a address of syscall function (eg. close) as reference|[Puszek](Puszek/)| [In Puszek](Puszek/rootkit.c#L1004)|
|Function Hooking (1)| Get the address of the function to be hooked and then Modify CR0 to remove write protect bit and then add a jump instruction to the hooked function| [Khook](khook/) and [Reptile (uses Khook)](Reptile/) | [in Khook](Khook/x86/hook.c#L75) and [detailed explanation](Khook/Readme.md)|
|Syscall Table Hooking (1)|Modify CR0 to remove write protect bit and change syscall table|[Puszek](Puszek/)|[In Puszek](Puszek/rootkit.c#L1081)|
|Syscall Table Hooking (2)|Make Syscall table writeable and then modify it|[Puszek](Puszek/)| [In Puszek](Puszek/rootkit.c#L133)|
|Syscall Table Hooking (3)|Hook the syscall functions by using the Function Hooking(1) Technique| [Reptile (uses Khook)](Reptile/)| [In Reptile](Reptile/kernel/main.c#L76) | 
|Hide Rootkit|Hook open syscall and modify the contents of the files (/proc/modules) which contain the name of the rooktit|[Puszek](Puszek/)| [In Puszek](Puszek/rootkit.c#L783) |
|Unable to rmmod module|Hook open syscall and make it not possible to open the rootkit module|[Puszek](Puszek/)|[In Puszek](Puszek/rootkit.c#L864)|
|Hide Files|Hook `new_sys_getdents` and `new_sys_getdents64` syscalls and modify it's result to hide files matching a specified prefix|[Puszek](Puszek/)|[In Puszek](Puszek/rootkit.c#L410)|
|Intercept Http Requests and Leak Data|Hook `send_to` syscall and the tcp data sent is checked for presence of headers. This is then searchd for presence of passwords etc|[Puszek](Puszek/)|[In Puszek](Puszek/rootkit.c#L535)|

## Test Environment

All rootkits are tested in a fresh Ubuntu 16.04 VM where `uname -a` returns `Linux r3x-VirtualBox 4.15.0-112-generic #113~16.04.1-Ubuntu SMP Fri Jul 10 04:37:08 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux`


## Disclaimer

The rootkits were written by their respective authors and the code belongs to them. This is just an attempt to preserve rootkits with a report on how they work and the OS they were tested on.

If your rootkit is present here and you wish it to be removed please contact me at `siddharth.muralee[AT]gmail[DOT]com`
