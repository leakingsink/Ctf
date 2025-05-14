
```bash
mergecap -w combined.pcap chall-i1.pcap chall-i2.pcap chall-i3.pcap
tshark -r combined.pcap -T json > CombinedFilter
```

```python
import json

with open("CombinedFilter", "r") as pcap:
    JSONdata = pcap.read()
data = json.loads(JSONdata)

print(type(data))  # Should output <class 'list'>

payloadList = []
seqNoSet = set()

for element in data:
    tcp_layer = element['_source']['layers']['tcp']
    
    # Check if the nested keys exist
    if ('tcp.options_tree' in tcp_layer and 
        'mptcp' in tcp_layer['tcp.options_tree'] and 
        'tcp.options.mptcp.rawdataseqno' in tcp_layer['tcp.options_tree']['mptcp']):
        
        seqNo = int(tcp_layer['tcp.options_tree']['mptcp']['tcp.options.mptcp.rawdataseqno'])
        
        if 'tcp.payload' in tcp_layer and seqNo not in seqNoSet:
            payLoad = bytes.fromhex(tcp_layer['tcp.payload'].replace(':', ''))
            payloadList.append((seqNo, payLoad))
            seqNoSet.add(seqNo)
    else:
        # If the key is missing, skip this packet.
        continue

payloadList.sort()

with open("alldata", "wb") as writeout:
    for seq, payLoad in payloadList:
        writeout.write(payLoad)

with open("alldata", "rb") as filehandle:
    with open("truncated", "wb") as writebinary:
        writebinary.write(filehandle.read()[0xf4:])
```

```bash
binwalk -Mea alldata
```

