# User Input
**We know something is vulnerable to buffer overflows when input can write past its memory**
- Look for calls like `char buf[64];` or `malloc(64);`
	- If the input function reads more than this size, an overflow can occur.
# malloc
- Allocated is rounded
- `requested + 0x10` -> We round this to the nearest multiple of 16.
	- The returned pointer from `malloc` is after the header
		- `[ chunk header (16 bytes) ] [ user data area (usable bytes) ]`
# printf
- `printf(input)`
	- https://owasp.org/www-community/attacks/Format_string_attack