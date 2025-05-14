
```python
import hashlib

salt = "f789bbc328a3d1a3"
i = 100000000
while 1:
    password = salt + str(i)
    password = password.encode('utf8')
    new_hash = hashlib.md5(password).hexdigest()
    if new_hash[0:2] == "0e" and new_hash[2:32].isdigit():
        print(password, str(i), new_hash)
        exit(0)
    i += 1
```

b'f789bbc328a3d1a3237701818' 237701818 0e66827140348492259952792953401

<https://98a966bc8786d983.247ctf.com/?password=237701818>
