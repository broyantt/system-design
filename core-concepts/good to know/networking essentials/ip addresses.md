
An IP address is just a **32-bit number** — but instead of showing you `11000000101010000000000100101101` they write it as 4 groups of decimal numbers separated by dots:

```
192      .    168      .    1        .    45
11000000      10101000      00000001      00101101
```

Each group (called an **octet**) is 8 bits, so it can be any number from 0–255.  An IP address is just a human-readable label for a 32-bit number that uniquely identifies a device on a network.

Because it's 32 bits, IPv4 can represent **~4.3 billion unique addresses** (2³²). The internet grew so big we're running out of them, which is why IPv6 exists (128 bits, practically unlimited addresses).

---

## What is a Subnet?

The internet isn't one giant flat network where every device talks directly to every other device. It's organized into **smaller networks called subnets** (short for subnetwork).

```
The Internet
├── Google's network        (a subnet)
├── AWS's network           (a subnet)
├── Telkom Indonesia        (a subnet)
│   ├── Jakarta region      (a smaller subnet)
│   └── Bandung region      (a smaller subnet)
└── Your home network       (a tiny subnet)
```

A subnet is just a **contiguous block of IP addresses that belong to the same network**. Devices within the same subnet can talk to each other directly. To talk to a device outside your subnet, traffic has to go through a router.

---

## Subnet Masks — How Subnets Are Defined

Every subnet is defined by two things:

- A **network address** — the starting IP of the subnet
- A **subnet mask** — defines how large the subnet is

It's written in **CIDR notation** like this:

```
192.168.1.0/24
```

The `/24` means the **first 24 bits are the network part** (fixed, identifies the subnet), and the **remaining 8 bits are the host part** (variable, identifies individual devices within the subnet).

```
192.168.1  .  0
|--------|    |--|
network         host
(fixed 24 bits) (8 bits = 256 possible addresses)
```

So `192.168.1.0/24` covers addresses `192.168.1.0` through `192.168.1.255` — that's 256 addresses in this subnet.

---

## CIDR Notation — Reading It Quickly

The number after the slash tells how many bits are locked to the network:

```
/8   → 8 bits network,  24 bits host → 16 million addresses  (huge, e.g. a country ISP)
/16  → 16 bits network, 16 bits host → 65,536 addresses      (large corp network)
/24  → 24 bits network, 8 bits host  → 256 addresses          (typical home/office)
/32  → 32 bits network, 0 bits host  → 1 address              (a single specific device)
```

Bigger the number → smaller the subnet → fewer devices it can hold.

---

## Private vs Public IP Ranges

Not all IP addresses are equal. Certain ranges are **reserved for private networks** by international agreement — routers on the public internet will never forward packets to these addresses:

```
10.0.0.0/8          → 10.x.x.x           (large private networks, AWS VPCs use this)
172.16.0.0/12       → 172.16.x.x through 172.31.x.x
192.168.0.0/16      → 192.168.x.x        (your home network)
```

Everything else is public and routable on the internet.

This is why your laptop gets `192.168.1.x` — it's in the private range. Your home router has one public IP facing the internet, and NAT handles the translation between them.

---

## How NAT Actually Works

NAT (**Network Address Translation**) is what allows multiple private devices to share a single public IP. The router maintains a **translation table**:

```
When your laptop (192.168.1.5) requests google.com:

1. Packet leaves laptop:
   src: 192.168.1.5:54321  →  dst: 142.250.4.46:443

2. Router rewrites the source:
   src: 116.206.x.x:54321  →  dst: 142.250.4.46:443
   (stores this mapping in its NAT table)

3. Google responds to your router's public IP:
   src: 142.250.4.46:443   →  dst: 116.206.x.x:54321

4. Router checks NAT table, rewrites destination:
   src: 142.250.4.46:443   →  dst: 192.168.1.5:54321
   (delivers to your laptop)
```

Our laptop never exposes its private IP to the internet. Google only ever sees our router's public IP.
