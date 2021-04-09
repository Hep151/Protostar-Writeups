# Stack0
## Briefing
This level introduces the concept that memory can be accessed outside of its allocated region, how the stack variables are laid out, and that modifying outside of the allocated memory can modify program execution.

This level is at /opt/protostar/bin/stack0
## Source Code
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```
## Solution
So we here that this program is vulnerable to a stack buffer-overflow attack, because it calls gets on a character array, that the user can control.

When we input more bytes of data, than the program can handle, the input will spill into nearby memory and overwrite variables, and other data on the stack.

So for this program if we enter the character 'A' 68 times, 64 of those characters will fill the 'buffer' array, and then the next 4 will spill into the 'modified' variable.

We can quickly print these characters with a quick command line script.

`python -c "print('A' * 68)" > input`

And then feed our input file into the program.

`./stack0 < input`

The program then outputs:

`you have changed the 'modified' variable`

Success! We have overwriten the variable to contain 4 'A' characters.
