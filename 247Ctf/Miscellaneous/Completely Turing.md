
```python
from pwn import *
from string import ascii_uppercase
import re
interpreter = "bfi"  # or "beef" depending on what you have installed
program = "./completely_turing"
flag_pattern = re.compile(r'247CTF{[^}]+}')
for i in ascii_uppercase:
    for j in ascii_uppercase:
        p = process([interpreter, program])
        p.sendlineafter(b"index 0?", i.encode())
        p.sendlineafter(b"index 1?", j.encode())
        output = p.recvall().decode('latin-1')
        match = flag_pattern.search(output)
        if match:
            print(f"Found flag with keys {i} and {j}: {match.group(0)}")
            exit(0)
        if "247" in output:
            print(f"Found '247' with keys {i} and {j}: {output}")
            exit(0)
        p.close()
        if (ord(i) - ord('A')) % 5 == 0 and (ord(j) - ord('A')) % 5 == 0:
            print(f"Tested {i}{j}...")
print("No flag found.")
```
