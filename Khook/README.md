# Khook Hooking engine

Linux kernel hooking engine allows you to hook into any function in the kernel space and then run a function of your choosing whenever the hooked function is called.

## Implementation details

### Getting things ready

For each hook that was defined by the user, khook tries to find the address of the function

the relevant code is - 

```c
static void khook_resolve(void)
{
	khook_t *p;
	KHOOK_FOREACH_HOOK(p) {
		p->target.addr = (void *)khook_lookup_name(p->target.name);
		if (!p->target.addr) printk("khook: failed to lookup %s symbol\n", p->target.name);
	}
}
```

Here, what `khook_lookup_name` does is it searches kallsyms for the function and returns the address. (if kprobes are enabled that can also be used)

Note: Here there is no direct syscall table overwrite, we just overwrite the function that implements the syscall.

Once all the addresses are resolved khook engine starts the process of rewriting the functions. Now, since it plans to add assembly instructions the process is architecture specific.

```c
static int khook_sm_init_hooks(void *arg)
{
	khook_t *p;
	KHOOK_FOREACH_HOOK(p) {
		if (!p->target.addr) continue;
		khook_arch_sm_init_one(p);
	}
	return 0;
}
```

### The hooking part

The [hooking function](/Khook/khook/x86/hook.c#L75) 
- Given an address khook initially checks if there is already a jump at that address
- Khook uses an stub to do the hooking process. it overwrites the address of a function to jump to the stub of a function. (Amazing well implemented macros to create stubs per function).	- Each Stub contains a pointer to the original function and also a counter which counts the number of uses.
  - When a function is called the use count is incremented and similarly decremented once the call ends. (it's similar to a semaphore)
  - The code for the stub can be found [here](khook/x86/stub.S)
  - The stub contains a place holder `0xcacacaca` which is then replaced with the address that we need.
  - The In-kernel Dissambler is used for finding length of instruction. (unsure of how this works!)
- In the stub, khook places a jump to the original function. (This is in writable memory)
- Now, we need to replace the original function with a jump to the stub, for this we replace the write bit on the CRO register.
- The X bytes overwritten are stored and reused, so that we don't loose any bytes.

```
CALLER
| ...
| CALL X -(1)---> X
| ...  <----.     | JUMP -(2)----> STUB.hook
` RET       |     | ???            | INCR use_count
            |     | ...  <----.    | CALL handler -(3)------> HOOK.fn
            |     | ...       |    | DECR use_count <----.    | ...
            |     ` RET -.    |    ` RET -.              |    | CALL origin -(4)-----> STUB.orig
            |            |    |           |              |    | ...  <----.            | N bytes of X
            |            |    |           |              |    ` RET -.    |            ` JMP X + N -.
            `------------|----|-------(8)-'              '-------(7)-'    |                         |
                         |    `-------------------------------------------|---------------------(5)-'
                         `-(6)--------------------------------------------'
```
^ diagram shamelessly copied from the khook README

### Misc stuff

Apparently module_alloc memory isn't executable by default and has to be made executable.
