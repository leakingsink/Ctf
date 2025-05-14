
```python
from pwn import *

conn = remote('1d0d6fcfb5ccf756.247ctf.com', 50058)
print(conn.recvline().decode().strip())
print(conn.recvline().decode().strip())
for i in range(500):
    question = conn.recvline().decode().strip()
    print(f"Problem {i+1}: {question}")
    if "What is the answer to " in question:
        expression = question.split("What is the answer to ")[1].split("?")[0]
        nums = expression.split(" + ")
        num1 = int(nums[0])
        num2 = int(nums[1])
        answer = num1 + num2
        print(f"Sending answer: {answer}")
        conn.send(str(answer).encode() + b'\r\n')
        if i < 499:
            try:
                response = conn.recvline(timeout=2).decode().strip()
                print(f"Response: {response}")
            except:
                print("No response received, continuing...")
    else:
        print(f"Unexpected question format: {question}")
        break
try:
    print("\nWaiting for flag...")
    flag = conn.recvall(timeout=5).decode().strip()
    print(f"Flag: {flag}")
except:
    print("\nTimeout waiting for flag. Try checking the connection output.")

conn.close()
```
