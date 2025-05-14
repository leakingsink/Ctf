
```python
#!/usr/bin/env python3

from itertools import cycle

def xor_bytes(data, key):
    return bytes(d ^ k for d, k in zip(data, cycle(key)))

def main():
    with open('exclusive_key', 'rb') as f:
        encrypted_data = f.read()
    known_plaintext = b"247CTF{"
    encrypted_start = encrypted_data[:len(known_plaintext)]
    potential_key_bytes = xor_bytes(encrypted_start, known_plaintext)
    potential_key = potential_key_bytes.decode('utf-8', errors='replace')
    print(f"Potential key start (from known plaintext): {potential_key}")
    wikipedia_key = b'<!DOCTYPE html>\n<html class="client-nojs'
    decrypted = xor_bytes(encrypted_data, wikipedia_key)
    with open('decrypted_flag.txt', 'wb') as f:
        f.write(decrypted)
    decrypted_text = decrypted.decode('utf-8', errors='replace')
    flag_index = decrypted_text.find("247CTF{")
    if flag_index != -1:
        end_index = decrypted_text.find("}", flag_index)
        if end_index != -1:
            flag = decrypted_text[flag_index:end_index+1]
            print(f"Found flag: {flag}")
        else:
            print(f"Flag start found at position {flag_index} but no closing brace found")
            print(f"Partial flag: {decrypted_text[flag_index:flag_index+50]}...")
    else:
        print("Flag not found in decrypted content. Showing first 100 bytes:")
        print(decrypted_text[:100])
    if flag_index != -1:
        flag_bytes = decrypted_text[flag_index:flag_index+len("247CTF{")].encode('utf-8')
        derived_key = xor_bytes(encrypted_data[flag_index:flag_index+len("247CTF{")], flag_bytes)
        print(f"Derived key from flag position: {derived_key.decode('utf-8', errors='replace')}")

if __name__ == "__main__":
    main()
```
