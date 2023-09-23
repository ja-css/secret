From: https://www.youtube.com/watch?v=pbR_BNSOaMk&ab_channel=JohnHammond

---

Ping all hosts in range `10.1.2.0` - `10.1.2.254`.  
Output which of them respond.
```bash
% for i in $(seq 254); do ping 10.1.2.${i} -c1 -W1 & done | grep from
```

---

Get *statically comiled* binaries from GitHub `andrew-d / static binaries` on github.
https://github.com/andrew-d/static-binaries
So you can just upload this on a machine (e.g scp) that doesn't have those binaries and run them.

---

Chisel provides tunneling via SSH.
jpillora / chisel on GitHub
https://github.com/jpillora/chisel

Ultimately we want chisel on both our machine and pivot box.
Which sucks kinda.

Chisel *in reverse* authenticates a server to connect back to by Fingerprint. 

---

