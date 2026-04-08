---
title: "Packet Radio, Part 1: The Stack"
date: 2026-04-07 09:00:00
tags: Packet Radio, AX.25, NET/ROM, Amateur Radio, BPQ32, Direwolf
---

This is the first in a short series about VHF packet radio. Later posts will cover a specific station build and on-air operation; this one is a reference — a map of the protocols and software involved, and how they relate. Packet has a lot of acronyms. Most of them make more sense once you see how they layer onto each other — and which pieces are drop-in alternatives, like Direwolf standing in for a hardware TNC.

HF packet has its own quirks (different tones, LSB instead of FM, sparser activity) and may get its own post later.

## Packet Radio and the OSI model

Here's the short version; the rest of the post walks down the stack.

| OSI layer       | Packet-radio equivalent                                  | Example software / hardware     |
|-----------------|----------------------------------------------------------|----------------------------------|
| 7 Application   | BBS (FBB, BPQMail)                                       | BPQMail, BPQTerminal, Paracon    |
| 4 Transport     | NET/ROM circuits                                         | BPQ32                            |
| 3 Network       | NET/ROM routing (NODES broadcasts, quality metric)       | BPQ32                            |
| 2 Data link     | AX.25 (UI frames + connected mode)                       | Direwolf or BPQ32's AX.25 stack  |
| 1 Physical      | 1200 baud AFSK / Bell 202 tones over FM VHF              | VHF radio + hardware or software TNC |

## Layer 1 — Physical: the radio and the modem

At the bottom is an ordinary VHF FM radio, used as a pipe for audio tones. The computer generates two audio tones — one called **mark** and one called **space** — feeds that audio into the radio's mic input, and the radio transmits it as ordinary FM voice. It has no idea it's carrying data. The receiving station does the reverse. This scheme of encoding data by shifting between audio frequencies is called **audio frequency-shift keying**, or AFSK.

