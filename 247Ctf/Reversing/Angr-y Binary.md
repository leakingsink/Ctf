
```python
#!/usr/bin/env python3
import angr
import sys

project = angr.Project("./angr-y_binary")
state = project.factory.entry_state()
simmgr = project.factory.simulation_manager(state)
find = 0xFFFFAAAC
avoid = 0xFFFFAAAB
simmgr.explore(find=find,avoid=avoid)
if simmgr.found[0]:
    print("found a solution")
    print(simmgr.found[0].posix.dumps(sys.stdin.fileno()))
else:
    print("No found solutions")
```

wgIdWOS6Df9sCzAfiK\xd0\x01

nc and send psw to get the flag
