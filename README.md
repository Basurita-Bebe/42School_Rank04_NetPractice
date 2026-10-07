*This project has been created as part of the 42 curriculum by bbeceiro.*

# NetPractice


## DESCRIPTION

NetPractice is a practical introduction to computer networking. It is a set of 10 small simulated networks,
opened in a web browser, where each one is broken on purpose. A level is solved when the machines named in
its goals can exchange **packets** (small units of data, each carrying the address it comes from and the
address it is going to) in both directions.


# To repair a network, three kinds of fields are filled in:

- **IP addresses:** the number that identifies one machine's connection to a network, written as four numbers
from 0 to 255, for example `104.198.14.2`.

- **Subnet masks:** the setting that says which part of an IP address names the network and which part names
the machine inside that network.

- **Routes:** lines that tell a machine where to send packets for networks it is not directly connected to.


# The diagrams contain four kinds of machines:

- **Hosts:** ordinary computers, with one network connection each.

- **Switches:** boxes that connect several machines inside the same network.

- **Routers:** machines that connect different networks to each other and pass packets from one to the next.

- **The Internet:** from level 6, a single node that stands for everything outside the local networks.

Nothing is coded in this project. What it trains is the reasoning behind TCP/IP addressing: deciding which
machines belong to the same network, how a packet chooses where to go next, and how the answer finds its way back.

Every networking term used in this README is defined in the
[Resources] section.

| Levels | What they introduce |
|--------|---------------------|
| 1 – 5  | IP addresses and subnet masks, hosts on the same network, switches, a first router and default gateways.
| 6      | The Internet as a node: a default route on the router and a return route on the Internet side.
| 7      | Two routers in a chain: one network per cable, no overlapping networks.
| 8      | Splitting one address pool into several networks, and one route that covers several of them.
| 9      | Five networks at once, a `/18` mask, public versus private addresses |
| 10     | Fitting a last network into the space left by fixed addresses, and a single route for the whole pool.

During the defense, three random levels from 6 to 10 must be solved in a limited time, with newly generated
addresses and no external tools except a basic calculator.


## INSTRUCTIONS

### Running the training interface

The interface is a static website that must be served by a local web server.

	sh
	cd net_practice
	./run.sh

The script starts a web server and opens the page in the default browser. If it does not work, start the server
manually and open the URL yourself:

	sh
	cd net_practice
	python3 -m http.server 49242

Then browse to http://localhost:49242


On the welcome page:

- **Training** tab: enter your intranet login. The addresses are generated from the login, so the exported files
are tied to it.
- **Evaluation** tab: generates three random levels with random addresses, as used during the defense.


### Solving a level :

Each level shows a network diagram and one or more goals at the top. Only the unshaded fields can be edited.
**Check again** tests the configuration, and the log panel at the bottom explains, hop by hop, where a packet was
dropped.


### Exporting a configuration :

When a level is solved, click **Get my config** before moving to the next level. The browser downloads one file
for that level (`level1.json`, …).


### Submission details :

- **10 exported configuration files, one per level, must be placed at the root of the repository.**
- They must be exported with the intranet login entered in the interface.
- This `README.md` must also be at the root.
- Only the content of the repository is evaluated.


## How I approach a level :

1. **Count the networks.** One per cable. A switch joins its cables into a single network; a router separates them.

2. **Read the fixed (grey) fields.** They decide the masks and often the addresses of their neighbours.

3. **Give every network its own block** (its own range of addresses). No overlap, and no private addresses for
anything that must reach the Internet.

4. **Hosts:** a default route (the route used when nothing more precise matches) whose gateway is the router interface
on the host's own cable.

5. **Routers:** a default route towards the Internet, and precise routes for the networks hidden behind another router.

6. **Internet:** one return route for each network that has to reach it.

7. **Check both directions.** A goal needs a forward way (the packet reaches its destination) and a reverse way (the
answer comes back).


### Notes that helped me :

- Block size in the last byte = `256 − last mask number`. Blocks start at 0 and repeat, and a network is the block that
contains the interface address.
- In every block, the first address is the network address and the last one is the broadcast address. Neither can be given
to an interface.
- A route reads `destination => next hop`. The destination is a network (`network address/prefix`) that can be far away.
The next hop is always a direct neighbour, on one of the machine's own networks.
- Routes are read from top to bottom and the first match wins, so precise routes go above the default route.
- The Internet does not route private addresses: `10.0.0.0/8`, `172.16.0.0/12` and `192.168.0.0/16`.

| Prefix | Mask              | Block size | Usable addresses |
|--------|-------------------|-----------:|-----------------:|
| /24    | 255.255.255.0     | 256        | 254              |
| /25    | 255.255.255.128   | 128        | 126              |
| /26    | 255.255.255.192   | 64         | 62               |
| /27    | 255.255.255.224   | 32         | 30               |
| /28    | 255.255.255.240   | 16         | 14               |
| /29    | 255.255.255.248   | 8          | 6                |
| /30    | 255.255.255.252   | 4          | 2                |

A prefix below `/24` cuts an earlier byte in the same way: `/18` is
`255.255.192.0`, blocks of 64 in the third byte.


## RESOURCES

### Networking concepts studied :

Each concept below is given with a short definition and with what it meant in practice in this project.


#### TCP/IP addressing :

