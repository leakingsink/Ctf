
```bash
bruteforce-salted-openssl -f /usr/share/dict/rockyou.txt -d sha256 encrypted_flag.enc
```

Password candidate: (algorithm)crypto

```bash
openssl aes-256-cbc -d -in encrypted_flag.enc -out decrypted.txt -k "(algorithm)crypto"
```

```bash
cat decrypted.txt
```

