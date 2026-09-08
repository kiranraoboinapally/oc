
# Part 1 — What is a Network?

Before IP addresses, OpenShift, containers, or Kubernetes, let's understand the basic idea of a **network**.

---

## 1. Imagine two computers

Suppose we have:

```text
Computer A                    Computer B

┌────────────┐                ┌────────────┐
│            │                │            │
│   Laptop   │────────────────│   Server   │
│            │                │            │
└────────────┘                └────────────┘
```

The cable/wireless connection allows them to communicate.

For example:

```text
Laptop → "Hello Server"
Server → "Hello Laptop"
```

That's a **network**.

A network is simply a way for devices to **communicate with each other**.

---

# 2. What can be connected?

Not just computers.

A network can contain:

```text
                 Network
                    │
       ┌────────────┼────────────┐
       │            │            │
     Laptop       Server       Phone
       │            │            │
     Printer      Database      Camera
```

In a real company:

```text
                         Network
                            │
       ┌────────────────────┼────────────────────┐
       │                    │                    │
    Users                 Servers             Internet
       │                    │
   ┌───┴───┐          ┌─────┴─────┐
   │       │          │           │
Laptop   Desktop    Web server   Database
```

---

# 3. How does one computer know another computer?

This is where **addresses** come in.

Think about your house.

If I want to send you something, I need:

```text
House address
```

Networking is similar.

A computer needs a network address.

That's the **IP address**.

For example:

```text
Computer A
IP: 192.168.1.10

Computer B
IP: 192.168.1.20
```

Now we have:

```text
192.168.1.10  ───────────>  192.168.1.20
   Computer A                 Computer B
```

So:

> **IP address = a logical address used to identify a device/interface on an IP network.**

---

# 4. Your classroom

Now let's apply this to your actual lab.

You have:

```text
workstation
172.25.250.9
```

and:

```text
master01
172.25.250.140
```

and:

```text
registry
172.25.250.150
```

So you can imagine:

```text
                   Classroom Network

      ┌──────────────┐
      │ workstation  │
      │ .9           │
      └──────┬───────┘
             │
             │
      ┌──────▼───────┐
      │    Network   │
      └──────┬───────┘
             │
       ┌─────┴──────────────┐
       │                    │
       ▼                    ▼
 ┌────────────┐       ┌────────────┐
 │  master01  │       │  registry  │
 │ .140       │       │ .150       │
 └────────────┘       └────────────┘
```

Your workstation can communicate with the other machines because they're connected to the lab network.

---

# 5. But there's another address!

Here's where beginners often get confused.

A computer can have an:

**IP address**

AND a:

**MAC address**

For example:

```text
IP address:
172.25.250.9

MAC address:
52:54:00:12:34:56
```

They are **not the same thing**.

Very roughly:

```text
IP address
   ↓
"Where is the device logically?"

MAC address
   ↓
"Which network interface is it?"
```

We'll go deeply into MAC addresses and **ARP** later.

Don't worry about memorizing that yet.

---

# 6. Network equipment

Computers don't normally all connect directly to each other.

We use networking equipment.

### Switch

A switch connects devices on a local network:

```text
             Switch
          ┌──────────┐
          │          │
     ┌────┤          ├────┐
     │    │          │    │
     ▼    └──────────┘    ▼
  Laptop                Server
```

### Router

A router connects **different networks**:

```text
Network A                     Network B

10.10.10.0/24                10.20.20.0/24

    │                              │
    │                              │
    └────────── Router ────────────┘
```

This distinction becomes **extremely important** later.

---

# 7. Your first mental model

For now, remember:

```text
DEVICE
   │
   │ has
   ▼
NETWORK INTERFACE
   │
   │ has
   ├──────────► MAC address
   │
   └──────────► IP address
                     │
                     ▼
                  NETWORK
                     │
                     ▼
                 SWITCH
                     │
                     ▼
                  ROUTER
                     │
                     ▼
               OTHER NETWORKS
```

Don't worry about all the details yet.

---

# 8. Tiny practical exercise

On your `workstation`, run:

```bash
ip addr
```

Don't worry about understanding the entire output.

Find the line containing something like:

```text
inet 172.25.250.9/24
```

Then look a few lines above it for something like:

```text
link/ether xx:xx:xx:xx:xx:xx
```

You have just found:

```text
IP address  → 172.25.250.9
MAC address → xx:xx:xx:xx:xx:xx
```

That's our starting point.

---

## What we have learned

```text
Network
   ↓
allows devices to communicate

IP address
   ↓
logical network address

MAC address
   ↓
network-interface address

Switch
   ↓
connects devices on a local network

Router
   ↓
connects different networks
```

### Next: Part 2 — IP addresses

We'll take **`172.25.250.9` apart character by character**, explain what the four numbers mean, why they go from `0–255`, what `/24` means, and why your classroom chose `172.25.250.0/24`.

**Don't move to OpenShift yet.** Once IP addressing is solid, the rest becomes much easier.
