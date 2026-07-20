---
layout: post
title: The bash that wasn't
tags: [security, sysadmin, incident]
description: A process called -bash was eating 15 cores. It was not bash.
---

![A ps listing showing a process named -bash using 40 threads and 1515% CPU, with its binary deleted — captioned "This is not bash. It's a miner."]({{ site.baseurl }}/images/bash-that-wasnt.svg)

It started with `btop`. One process, named `-bash`, quietly devouring fifteen CPU cores like it paid rent. Real `bash` is a shell. It waits for you to type. It does not pin a CPU, let alone fifteen of them.

So I looked closer.

```
ps: 40 threads, 2.4 GB RAM, 1515% CPU, parent PID 1
```

Forty threads. Two and a half gigs. `bash` uses about none of that. This process had simply put on a `bash` costume and hoped nobody would check the receipts. I checked the receipts.

## The receipts

```
/proc/63572/exe -> '/tmp/.bash/bash (deleted)'
```

Three red flags in one line. Running from `/tmp`. In a hidden directory. And **it had deleted its own binary** — the software equivalent of wiping the doorknob on your way out. Legitimate programs do not do this. Cryptominers do this before breakfast.

Its open files told the rest of the story: logs written to an unlinked file (invisible to `ls`), a working file named `.snap-private-bash` to look like something Ubuntu made, and a live connection to `24.199.107.111:80`. Port 80, to blend in with web traffic. A mining pool wearing a hi-vis vest.

## How it got in (spoiler: I held the door)

Two dormant accounts, `john` and `doe`, created long ago for reasons past-me thought were excellent. Weak passwords. SSH password auth still on.

The logs showed **440,000 failed login attempts** across ten days. Then two successes. Then, eight seconds after one login, a miner and a cron job. Efficient work.

But the attempts all came from `::1`. **Localhost.** The call was coming from inside the house — which is impossible, because nobody was in the house.

Except I'd built a secret passage and forgotten about it. An old `autossh` reverse tunnel — `ssh -R 63100:localhost:22` — quietly re-published this box's SSH to the internet via a server running `GatewayPorts yes`. Every internet scanner on Earth could knock on my SSH, and by the time the knock arrived it looked local. So my firewall waved it through. My fail2ban never saw a thing, because the traffic was forwarded raw and never actually *authenticated* on the exposed host. Two security tools, both technically working, both looking the wrong way.

**Lesson one:** a reverse tunnel launders the source IP. Every IP-based defence downstream goes blind. You cannot fix this at the destination. You fix it by not publishing the port.

## The cleanup (in the correct, paranoid order)

1. **Copy the evidence first.** The binary deleted itself, so the only surviving sample lived in `/proc/<pid>/exe`. Kill the process and it's gone forever.
2. **Kill persistence before the process.** I killed PID 63572. Cron cheerfully spawned PID 94204 sixty seconds later. Right. Remove the crontab *first*, then kill. Verify past the next cron tick.
3. **Then, and only then,** kill it and delete `/tmp/.bash`.

**Lesson two:** always check the kill held. Malware with a cron job is a whack-a-mole with a respawn timer.

## The part where I declared victory too early

I removed `/tmp/.bash`, confirmed no processes, and announced the host clean.

The host was not clean.

A later sweep for files owned by the deleted accounts found a **second copy** of the miner sitting in `/tmp/.obbtsq/hsooya`, dropped in the same operation, waiting patiently. It wasn't running — but "I deleted the paths I knew about" is not the same as "I found everything."

**Lesson three:** hunt by owner, hash, and timestamp — not just by the paths you already know. Attackers keep spares.

## The plot twist that wasn't

Then I found `john` and `doe` logging into *another* of my servers by SSH key, on the same day as the miner drop. Heart rate: elevated. Lateral movement?

No. The keys were installed **sixteen months earlier**, long before any of this. Same account names, completely unrelated credentials, on a different machine. The other box used key auth only; the cracked passwords were worthless there. Pure coincidence, expertly timed to ruin my afternoon.

**Lesson four:** same username on two Linux boxes means nothing. Different `/etc/shadow`, different keys, different everything. Correlate on mechanism and timing, not names — and check file mtimes before you panic.

## Bonus: the OOM that wasn't the malware either

That night, `sshd` died and the box ran out of memory. Surely the miner?

The miner had been dead for eight hours. The real culprit: a MySQL container configured with an `innodb_buffer_pool_size` of **64.5 GB**. On a 31 GB machine. To serve 14 GB of data. It was reserving twice the physical RAM to cache a database smaller than the reservation, on a host with zero swap. The kernel did the only thing it could and started shooting.

**Lesson five:** not every fire is arson. Sometimes it's just capacity planning done by a percentage-of-RAM formula that, inside a container, reads the *host's* RAM. Set your buffer pool to a literal number. Cap your containers. Keep some swap around as a seatbelt.

## The scoreboard

- **Root cause:** forgotten accounts + weak passwords + a reverse tunnel I built myself.
- **Blast radius:** one unprivileged miner. No root, no data, no lateral movement.
- **Time held:** ~5 weeks of access, 7 days of actual mining.
- **Dignity lost:** catastrophic.

The attacker was not sophisticated. They ran a commodity username dictionary against a door I'd propped open and dropped an off-the-shelf miner. That's the humbling part — this wasn't a master thief. It was someone walking down the street trying every doorknob, and mine turned.

Close your dormant accounts. Kill password auth. And go run `ss -tlnp | grep -v 127.0.0.1` right now — I'll wait. You might be surprised what's saying hello to the internet on your behalf.
