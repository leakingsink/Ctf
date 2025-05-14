
<https://dogbolt.org/?id=6e1118f0-bdca-4796-9c2b-2856777ea421#Hex-Rays=171>

```python
s = "875e9409f9811ba8560beee6fb0c77d2"
qwords = [
    0x5A53010106040309,
    0x5C585354500A5B00,
    0x555157570108520D,
    0x5707530453040752
]
s2 = []
for qword in qwords:
    for i in range(8):
        s2.append((qword >> (i * 8)) & 0xFF)
result = ''
for i in range(min(len(s), len(s2))):
    xor_result = ord(s[i]) ^ s2[i]
    result += chr(xor_result)
print(f"Password: {result}")
print(f"Flag: 247CTF{{{result}}}")
```
