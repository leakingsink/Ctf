
```python
#!/usr/bin/env python3

rounds = 5
block_size = 8

inv_sa = {
    15: 0, 
    2: 1, 
    14: 2, 
    0: 3, 
    1: 4, 
    3: 5, 
    10: 6, 
    6: 7, 
    4: 8, 
    11: 9, 
    9: 10, 
    7: 11, 
    13: 12, 
    12: 13, 
    8: 14, 
    5: 15
}

inv_sb = {
    12: 0, 
    8: 1, 
    13: 2, 
    6: 3, 
    9: 4, 
    1: 5, 
    11: 6, 
    14: 7, 
    5: 8, 
    10: 9, 
    3: 10, 
    4: 11, 
    0: 12, 
    15: 13, 
    7: 14, 
    2: 15
}

to_bin = lambda x, n=block_size: format(x, "b").zfill(n)
to_int = lambda x: int(x, 2)
to_chr = lambda x: "".join([chr(i) for i in x])
to_ord = lambda x: [ord(i) for i in x]
bin_join = lambda x, n=int(block_size / 2): (str(x[0]).zfill(n) + str(x[1]).zfill(n))
bin_split = lambda x: (x[0 : int(block_size / 2)], x[int(block_size / 2) :])
str_split = lambda x: [x[i : i + block_size] for i in range(0, len(x), block_size)]
xor = lambda x, y: x ^ y

def inv_s(a, b):
    return inv_sa[a], inv_sb[b]

def inv_p(a):
    return a[5] + a[3] + a[1] + a[2] + a[7] + a[0] + a[4] + a[6]

def ks(k):
    return [
        k[i : i + int(block_size)] + k[0 : (i + block_size) - len(k)]
        for i in range(rounds)
    ]

def kx(state, k):
    return [xor(state[i], k[i]) for i in range(len(state))]

def de_kx(val, key):
    return [xor(val, key)]

def de(state, key):
    decrypted = []
    count = 0
    for i in state:
        pe = inv_p(to_bin(i))
        a, b = bin_split(pe)
        sa, sb = inv_s(to_int(a), to_int(b))
        step1 = bin_join((to_bin(sa, int(block_size / 2)), to_bin(sb, int(block_size / 2))))
        step2 = to_int(step1)
        if count % 2 == 0:
            re = de_kx(step2, key[0])
        else:
            re = de_kx(step2, key[1])
        count += 1
        decrypted.append(re[0])
    return decrypted

def r(en_flags, key):
    keys = ks(key)
    state = []
    for i in range(0, len(en_flags), block_size):
        state.append(en_flags[i:i+block_size])
    for b in range(len(state)):
        for i in range(rounds):
            state[b] = de(state[b], keys[i])
    return [chr(e) for es in state for e in es]

en_flags = [190, 245, 36, 15, 132, 103, 116, 14, 59, 38, 28, 203, 158, 245, 222, 157, 36, 100, 240, 206, 36, 205, 51, 206, 90, 212, 222, 245, 83, 14, 222, 206, 163, 38, 59, 157, 83, 203, 28, 27]
print("Attempting to brute force the key...")
count = 0
for i in range(256):
    count += 1
    print(f"Trying key component 1: {i}/255 ({count/256*100:.1f}%)")
    
    for j in range(256):
        key = [i, j] * 4
        try:
            decrypted = r(en_flags, key)
            decrypted_string = ''.join(decrypted)
            if '247CTF' in decrypted_string:
                print('\nFound valid flag!')
                print(f'Decrypted flag: {decrypted_string}')
                print(f'Key: [{i}, {j}]')
                exit(0)
        except Exception as e:
            continue

print("No valid flag found.")
```
