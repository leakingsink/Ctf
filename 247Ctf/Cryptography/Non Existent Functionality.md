
```python
import requests
from base64 import b64encode
import concurrent.futures
import threading

TARGET_URL = "https://3cf0688a000c7906.247ctf.com/get_flag"
MAX_WORKERS = 80
print_lock = threading.Lock()

def try_byte(j, pos, data):
    try:
        test_data = bytearray(data)
        test_data[pos] = j
        encoded = b64encode(test_data).decode()
        r = requests.get(f"{TARGET_URL}?password={encoded}", timeout=3)
        if "Invalid password!" in r.text:
            with print_lock:
                print(f"Found: {hex(j)[2:]} at {pos}")
            return j
    except:
        pass
    return None

def find_prexored():
    all_prexored = []
    data = bytearray(48)
    for block in range(2):
        print(f"Block {block+1}")
        block_prexored = [] 
        for i in range(16):
            padding_val = i + 1
            print(f"Position {i}...", end="\r")
            if block_prexored:
                for j, prexor in enumerate(block_prexored):
                    data[j] = prexor ^ padding_val
            if block == 1 and all_prexored:
                target_text = b"t_admin_password"
                for k, prexor in enumerate(all_prexored[0]):
                    if k < len(target_text):
                        data[16 + k] = prexor ^ target_text[k]
            with concurrent.futures.ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
                futures = {executor.submit(try_byte, j, i, data): j for j in range(256)}
                for future in concurrent.futures.as_completed(futures):
                    result = future.result()
                    if result is not None:
                        for f in futures:
                            if not f.done():
                                f.cancel()
                        prexor_byte = result ^ padding_val
                        block_prexored.append(prexor_byte)
                        print(f"Position {i}: {hex(result)[2:]} -> {hex(prexor_byte)[2:]}")
                        break
        all_prexored.append(block_prexored)
        print(f"Block {block+1} values: {[hex(x)[2:] for x in block_prexored]}")
    return all_prexored

def craft_payload(prexors):
    target1 = bytearray([11] * 11) + b"secre"
    target2 = b"t_admin_password"
    iv = bytearray(x ^ y for x, y in zip(prexors[0], target1))
    middle = bytearray(prexors[1][i] ^ target2[i] for i in range(len(target2)))
    while len(middle) < 16:
        middle.append(prexors[1][len(middle)])
    return b64encode(iv + middle + bytearray(16)).decode()

def main():
    print("Starting attack...")
    prexors = find_prexored()
    if len(prexors) == 2:
        payload = craft_payload(prexors)
        print(f"Payload: {payload}")
        r = requests.get(f"{TARGET_URL}?password={payload}")
        print(f"Result: {r.text}")

if __name__ == "__main__":
    main()
```

```python
from base64 import b64encode

aes_raw = [0xd3, 0xe7, 0x44, 0xbe, 0xbb, 0xc3, 0x95, 0xa9, 0x03, 0xde, 0x61, 0xc2, 0x2b, 0x1c, 0x07, 0x83]
a = bytearray([0x0b] * 11 + [ord(c) for c in "secre"])
fin = bytearray()
for i in range(len(a)):
    fin.append(a[i] ^ aes_raw[i])
second_block = bytearray([0xa8, 0x69, 0x90, 0x23, 0xcd, 0x77, 0x33, 0xce, 0x66, 0x52, 0xeb, 0x69, 0xb7, 0x59, 0x3d, 0x30])
third_block = bytearray(16)
final_data = fin + second_block + third_block
encoded = b64encode(final_data)
print(encoded.decode())
```

2OxPtbDInqII1WqxTn915qhpkCPNdzPOZlLrabdZPTAAAAAAAAAAAAAAAAAAAAAA

<https://3cf0688a000c7906.247ctf.com/get_flag?password=2OxPtbDInqII1WqxTn915qhpkCPNdzPOZlLrabdZPTAAAAAAAAAAAAAAAAAAAAAA>
