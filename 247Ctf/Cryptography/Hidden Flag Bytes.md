
```python
#!/usr/bin/env python3

import requests
import binascii
import time

URL = 'https://77fd91873389de9f.247ctf.com'
allchar = "0123456789abcdefCTF{}_-"
block_size = 16
solution = '3234374354467b'
flag_so_far = "247CTF{"
print("Starting to extract the flag from " + URL + "...")
for c in range(33):
    pad = 'AA' * (3 * block_size - 1 - len(solution) // 2)
    found = False
    for i in allchar:
        try:
            x = binascii.hexlify(bytes(i, 'utf-8'))
            guess = str(x, 'ascii')
            r1 = requests.get(URL + '/encrypt?plaintext=' + pad)
            a = r1.text[:96]
            r2 = requests.get(URL + '/encrypt?plaintext=' + pad + solution + guess)
            b = r2.text[:96]
            if a == b:
                flag_so_far += i
                solution += guess
                found = True
                print(f"Found '{i}' | Current flag: {flag_so_far}")
                break   
        except Exception as e:
            print(f"Error occurred: {e}")
            time.sleep(2)
    if not found:
        print("No character found at this position. Stopping.")
        break
    if flag_so_far.endswith('}'):
        print(f"Complete flag found: {flag_so_far}")
        break
    time.sleep(0.2)
print(f"Final result: {flag_so_far}")
```
