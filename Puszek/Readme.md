# Puszek Rootkit

## Table of contents 
- [Puszek Rootkit](#puszek-rootkit)
  - [Table of contents](#table-of-contents)
  - [Features](#features)
  - [Digging into the features](#digging-into-the-features)
    - [Syscall Table hooking](#syscall-table-hooking)
    - [Hide module](#hide-module)
    - [Unable to rmmod](#unable-to-rmmod)
    - [Hiding Files and Processes](#hiding-files-and-processes)
    - [Inode modification trivia](#inode-modification-trivia)

## Features

    - Hide files that ends on configured suffix (".rootkit" by default).
    - Hide processes that cmdline contains defined text (COMMAND_CONTAINS - ".//./" by default).
    -  Intercept GET and POST HTTP requests and log to `/etc/http_requests.rootkit`.
        - When password is found in HTTP request it's additionally logged to `/etc/passwords.rootkit`
    - Rootkit module is invisible in lsmod output, file /proc/modules, and directory /sys/module/.
    - It isn't possible to unload rootkit by rmmod command.
    - Netstat and similar tools won't see TCP connections of hidden processes.

## Digging into the features
  

### Syscall Table hooking
        - Get address of syscall table
            - Syscall table address no longer available in the kallsysms 
                - Search memory for the pointer table! (maybe another chokepoint!)
                - https://bbs.archlinux.org/viewtopic.php?id=139406
        - Write to CR0 - Replace the write protect bit in CR0, hook functions
            - ```c
#if SYSCALL_MODIFY_METHOD == CR0
    original_cr0 = read_cr0();
    write_cr0(original_cr0 & ~0x00010000);```
        - Make the table writeable -  using the address
            - ```c
#if SYSCALL_MODIFY_METHOD == PAGE_RW
    make_rw((long unsigned int)sys_call_table_);
#endif

... hook here...

#if SYSCALL_MODIFY_METHOD == PAGE_RW
    make_ro((long unsigned int)sys_call_table_);
#endif

int make_rw(unsigned long address)
{
    unsigned int level;
    pte_t *pte = lookup_address(address, &level);
    pte->pte |= _PAGE_RW;
    return 0;
}
 
//set a page read only
int make_ro(unsigned long address)
{ 
    unsigned int level;
    pte_t *pte = lookup_address(address, &level);
    pte->pte = pte->pte & ~_PAGE_RW;
    return 0;
}```

### Hide module
        - `/proc/modules` - 
            - Read from the old one, replace rootkit and return. Done by hooking open syscall [fake file at `/etc/modules.rootkits`]
            - ```c
if (strcmp(filename, "/proc/modules") == 0)
{
	   
    new_path = kzalloc(strlen("/etc/modules") + strlen(FILE_SUFFIX) + 1,
    	    GFP_KERNEL);
	// open new path
  	// read the realmodule file
    // replace the rootkit after reading
    // write fake to the old file
}```
        - `/proc/net/tcp` - 
            - Same as above with the temp file being - `/etc/net.rootkits`
            - Parse each line, check if inode belongs to the process if yes then remove
        - These make it hard for other commands written on top of `/proc/modules` hard to detect the module. 
### Unable to rmmod
        - It seems that allowing the userspace process not to open the rootkit blocks it from removing the module
            - ```c
#if UNABLE_TO_UNLOAD
    }
    else if (strncmp(filename, rootkit_path, rootkit_path_len) == 0)
    {
    	ret = -ENOENT;
#endif```

### Hiding Files and Processes
        - Using modifications to inodes
        - Hiding files - 
            - hooking `new_sys_getdents` and `new_sys_getdents64` - checks if the file with prefix is present in the dirent. if found the entry is deleted.
    - **Storing  the HTTP requests**
        - syscall - `send_to` syscall hooked and the tcp data is checked for presence of headers.

### Inode modification trivia
    - It maintains a list of hidden inodes for the processes
    - The list is then checked during each iteration
    - Inodes of processes are collected by opening `/proc/<pid>/fd` directory and then reading the links to get the inodes. (it ignores fd's 0-2 maybe because they are the standard ones)
