# Part 2 — IP Addresses

Now we'll understand **exactly what `172.25.250.9` means**.

Don't worry about OpenShift yet. This is pure networking.

---

## 1. An IP address is 32 bits

An IPv4 address looks like:

```text
172.25.250.9
```

It has **4 sections**, called octets:

```text
172    .    25    .    250    .    9
 ↑          ↑           ↑          ↑
octet      octet       octet      octet
```

Each octet can be from:

```text
0 → 255
```

Why 255?

Because each octet contains **8 bits**:

```text
8 bits = 2⁸ = 256 possible values
```

Those values are:

```text
0 through 255
```

So an IPv4 address has:

```text
4 × 8 = 32 bits
```

---

# 2. Why binary?

Computers ultimately work with bits:

```text
0
1
```

So:

```text
172.25.250.9
```

is really:

```text
10101100.00011001.11111010.00001001
```

You **do not need to memorize this conversion** right now.

Just understand:

> Humans normally write IPv4 addresses in decimal, while computers operate on the underlying binary representation.

---

# 3. Now the important question

If I give you:

```text
172.25.250.9
```

can you tell me:

> Which part is the network?

Not yet.

Because **the IP address alone doesn't tell us where the network boundary is.**

We need another piece of information.

That's the:

# Subnet mask / CIDR prefix

For example:

```text
172.25.250.9/24
```

The `/24` is extremely important.

---

# 4. What does `/24` mean?

It means:

> **The first 24 bits belong to the network portion.**

IPv4 has 32 bits total.

Therefore:

```text
32 - 24 = 8
```

bits remain for hosts.

So:

```text
/24
│
├── 24 bits = network
└── 8 bits  = host
```

Visually:

```text
172.25.250.9
████████████████████████ ████████
       NETWORK              HOST
          24 bits             8 bits
```

Therefore:

```text
Network: 172.25.250
Host:    9
```

More precisely, the network itself is:

```text
172.25.250.0/24
```

---

# 5. What addresses exist in that network?

If we have:

```text
172.25.250.0/24
```

we have:

```text
172.25.250.0
172.25.250.1
172.25.250.2
172.25.250.3
...
172.25.250.254
172.25.250.255
```

There are 256 total addresses.

But two have special purposes:

```text
172.25.250.0
```

is the **network address**.

And:

```text
172.25.250.255
```

is the **broadcast address**.

So traditionally you have:

```text
254 usable host addresses
```

from:

```text
172.25.250.1
```

through:

```text
172.25.250.254
```

---

# 6. Now your classroom makes sense

Your documentation gives us:

```text
workstation = 172.25.250.9
master01    = 172.25.250.140
registry    = 172.25.250.150
utility     = 172.25.250.220
```

Assuming the lab network is:

```text
172.25.250.0/24
```

then:

```text
             172.25.250.0/24
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
   workstation     master01      registry
   .9              .140          .150
```

They're all members of the same IP subnet.

---

# 7. Why does being on the same subnet matter?

This is a **very important networking concept**.

Suppose:

```text
A = 172.25.250.9/24
B = 172.25.250.140/24
```

A wants to communicate with B.

A checks:

```text
Are we on the same network?
```

A:

```text
172.25.250.9
```

B:

```text
172.25.250.140
```

With `/24`, both belong to:

```text
172.25.250.0/24
```

So A says, effectively:

> "B is on my local network."

It doesn't need to send the traffic to a router to reach B.

This eventually leads us to **ARP + MAC addresses + switches**.

That's our next layer.

---

# 8. What if the destination is different?

Suppose your workstation:

```text
172.25.250.9/24
```

wants to contact:

```text
10.10.10.50
```

Now:

```text
172.25.250.9/24
```

and:

```text
10.10.10.50
```

are **not on the same subnet**.

So the workstation needs a **router/default gateway**.

Conceptually:

```text
workstation
172.25.250.9
      │
      │ destination isn't local
      ▼
default gateway
      │
      ▼
other network
10.10.10.0/24
      │
      ▼
10.10.10.50
```

This is why subnetting matters.

---

# 9. `/24` isn't the only possibility

You will see things like:

```text
10.0.0.0/8
172.16.0.0/16
192.168.1.0/24
192.168.1.0/25
192.168.1.0/26
```

The number after `/` tells us where the network/host boundary is.

For example:

```text
/24
24 network bits
8 host bits
```

while:

```text
/16
16 network bits
16 host bits
```

and:

```text
/8
8 network bits
24 host bits
```

This is called **CIDR**.

We'll spend a whole lesson on it because it's extremely important for Kubernetes/OpenShift.

---

# 10. One thing I want you to remember

Don't think:

> "`172.25` means private network."

Instead:

```text
172.25.250.9/24
```

contains **two pieces of information**:

```text
IP address
    ↓
172.25.250.9

Prefix
    ↓
/24
```

Together they tell us:

```text
Address:  172.25.250.9
Network:  172.25.250.0/24
Host:     .9
```

---

# Your mini-test 🧠

Without looking above, answer these:

### Q1

What is the range of one IPv4 octet?

```text
?
```

### Q2

How many bits are in an IPv4 address?

```text
?
```

### Q3

What does `/24` mean?

```text
?
```

### Q4

For:

```text
192.168.10.25/24
```

what is the network address?

```text
?
```

### Q5

Are these two machines on the same `/24` subnet?

```text
A = 172.25.250.9/24
B = 172.25.250.140/24
```

Answer those five. **Then we'll do Part 3: subnetting/CIDR properly**, where I'll show you why `/25`, `/26`, `/27`, etc. exist and how to calculate them without memorizing tables.
