
```python
#!/usr/bin/env python3
import socket
import time
import random
import re

HOST = "3ade9d1c80194fe3.247ctf.com"
PORT = 50278

def get_winning_number():
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    s.recv(1024)
    s.send(b"0\n")
    response = s.recv(1024).decode()
    s.close()
    match = re.search(r"winning number was ([\d\.]+)", response)
    if match:
        return float(match.group(1))
    return None

def find_matching_seed(winning_number):
    current_time = int(time.time())
    for offset in range(-100, 101):
        seed_to_try = current_time + offset
        secret = random.Random()
        secret.seed(seed_to_try)
        generated = secret.random()
        winning_str = "{:.12f}".format(winning_number)
        generated_str = "{:.12f}".format(generated)
        if winning_str == generated_str:
            print(f"Found matching seed: {seed_to_try} (offset {offset})")
            return seed_to_try
    print("Could not find a matching seed")
    return None

def predict_next_number(seed):
    next_seed = seed + 1
    secret = random.Random()
    secret.seed(next_seed)
    return secret.random()

def win_lottery(predicted_number):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    prompt = s.recv(1024).decode().strip()
    print(f"Server: {prompt}")
    guess = "{:.12f}".format(predicted_number)
    print(f"Sending guess: {guess}")
    s.send((guess + "\n").encode())
    response = s.recv(1024).decode().strip()
    print(f"Server: {response}")
    s.close()
    return "Congratulations" in response

def exploit():
    winning_number = get_winning_number()
    if winning_number is None:
        print("Failed to get the winning number")
        return
    print(f"Current winning number: {winning_number}")
    seed = find_matching_seed(winning_number)
    if seed is None:
        return
    next_number = predict_next_number(seed)
    print(f"Predicted next winning number: {next_number}")
    success = win_lottery(next_number)
    if not success:
        print("Failed to win. Let's try with exact string matching...")
        winning_number = get_winning_number()
        win_lottery(winning_number)

if __name__ == "__main__":
    exploit()
```
