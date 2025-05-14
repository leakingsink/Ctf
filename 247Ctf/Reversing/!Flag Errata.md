
```
objdump -d flag_errata.exe

winedbg --gdb flag_errata.exe

WINEDEBUG=trace+all wine flag_errata.exe

frida -f flag_errata.exe -l h.js


❯ (echo -en 'cac\n')|WINEDEBUG=+relay wine flag_errata.exe 2>&1 | grep -i GetLastError
❯ WINEDEBUG=+relay wine flag_errata.exe 2>&1 | grep -i GetLastError

0138:Call KERNEL32.GetLastError() ret=00403094
0138:Ret  KERNEL32.GetLastError() retval=00000002 ret=00403094
0138:Call KERNEL32.GetLastError() ret=0040309d
0138:Ret  KERNEL32.GetLastError() retval=00000003 ret=0040309d
0138:Call KERNEL32.GetLastError() ret=004030a6
0138:Ret  KERNEL32.GetLastError() retval=00000020 ret=004030a6
0138:Call KERNEL32.GetLastError() ret=004030af
0138:Ret  KERNEL32.GetLastError() retval=00000006 ret=004030af
0138:Call KERNEL32.GetLastError() ret=004030b8
0138:Ret  KERNEL32.GetLastError() retval=00000012 ret=004030b8
0138:Call KERNEL32.GetLastError() ret=004030c1
0138:Ret  KERNEL32.GetLastError() retval=00000018 ret=004030c1
0138:Call KERNEL32.GetLastError() ret=004030ca
0138:Ret  KERNEL32.GetLastError() retval=00000020 ret=004030ca
0138:Call KERNEL32.GetLastError() ret=004030d3
0138:Ret  KERNEL32.GetLastError() retval=00000057 ret=004030d3
0138:Call KERNEL32.GetLastError() ret=004030dc
0138:Ret  KERNEL32.GetLastError() retval=00000057 ret=004030dc
0138:Call KERNEL32.GetLastError() ret=004030e5
0138:Ret  KERNEL32.GetLastError() retval=00000002 ret=004030e5
0138:Call KERNEL32.GetLastError() ret=004030ee
0138:Ret  KERNEL32.GetLastError() retval=0000007e ret=004030ee
0138:Call KERNEL32.GetLastError() ret=004030f7
0138:Ret  KERNEL32.GetLastError() retval=000000b7 ret=004030f7
0138:Call KERNEL32.GetLastError() ret=00403100
0138:Ret  KERNEL32.GetLastError() retval=000000c1 ret=00403100
0138:Call KERNEL32.GetLastError() ret=00403109
0138:Ret  KERNEL32.GetLastError() retval=00000002 ret=00403109
0138:Call KERNEL32.GetLastError() ret=00403112
0138:Ret  KERNEL32.GetLastError() retval=00000003 ret=00403112
0138:Call KERNEL32.GetLastError() ret=0040311b
0138:Ret  KERNEL32.GetLastError() retval=00000020 ret=0040311b

```

chmod +x 


1 extra/qemu-user-binfmt 9.2.3-1 (16.4 KiB 56.1 KiB) 
    Binary format rules for QEMU user mode emulation

❯ sudo mkdir /etc/qemu-binfmt
❯ sudo ln -s /usr/mipsel-linux-gnu /etc/qemu-binfmt/mipsel
❯ sudo ln -s /usr/arm-linux-gnueabihf /etc/qemu-binfmt/arm
