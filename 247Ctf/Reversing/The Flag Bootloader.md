
```bash
file ./flag.com
```

```bash
qemu-system-x86_64 -drive format=raw,file=./flag.com
```

debugger

```bash
qemu-system-x86_64 -drive format=raw,file=./flag.com -s -S
```

```python
#!/usr/bin/env python3
import gdb
import re

class FlagSolver:
    def __init__(self):
        self.breakpoints = [
            0x7c7a, 0x7c8b, 0x7c9c, 0x7cad, 
            0x7cbe, 0x7ccf, 0x7ce0, 0x7cf1, 
            0x7d02, 0x7d11, 0x7d20, 0x7d2f, 
            0x7d3e, 0x7d4d, 0x7d5c, 0x7d6b
        ]

    def solve(self):
        try:
            gdb.execute("target remote localhost:1234")
            gdb.execute("break *0x7c00")
            gdb.execute("continue")
            for addr in self.breakpoints:
                gdb.execute(f"break *{hex(addr)}")
                gdb.execute("continue")
                gdb.execute("set $eflags |= (1 << 6)")
            flag_output = gdb.execute("x/s 0x7DAA", to_string=True)
            flag_match = re.search(r'"(247CTF\{.*?\})', flag_output)
            if flag_match:
                flag = flag_match.group(1).strip('"\n\r')
                print("\n=== FLAG FOUND ===")
                print(f"Flag: {flag}")
            else:
                print("Flag not found")
        except Exception as e:
            print(f"Error solving flag: {e}")

def main():
    solver = FlagSolver()
    solver.solve()

if __name__ == "__main__":
    main()
```

```bash
gdb -x poc.py
```

in the program put a psw

```bash
x/s 0x7DAA
hexdump 0x7DAA
```

`247CTF{******}`

psw also showed in the program output

or

```python
si = [0x77, 0x21, 0x67, 0x30, 0x60, 0x35, 0x0c, 0x0c, 0x78, 0x79, 0x2e, \
      0x2e, 0x20, 0x72, 0x70, 0x75, 0x29, 0x2b, 0x00, 0x5c, 0x21, 0x70, \
      0x62, 0x63, 0x65, 0x60, 0x07, 0x0d, 0x06, 0x02, 0x3b, 0x3b]

def xs(al,si,c):
    si[c] ^= al
    si[c+1] ^= al
    return c+2

c = 0
c = xs(0x4b^0x0c,si,c)
c = xs(0x53^0x06,si,c)
c = xs(0x58-0x01,si,c)
c = xs(0x62-0x29,si,c)
c = xs(0x68^0x23,si,c)
c = xs(0x4b^0x00,si,c)
c = xs(0x62-0x1e,si,c)
c = xs(0x4d-0x0b,si,c)
c = xs(0x45^0x0d,si,c)
c = xs(0x10^0x28,si,c)
c = xs(0x58^0x1d,si,c)
c = xs(0x7a^0x28,si,c)
c = xs(0x65-0x13,si,c)
c = xs(0x33^0x07,si,c)
c = xs(0x25^0x15,si,c)
c = xs(0x4c+0x0c,si,c)
flag = ''
for i in si:
    flag += chr(i)
print('247CTF{%s}' % flag)
```
