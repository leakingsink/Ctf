
```bash
openssl x509 -inform DER -in public-cert-key.der -pubkey -noout > key.pub
```

```bash
cat key.pub
```

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDVzrM58EOtSsBE5WgLJiNjPa/h
ZtArVRS0PjS01u6DyAlvAWwmSEbhQNiwC74V9TANRDCdKShcsf58Ij0BGeE0ybsp
2sqbDRUkuJ5siVCNh6OdhMnHLySTcU+3jKWsPNNz8U2BaETEVafB9yggAgjWqEbl
xXq0q3uc464SDnWZawIDAQAB
-----END PUBLIC KEY-----
```

Use RsaCtfTool to generate private key and save it as private.key:

```bash
rsactftool --publickey key.pub --private --attack factordb > private.key
```

Use tshark to decrypt data:

```bash
tshark -r multiplication_tables.pcap -o "tls.keys_list: 192.168.10.111,8443,http,private.key" -z "follow,ssl,ascii,1"
```
