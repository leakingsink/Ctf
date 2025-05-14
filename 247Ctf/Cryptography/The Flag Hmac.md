
```python
#!/usr/bin/env python3
import hashpumpy
import requests
import urllib.parse

URL = "https://4c733c2dad257c2a.247ctf.com/"
KNOWN_HASH = "941f351a0c83589622bb5b81cddb18f4a74a7e877cd9b9548e37fec58370fc3e"
ORIGINAL_DATA = "742"  # strrev(247)
SECRET_LENGTH = 40  # Typical length for 247CTF{...} flags
APPEND_DATA = "x"  # Append anything other than 247 reversed

def perform_attack():
    print(f"[*] Starting length extension attack")
    print(f"[*] Original hash: {KNOWN_HASH}")
    print(f"[*] Original data: {ORIGINAL_DATA}")
    print(f"[*] Estimated secret length: {SECRET_LENGTH}")
    print(f"[*] Data to append: {APPEND_DATA}")
    new_hash, extended_data = hashpumpy.hashpump(KNOWN_HASH, ORIGINAL_DATA, APPEND_DATA, SECRET_LENGTH)
    print(f"[+] New hash: {new_hash}")
    print(f"[+] Extended data (hex): {extended_data.hex()}")
    reversed_extended_data = extended_data[::-1]
    encoded_data = urllib.parse.quote(reversed_extended_data)
    print(f"[+] Reversed & URL-encoded data: {encoded_data}")
    attack_url = f"{URL}?user={encoded_data}&hmac={new_hash}"
    print(f"[*] Sending request to: {attack_url}")
    response = requests.get(attack_url)
    if "247CTF{" in response.text:
        flag = response.text.split("<?php")[0].strip()
        print(f"[+] Success! Flag: {flag}")
    else:
        print(f"[-] Attack failed. Response:")
        print(response.text[:200] + "..." if len(response.text) > 200 else response.text)

if __name__ == "__main__":
    perform_attack()
```
