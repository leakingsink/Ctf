
```python
import requests

url = "https://2950e5384e7e7d6b.247ctf.com"
padding = "0" * 32
target_hex = "696d706f737369626c655f666c61675f75736572"  # "impossible_flag_user"
response = requests.get(f"{url}/encrypt?user={padding}{target_hex}")
ciphertext = response.text
target_ciphertext = ciphertext[32:]
flag_response = requests.get(f"{url}/get_flag?user={target_ciphertext}")
print(flag_response.text)
```
