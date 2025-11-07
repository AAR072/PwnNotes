# Template
```python
from pwn import *

#                 'BINARY'
# target = process('./')
#                'IP', PORT
# target = remote('', 1337)

```
# Setup
## Setting Target
- Process objects can be created from a local binary, or created from a remote socket
	- `target = process('./target')`
	- `target = remote('127.0.0.1', 1337)`
- We can also pass arguments or env variables.
	- `target = process(['./target', '--arg1', 'some data'], env={'env1': 'some data'})`
## GDB Interaction
- We can attach gdb to something that is already running.
	- `gdb.attach(target)`
- We can also start under GDB, disable `ASLR`, send a script, etc.
	- `target = gdb.debug('./target', aslr=False, gdbscript='b *main+123')`
# Data / IO
## Reading Data
- We can read data from `stdout`.
	- To get useful output
		- `target.recv(timeout=1).decode(errors='ignore')`
	- To read everything:
		- `target.readall()`
	- To read until a `\n`:
		- `target.readline()`
	- To read 123 bytes:
		- `target.read(123)`
	- To read until a string, ex `Nice`:
		- `target.readuntil('Nice')`
## Sending Data
- We can send data to `stdin`.
	- `target.write(b'Hello World!')`
	- Some programs expect a `\n`. In this case we can use `writeline`.
		- `target.writeline(b'Hello World!')`