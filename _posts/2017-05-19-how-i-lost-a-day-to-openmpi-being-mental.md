---
title: "How I lost a Day to OpenMPI Being Mental"
date: "2017-05-19 13:28:58 +0100"
tags:
- Code
tags:
- openmpi
- code
- hpc
- mpi
- fortran
image: openmpi-being-mental/cover.jpg
---

So at Glasgow Uni we have this little cluster for the maths department which happens to including about ten machines set up to work with torque (a job scheduling system). I discovered that these machines hadn't had _anything_ run on them for literally months, what a waste of resources! To rectify this atrocity I decided to try and run my MPI enabled code on _all ten machines_.

### Problem One
Turns out two of the machines have eight cores and the other eight have twelve cores. I can't mix and match with core number (AFAIK) so I guess I'll just run the code on _all eight machines_.

### Problem Two
The vendor supplied fortran compiler and openmpi library are pretty old and pretty useless. They produce binaries that run about 3x slower than current versions so I guess I'll have to build the latest versions myself.

### Problem Three
I run the code on a single machine. Works great, not so great performance but I can perhaps sort that out later. I run the code on multiple machines and __instant__ crash.

```
ERROR: received unexpected process identifier
```

Google doesn't help, the only result is someone trying to run openmpi on one machine with multiple IP addresses on a single interface. So I spend _six hours_ looking through the verbose MPI output to discover that upon MPI setting itself up, each MPI process contacts every other process.

Contacting another process on the same machine? Absolutely fine, nae bother.

Contacting another machine over the network? Not fine, definitely bother.

Why? It was contacting the other machines through the interface `virbr0` which happens to have the _same IP address_ on _every single machine_. So the conversation between a machine _and itself_ was going something like this:

Machine A: Hey, 192.168.1.1, you're running this MPI thing?\\
Machine A: Do I know you?\\
Machine A: Apparently not.\\
Machine A: Well, let's decide to produce a very confusing error message and hang.\\
Machine A: Agreed.

Machine B: Me too.

After I found this out simply passing `--mca btl_tcp_if_exclude virbr0,lo` to `mpirun` suppressed any contact through that odd interface and loopback and it worked perfectly.

### Problem Four
It crashes on more than four machines.

### Problem Five
It's about 7 times slower than running on a single one of the newer machines in the cluster.

### Lesson Learned:
Sometimes clusters are better left unused.

---

Header Image Credit: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px;" href="https://unsplash.com/@charlesdeluvio?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Charles Deluvio 🇵🇭🇨🇦"><span style="display:inline-block;padding:2px 3px;"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white;" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px;">Charles Deluvio 🇵🇭🇨🇦</span></a>
