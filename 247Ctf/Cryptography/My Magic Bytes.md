
```bash
xxd my_magic_bytes.jpg.enc | head -n 1
```

```
00000000: b914 0645 71e0 b5f7 3707 cb85 47cc f9a4  ...Eq...7...G...
```

```python
#!/usr/bin/env python3
import sys,binascii,itertools,os
def xor(d,k):return bytes(a^b for a,b in zip(d,itertools.cycle(k)))
if len(sys.argv)<3:print(f"Usage: {sys.argv[0]} data key [-o output]");sys.exit(1)
try:
    d=open(sys.argv[1],"rb").read() if os.path.isfile(sys.argv[1]) else binascii.unhexlify(sys.argv[1].replace(" ",""))
    k=binascii.unhexlify(sys.argv[2].replace(" ",""))
    r=xor(d,k)
    if len(sys.argv)>3 and sys.argv[3]=="-o" and len(sys.argv)>4:
        with open(sys.argv[4],"wb") as f:f.write(r)
        print(f"Saved to {sys.argv[4]}")
    else:print(r.hex())
except Exception as e:print(f"Error: {e}");sys.exit(1)
```

```bash
xor FFD8FFDB b9140645
```

46ccf99e

```bash
xor FFD8FFEE b9140645
```

46ccf9ab

```bash
xor FFD8FFE000104A4649460001 b914064571e0b5f73707cb85
```

46ccf9a571f0ffb17e41cb84

```bash
xor my_magic_bytes.jpg.enc 46ccf99e | xxd -r -p | file -
xor my_magic_bytes.jpg.enc 46ccf9ab | xxd -r -p | file -
```

JPEG image data
JPEG image data

```bash
xor my_magic_bytes.jpg.enc 46ccf9a571f0ffb17e41cb84 | xxd -r -p | file -
```

JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, progressive, precision 8, 500x500, components 3

```bash
xor my_magic_bytes.jpg.enc 46ccf9a571f0ffb17e41cb84 | xxd -r -p > flag.jpg
```

or

```python
from itertools import cycle

with open("my_magic_bytes.jpg.enc", "rb") as f:
    encrypted_data = f.read()
key = bytes.fromhex("46ccf9a571f0ffb17e41cb84")
def xor_decrypt(data, key):
    return bytes(a ^ b for a, b in zip(data, cycle(key)))
decrypted_data = xor_decrypt(encrypted_data, key)
with open("flag.jpg", "wb") as f:
    f.write(decrypted_data)
print("Decryption completed! Check flag.jpg for the flag.")
```

```bash
feh flag.jpg
```