The bits are not a direct 1:1 mapping to the two tones. Amateur packet uses NRZI encoding on top of AFSK: [per Wikipedia's Packet radio article](https://en.wikipedia.org/wiki/Packet_radio), "a data zero bit is encoded by a change in tones and a data one bit is encoded by no change in tones." The data rides in the *transitions* between mark and space, not in which tone is present at any given instant.

The dominant flavor on VHF is 1200 bits per second using **Bell 202 tones** — 1200 Hz for mark, 2200 Hz for space — a standard originally designed for 1970s telephone modems ([Wikipedia: Packet radio](https://en.wikipedia.org/wiki/Packet_radio)). The practical consequence is that 1200 bit/s is very slow by modern standards, which is why almost everything above this layer is built around short messages and efficient retries rather than throughput.

Between the radio and the computer sits a **TNC** (Terminal Node Controller). A TNC does two jobs:

1. **Modem** — generate Bell 202 tones on transmit, decode them on receive.
2. **Framer** — group bits into discrete, self-contained units called **AX.25 frames**. A frame bundles a destination address, a source address, a payload, and a checksum into one atomic chunk that either arrives intact or is thrown out.

Everything above this layer thinks in frames, not bits.

A TNC can be dedicated hardware — the [Mobilinkd TNC3](https://store.mobilinkd.com/) is a common modern example — or it can be software running on the host computer, paired with a USB sound-card-plus-PTT interface like the [Digirig Mobile](https://digirig.net/) for audio and PTT to the radio. The software TNC most stations reach for today is [**Direwolf** by WB2OSZ](https://github.com/wb2osz/direwolf), which describes itself as a "software 'soundcard' AX.25 packet modem/TNC and APRS encoder/decoder."

Direwolf exposes its framed output to host applications over two interfaces:

- **KISS** — a deliberately dumb pipe: the host sends and receives raw AX.25 frames and the TNC just modulates and demodulates. [Designed by Mike Cheponis (K3MC) and Phil Karn (KA9Q) in 1987](https://en.wikipedia.org/wiki/KISS_(TNC)) to move protocol logic out of "smart" TNCs and into the host.
- **AGW** — an alternative host interface originally from SV2AGW's AGW Packet Engine. It exposes more structured per-port information than KISS, and several terminal programs prefer it. Direwolf's README lists both ["KISS Interface (TCP/IP, serial port, Bluetooth) & AGW network Interface (TCP/IP)"](https://github.com/wb2osz/direwolf).

## Layer 2 — Data link: AX.25

AX.25 is the link-layer protocol that amateur packet runs on. It was [derived from layer 2 of the X.25 protocol suite](https://en.wikipedia.org/wiki/AX.25) in the early 1980s and adapted for amateur use — most notably, its addresses are callsigns instead of the numeric addresses X.25 used.

**SSIDs.** Each AX.25 address is a callsign plus a 4-bit Secondary Station Identifier, [giving a range of 0 through 15](https://en.wikipedia.org/wiki/AX.25). SSIDs let a single licensee run several logically distinct services at the same station — one sysop's station might present as `W9CPZ-4` for the NET/ROM node switch, `W9CPZ-11` for the BBS, and `W9CPZ-12` for a chat server.

**Two modes of operation.** AX.25 [supports both a connected virtual-circuit mode and a connectionless datagram mode](https://en.wikipedia.org/wiki/AX.25):

- **UI frames (unconnected)** — broadcast, one-shot, no acknowledgement. If somebody hears them, great; if not, they're gone. APRS, beacons, and NET/ROM's own routing broadcasts all ride on UI frames.
- **Connected mode** — a reliable, ordered, point-to-point session between two specific stations. "Connecting to" another packet station means setting up one of these.

### Digipeaters: a layer-2 repeater

Before NET/ROM existed, the only way to extend a packet station's reach beyond one RF hop was the **digipeater**: [Wikipedia defines one](https://en.wikipedia.org/wiki/Digipeater) as "a repeater node in a packet radio network [that] performs a store and forward function, passing on packets of information from one node to another."

Mechanically, this works through AX.25's address field: alongside source and destination, a frame can carry a list of digipeater callsigns to be relayed through. A station that sees its own callsign in that list rebroadcasts the frame; a station that doesn't, ignores it. The sender specifies the path in advance, every frame carries it, and the digipeaters themselves hold no state about the network.

This is the classic layer-2 extension mechanism, and it still exists, but it has obvious limits: the sender has to know the path ahead of time, and there is no way to route around a hop that becomes unreachable. That is the problem NET/ROM set out to solve.

## Layer 3 — Network: NET/ROM

NET/ROM is a [protocol used extensively by radio amateurs](https://www.mankier.com/4/netrom) that adds a routing layer on top of AX.25. NET/ROM nodes form a mesh: each one maintains a table of the other nodes it knows about, learns new ones by listening, and connects user-visible "node switches" together with internode AX.25 links.

Each node periodically broadcasts a **NODES frame** — a UI frame listing which other nodes it can reach and how good each path looks. Every other node that hears it updates its routing table accordingly. [Packet-Radio.net describes the mechanism](https://packet-radio.net/netrom-link-quality/) as "self learning, building routing tables from broadcasts heard from other nodes."

The "how good" part is the **quality metric** — a number from 0 to 255 that, per [Packet-Radio.net](https://packet-radio.net/netrom-link-quality/), "is used by the software to select the route to use when more than one route exists between two nodes." Higher is better. The standard default for a freshly heard neighbor is 192. When a route spans multiple hops, the route quality is the product of the link qualities divided by 256 — so two hops at quality 192 come out to 144, and the numbers degrade as hops chain together.

From the user's perspective, the topology does not have to be known in advance. Connect to the local NET/ROM node, type `C DESTINATION`, and the network finds the path.

### Digipeater vs NET/ROM node, at a glance

| Aspect             | Digipeater                          | NET/ROM node                                 |
|--------------------|-------------------------------------|----------------------------------------------|
| Layer              | 2 (data link)                       | 3/4 (network + transport)                    |
| Knows the topology | No                                  | Yes — learns from NODES broadcasts           |
| Path selection     | Caller specifies it                 | Node chooses, based on quality metric        |
| Failure handling   | Silent drop                         | Can try alternate routes if one exists       |
| Typical user call  | `C DEST VIA W9CPZ`                  | `C DEST` (connect to local node first)       |

## Layer 4 — Transport

NET/ROM does not stop at routing. It also provides a transport-layer **circuit** — a reliable, ordered, end-to-end connection between two nodes, layered on top of whatever AX.25 hops it took to get there. "Reliable and ordered" is the same guarantee TCP makes: data sent at one end comes out the other end exactly once, in the order it was sent, with retransmissions hidden from the application.

Unlike TCP, though, NET/ROM preserves message boundaries. The Linux [`netrom(4)` man page](https://www.mankier.com/4/netrom) notes that NET/ROM "only supports connected mode" and presents itself to applications as a `SOCK_SEQPACKET` socket, which [`socket(2)`](https://man7.org/linux/man-pages/man2/socket.2.html) defines as "a sequenced, reliable, two-way connection-based data transmission path ... for datagrams of fixed maximum length." TCP by contrast uses `SOCK_STREAM`, which is an undifferentiated byte stream where send boundaries are not preserved.

## Layer 7 — Applications

On top of that transport sit the actual **applications** that make packet radio useful. The dominant one — and the focus of the rest of this series — is the **BBS** (Bulletin Board System), a multiuser mailbox and message store. FBB (F6FBB) and [BPQMail by G8BPQ](https://www.cantab.net/users/john.wiseman/Documents/BPQ32.html) are the two most commonly seen on the air today.

### Hierarchical BBS addressing

BBSes add their own addressing on top of everything else: the hierarchical format `CALL.#REGION.STATE.COUNTRY.CONTINENT`, as [documented by Ohio Packet](https://ohiopacket.org/index.php/BBS_Hierarchical_Routing_Addresses). A concrete example from my area is `W9CPZ.#WWA.WA.USA.NOAM`. The `#` prefix on the local-area component is a convention [adopted to avoid collisions](https://www.qsl.net/w6fff/part7.htm) with numeric postal codes used as local identifiers in some countries.

The forwarding rule is postal-routing in reverse: broadest on the right, most specific on the left. Each BBS along the way consults a **forward file** of suffixes it knows how to forward toward. [Per qsl.net/w6fff](https://www.qsl.net/w6fff/part7.htm), the BBS "will attempt to find a match ... starting with the left-most item in the address field" and falls back rightward when a more specific component doesn't match. The design goal, [per Ohio Packet](https://ohiopacket.org/index.php/BBS_Hierarchical_Routing_Addresses), is that "an intermediate BBS system does not need to know how to get directly to the destination BBS, only the next 'hop.'"

Concretely: a neighboring BBS that knows about `#WWA` (Western Washington) keeps a `W9CPZ.#WWA.WA.USA.NOAM` message local; one that only recognizes `.USA.NOAM` punts it toward a long-haul gateway.

This addressing is an application-layer convention. NET/ROM moves individual sessions between two specific node callsigns; the BBS hierarchy describes where a *message* should ultimately land, regardless of which links carried it. A message may pass through several BBSes on its way, each opening its own connected-mode session to the next — sometimes a direct AX.25 link, sometimes a NET/ROM circuit over multiple hops — until it finally lands in a mailbox.

## Coming up in part 2

Part 2 will walk through a specific VHF station build: the radio, the TNC, and a minimal `bpq32.cfg`. Part 3 will cover the on-air side — learning routes dynamically from a neighboring NET/ROM node, bringing up the BBS, and forwarding mail to another sysop.