- **TCP/IP:** the family of protocols (shared rules for communication) that the Internet runs on. **IP** (Internet Protocol)
gives every machine an address and carries packets from one network to another. **TCP** (Transmission Control Protocol)
works on top of IP and makes sure the data arrives complete and in order. NetPractice only exercises the IP part.
- **IPv4 address:** a 32-bit number that identifies one interface, written as four numbers from 0 to 255 separated by dots
(`93.198.14.2`). Each of the four numbers is one byte, which is 8 bits.
- **Interface:** one network connection of a machine. Every interface has its own IP address and its own mask. A host has one
interface, a router has one per network it is connected to.
- **Network (or subnet):** a group of addresses that share the same network part. Machines in the same network reach each other
directly, without a router.
- **Network address:** the first address of a network. It names the network itself and cannot be given to an interface.
- **Broadcast address:** the last address of a network. It is reserved for messages sent to every machine in that network and
cannot be given to an interface either.
- **Public and private addresses:** public addresses are unique on the Internet. Private addresses (`10.0.0.0/8`, `172.16.0.0/12`,
`192.168.0.0/16`) can be reused freely inside local networks, and the Internet refuses to carry packets addressed to them.
- **Loopback:** the range `127.0.0.0/8`, which a machine uses to talk to itself. It cannot be used on an interface that faces a
network.


#### Subnet masks :

- **Subnet mask:** a 32-bit value that splits an IP address in two. The bits set to 1 mark the *network part*, and the bits set to
0 mark the *host part* (the machine inside the network). Two interfaces are in the same network when they have the same mask and
the same network part.
- **Dotted notation:** the mask written like an address, for example `255.255.255.0`.
- **CIDR prefix:** the same mask written as the number of bits set to 1, after a slash. `/24` is `255.255.255.0`, `/30` is
`255.255.255.252`.
- **Block:** the range of addresses covered by one network. Its size depends on the mask: a `/26` gives blocks of 64 addresses, a
`/30` gives blocks of 4.
- **Subnetting:** cutting one range of addresses into several smaller networks by using a longer prefix. In level 8, one `/26`
pool is cut into three networks.
- **Route aggregation:** the opposite, describing several networks with one shorter prefix that contains them all. In level 10,
one `/24` route on the Internet covers all four local networks.


#### Default gateways and routing :

- **Routing table:** the list of routes held by a machine. When a packet is not for one of the machine's own networks, the table
is read from top to bottom and the first matching route is used.
- **Route:** one line of that table, written `destination => next hop`.
- **Destination:** the network the route is about, written as a network address and a prefix (`93.198.14.4/30`). It can be several
routers away.
- **Next hop (or gateway):** the address of the neighbour the packet is handed to. It must belong to one of the machine's own networks.
- **Default route:** the route that matches every destination, written `default` or `0.0.0.0/0`. It is used when no more precise route
matches.
- **Default gateway:** the next hop of the default route. For a host, it is the interface of its router on the host's own network.
- **Forward way and reverse way:** the two halves of a communication. The packet must reach its destination, and the answer must be
able to come back, so routes are needed in both directions.


#### Routers and switches :

- **Switch:** a device that connects several machines inside one network. It passes data between its cables without looking at IP
addresses, and in this project it has no IP address and no routes. All machines plugged into the same switch must be in the same network.
- **Router:** a device that connects different networks. It has one interface in each network and uses its routing table to forward
packets from one to another. Two interfaces of the same router can never be in the same network.


#### OSI layers :

The **OSI model** is a reference model that describes network communication as seven layers stacked on top of each other, each one
relying on the layer below. This project is about layer 3.

| Layer | Name         | Role                                             | In NetPractice                        |
|------:|--------------|--------------------------------------------------|---------------------------------------|
| 7     | Application  | What programs use (web, mail)                    | Not covered                           |
| 6     | Presentation | Format and encryption of the data                | Not covered                           |
| 5     | Session      | Opening and closing exchanges                    | Not covered                           |
| 4     | Transport    | Delivery between two programs (TCP, UDP)         | Not covered                           |
| 3     | Network      | Addresses and routing between networks (IP)      | IP addresses, masks, routes, routers  |
| 2     | Data link    | Delivery inside one network                      | Switches                              |
| 1     | Physical     | Cables and signals                               | The lines drawn between the machines  |


### References :

- [RFC 791 – Internet Protocol](https://www.rfc-editor.org/rfc/rfc791)
- [RFC 1918 – Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918)
- [RFC 4632 – Classless Inter-domain Routing (CIDR)](https://www.rfc-editor.org/rfc/rfc4632)
- [Cloudflare Learning – What is a subnet?](https://www.cloudflare.com/learning/network-layer/what-is-a-subnet/)
- [Wikipedia – Subnet](https://en.wikipedia.org/wiki/Subnet)
- [Wikipedia – Classless Inter-Domain Routing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)
- [Wikipedia – Default gateway](https://en.wikipedia.org/wiki/Default_gateway)
- [Wikipedia – Routing table](https://en.wikipedia.org/wiki/Routing_table)
- [Wikipedia – OSI model](https://en.wikipedia.org/wiki/OSI_model)


### How AI was used :

I used an AI assistant for the following tasks:

- **Learning, levels 6 to 10:** as a tutor. It explained the theory, gave worked examples with numbers different from my levels
and asked me questions to check understanding.
- **This README:** I wrote a draft then, the AI helped me with structure and syntax.
