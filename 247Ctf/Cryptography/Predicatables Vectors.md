
```python
#!/usr/bin/python3
import requests
import binascii

url = "https://a972f6db904b2e0e.247ctf.com/"
retrieved_flag = ''
retrieved_flag_in_chr = ''
request_session = requests.Session()
possible_hex_digits = ['61','62','63','64','65','66','30','31','32','33','34','35','36','37','38','39']
test_chars = ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','R','S','T','U','V','W','X','Y','Z','a','b','c','d','e','f','g']

def get_IV(converted_text):
    result = request_session.request('GET', url + 'encrypt?plaintext=' + converted_text)
    return result.text[-32:]

def xor_operation(a, b):
    xored = []
    for i in range(len(a)):
        xored_value = ord(a[i%len(a)]) ^ ord(b[i%len(b)])
        hex_value = hex(xored_value)[2:]
        if len(hex_value) == 1:
            hex_value = "0" + hex_value
        xored.append(hex_value)
    return ''.join(xored)

def send_plaintext_after_XOR_1(possible_value, text, count):
    global retrieved_flag, retrieved_flag_in_chr
    converted_text = binascii.hexlify(bytes(text, 'utf-8') * count).decode('utf-8')
    known_text = text * count
    IV = get_IV(converted_text)
    print(f'[+] IV value: {IV}')
    IV_XOR1 = bytes.fromhex(IV).decode('latin-1')
    xored_value1 = xor_operation(IV_XOR1, known_text[:16])
    result1 = request_session.request('GET', url + 'encrypt?plaintext=' + xored_value1 + converted_text[32:])
    print(f'[+] First request: {url}encrypt?plaintext={xored_value1}{converted_text[32:]}')
    print(f'[+] Response: {result1.text}')
    IV_XOR2 = bytes.fromhex(result1.text[-32:]).decode('latin-1')
    xored_value2 = xor_operation(IV_XOR2, known_text[:16])
    result2 = request_session.request('GET', url + 'encrypt?plaintext=' + xored_value2 + converted_text[32:] + retrieved_flag + possible_value)
    print(f'[+] Second request: {url}encrypt?plaintext={xored_value2}{converted_text[32:]}{retrieved_flag}{possible_value}')
    print(f'[+] Response: {result2.text}')
    print(f'[+] Testing hex digit: {possible_value}')
    if result1.text[32:64] == result2.text[32:64]:
        print('[+] Match found!')
        retrieved_flag = retrieved_flag + possible_value
        ascii_string = binascii.unhexlify(possible_value).decode('ascii')
        retrieved_flag_in_chr = retrieved_flag_in_chr + ascii_string
        return True
    return False

def send_plaintext_after_XOR_2(possible_value, text, iteration_count):
    global retrieved_flag, retrieved_flag_in_chr
    count = 31 - int(len(retrieved_flag)/2)
    converted_text = binascii.hexlify(bytes(text, 'utf-8') * count).decode('utf-8')
    known_text = text * count
    IV = get_IV(converted_text)
    print(f'[+] IV value: {IV}')
    result1 = request_session.request('GET', url + 'encrypt?plaintext=' + converted_text)
    print(f'[+] First request: {url}encrypt?plaintext={converted_text}')
    print(f'[+] Response: {result1.text}')
    IV = bytes.fromhex(IV).decode('latin-1')
    IV_2 = bytes.fromhex(result1.text[-32:]).decode('latin-1')
    xored_value1 = bytes.fromhex(xor_operation(IV, IV_2)).decode('latin-1')
    xored_value2 = xor_operation(xored_value1, known_text + retrieved_flag_in_chr[:iteration_count])
    result2 = request_session.request('GET', url + 'encrypt?plaintext=' + xored_value2 + retrieved_flag[iteration_count*2:] + possible_value)
    print(f'[+] Second request: {url}encrypt?plaintext={xored_value2}{retrieved_flag[iteration_count*2:]}{possible_value}')
    print(f'[+] Response: {result2.text}')
    print(f'[+] Testing hex digit: {possible_value}')
    if result1.text[32:64] == result2.text[32:64]:
        print('[+] Match found!')
        ascii_string = binascii.unhexlify(possible_value).decode('ascii')
        retrieved_flag_in_chr = retrieved_flag_in_chr + ascii_string
        retrieved_flag = retrieved_flag + possible_value
        return True
    return False

def main():
    global retrieved_flag, retrieved_flag_in_chr
    print("[+] Starting BEAST attack on Predictable Vectors...")
    print("\n[+] Phase 1: Extracting first 16 bytes of flag...")
    for i in range(16):
        print(f'\n[+] Finding byte {i+1}/16...')
        count = 31 - int(len(retrieved_flag)/2)
        found = False
        for possible_value, text in zip(possible_hex_digits, test_chars):
            if send_plaintext_after_XOR_1(possible_value, text, count):
                print(f'[+] Found byte {i+1}: {possible_value}')
                print(f'[+] Flag so far (hex): {retrieved_flag}')
                print(f'[+] Flag so far (ascii): {retrieved_flag_in_chr}')
                found = True
                break
        if not found:
            print(f'[!] Failed to find byte {i+1}')
            break
    first_half = retrieved_flag
    print("\n[+] Phase 2: Extracting second 16 bytes of flag...")
    for i in range(16):
        print(f'\n[+] Finding byte {i+17}/32...')
        iteration_count = i+1
        found = False
        for possible_value, text in zip(possible_hex_digits, test_chars[:16]):
            if send_plaintext_after_XOR_2(possible_value, text, iteration_count):
                print(f'[+] Found byte {i+17}: {possible_value}')
                print(f'[+] Flag so far (hex): {retrieved_flag}')
                print(f'[+] Flag so far (ascii): {retrieved_flag_in_chr}')
                found = True
                break
        if not found:
            print(f'[!] Failed to find byte {i+17}')
            break
    print('\n[+] Retrieved flag in hex: ' + retrieved_flag)
    print('[+] Actual flag: 247CTF{' + retrieved_flag_in_chr + '}')

if __name__ == "__main__":
    main()
```
