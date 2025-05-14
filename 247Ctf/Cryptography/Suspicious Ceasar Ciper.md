
```python
#!/usr/bin/env python3

with open('suspicious_caesar_cipher.out', 'r') as f:
    lines = f.readlines()
e = int(lines[1].strip())
n = int(lines[2].strip())
encrypted_values = [val.replace('L', '') for val in lines[4].strip().replace('[', '').replace(']', '').split(', ')]
decryption_map = {}
for i in range(32, 127):
    char = chr(i)
    encrypted_val = pow(ord(char), e, n)
    decryption_map[str(encrypted_val)] = char
flag = ""
for val in encrypted_values:
    flag += decryption_map.get(val, "?")
print(flag)
```
