

```python
#!/usr/bin/env python3

# https://fireshellsecurity.team/ritsec-pcap-me-if-you-can/


# Open and read the file containing all the packets' data field
with open('streams/24') as f:
    data = f.read().strip().split("\n")
    f.close()

for communication in range(0, len(data), 1):
    line = data[communication][8:]

    # Group the data on a 2-char basis and split it when equal to "00"
    request = []
    str = ""
    for i in range(0, len(line), 2):
        if(line[i:i+2] == "00"):
            request.append(str)
            str = ""
        else:
            str += line[i:i+2]

    for part in range(0, len(request)):

        # "Bytefy" the part being parsed
        for i in range(0, len(request[part]), 2):

            byte = int(request[part][i:i+2], 16)

            if byte >= 126:
                byte -= 126

            # Print the byte's equivalent ASCII char
            print(chr(byte), end="")

        print(" | ", end="")


    print("")
```

```python
#!/usr/bin/env python3

URL="6774def642fb433a.247ctf.com"
PORT=50035

from pwn import *

import binascii
import codecs


# Connect to the server
conn = remote(URL,PORT)

# remove \r\n from sessionID as we will add bytes to the packet
sessionID = conn.recvline()[:-2]

# define separator, counter, command and end
sep = b'00' # separator
counter = b'31'
command = b'34'
end = b'\r\n'

# Request format:
# sessionID 00 counter 00 command 00 redundancy \r\n

# calculate CRC redundancy
redundancy = binascii.crc32(binascii.a2b_hex(sessionID+sep+counter+sep+command))
red = b''
for i in str(redundancy):
    red += codecs.encode(str.encode(i), 'hex')

# create payload
payload = sessionID+sep+counter+sep+command+sep+red+end

conn.send(payload)
#print (payload)
response = conn.recvline()
#print (response)
print (unhex(response))

conn.close()



# 496e76616c69642073656374696f6e20636f756e7421
# Invalid section count!

# 496e76616c69642073657373696f6e206c656e67746821
# Invalid session length!

# 496e76616c696420726564756e64616e637920636865636b21
# Invalid redundancy check!

# 496e76616c6964206d65737361676520636f756e74657221
# Invalid message counter!
```
