---
layout:	post
title:	"What is DNS? A crash course on DNS for every developer"
Description: "FDid you ever think who understands google.com when you type that in your address bar? 🤔
How does this magic happen? ✨ There isn't a delivery boy to hand over an HTML page directly. Let's understand the crux of this superfast procedure 🚀 starting from typing a URL address to getting a page in your browser. 💻 
If you ever need to set up a blog/website, this is a must-know article that contains all the understanding of DNS, the lookup process, types of DNS records i.e. all you need to know explained within a single article."
date:	2021-03-21
categories: [ 'Programming', 'Web', 'Networking' ]
image: 0_CbBTtDG7nDVQVsE3
author: admin
---

Did you ever think who understands google.com when you type that in your address bar? 🤔


How does this magic happen? ✨ There isn't a delivery boy to hand over an HTML page directly. Let's understand the crux of this superfast procedure 🚀 starting from typing a URL address to getting a page in your browser. 💻 


If you ever need to set up a blog or website, this is a must-know article that contains all the understanding of DNS, the lookup process, types of DNS records i.e. all you need to know explained within a single article.

{% include pictures.html img="0_CbBTtDG7nDVQVsE3" alt="DNS is internet's phonebook library" %}
*DNS is internet's phonebook library*

***
### Table of Contents

1. What is DNS?
2. Why is it required?
3. Types of DNS servers
4. How DNS resolution takes place via DNS servers?
5. Types of DNS Query
6. What is DNS caching? Where is it cached?
7. DNS Record Types and Bonus cheatsheet
8. Issues and Concerns

***

### What is DNS?

DNS stands out for "**Domain Name System**" which translates human-readable domain names (for example, *www.amazon.com*) to machine-readable IP addresses (for example, *192.0.2.44*). DNS system is a public, hierarchical, distributed, and heavily cached database.

> DNS is like the Internet's own phonebook. Records are searched in order to identify the IP address associated with a particular name

An **IP address** is that **unique** address by which that server gets identified device on the internet (or on a local network). Once our computer/device has the IP address then we actually connect to the concerned website's server.

{% include pictures.html img="1_fSNP08gBc6O2-LnBjz3Bfw" alt="DNS server" %}
*Figure 1: **DNS server***

Thus DNS resolution of a website's address involves a lookup process which is the DNS lookup.

***

### Why it is required?

The domain name system (DNS) does the job of enabling a connection, for example to a website, without users having to know the corresponding IP.

Simple isn't it? 😊

***

### Types of DNS Servers

#### 1. Root Name Server
DNS root nameservers are known to every recursive resolver based on the extension of that domain (*.com*, *.net*, *.org*, etc.) and a request is further assigned to a particular TLD responsible for such extension.
> The root nameservers are overseen by a nonprofit called the Internet Corporation for Assigned Names and Numbers (**ICANN**).

#### 2. TLD Name Server
A TLD nameserver maintains information for all the domain names that share a common domain extension, such as .com, .net
> Management of TLD nameservers is handled by the Internet Assigned Numbers Authority (**IANA**), which is a branch of ICANN. 

The IANA breaks up the TLD servers into two main groups:
- **Generic top-level domains **-  domains that are not country-specific such as *.com*, *.org*, *.net*, *.edu*, and *.gov*.
- **Country code top-level domains** - any domains that are specific to a country or state such as *.uk* , *.us*, *.ru*, and *.jp*.

#### 3. Authoritative Name Server
When a recursive resolver receives a response from a TLD nameserver, that response will direct the resolver to an authoritative nameserver. The authoritative nameserver is usually the resolver's last step in the journey for an IP address.

***

### How DNS resolution takes place via DNS servers ⚙️ ?

**Step 1:** When you enter a URL into your browser, it starts searching for the corresponding IP-address in your computer's DNS cache. If it finds no information there, the request will be redirected until the IP address will be identified. 

**Step 2:** Then it passes the local DNS-Server (usually your internet router), the ISP(Internet service provider) DNS-Server, and the root name server, which is accountable for the respective Top Level Domain (**TLD**). 

**Step 3:** If there is still no information found, the request will be sent to the Network Information Center (**NIC**) responsible for the zone. In the case of the TLD *".com"*, this is Verisign.

**Step 4:** The NIC's server will send the address of the zone's authoritative nameserver to the ISP. The ISP will then ask this authoritative server for the IP, and send the information through your router back to your browser. That way the website can be accessed.

{% include pictures.html img="1_LbOZIL34bKbaeaEeKijCLA" alt="DNS Query resolution" %}
*Figure 2: **DNS Query Resolution***

***

### Types of DNS Query
- Recursive
- Iterative

A typical uncached DNS lookup will involve both recursive and iterative queries. Here, in figure 2- 
1. Query no. 2 to 7 are iteratively performed by the from ISP-Local DNS a.k.a recursive resolver.
2. DNS query after 1 is recursively performed by the recursive resolver (from the browser point of view)

> A typical uncached DNS lookup will involve both recursive and iterative queries.

***

### What is DNS caching? Where is it cached?

The purpose of caching is to serve the last stored record for improvements in **performance** and **reliability** for future requests. 

DNS caching involves storing data closer to the requesting client so that the DNS query can be resolved earlier and additional queries further down the DNS lookup chain can be avoided, thereby improving load times and reducing bandwidth/CPU consumption. DNS data can be cached in a variety of locations, each of which will store DNS records for a set amount of time determined by a time-to-live (**TTL**).

> At each level, caching takes place for the fetched records using TTL and other possible cache refreshment algorithms.

#### 1. Browser level
Moderns web browsers can do that. For e.g. you can check in chrome at -
*chrome://net-internals/#dns*

#### 2. OS level
The operating system level DNS resolver is the second and last local stop before a DNS query leaves your machine. The process inside your operating system that is designed to handle this query is commonly called a "stub resolver" or DNS client.

***

### DNS Record Types
When you register a domain, you can set many types of DNS records. Each record has a Type, a Host, and a Value - 
- "**Types**" are predefined
- "**Host**" represents the root (*@*) or a subdomain (*www*)
- "**Value**" is an IP or web address, or some other value.

1. **Address Mapping record (A Record)** - maps a hostname to the corresponding IPv4 address.

2. **IP Version 6 Address record (AAAA Record)** - maps a hostname to the corresponding IPv6 address.

3. **Canonical Name record (CNAME Record)** - alias a hostname to another hostname. If a record that contains a CNAME points to another hostname, the DNS resolution is repeated with the new hostname.
Once you define a *CNAME* record for a subdomain (host), you CAN'T DEFINE another record for that same subdomain. Because of this, you can't use *CNAME* at the root level (where you need other records to exist)

4. **Alias Name Record (ANAME Record)** - An *ANAME* record is like a *CNAME* record but at the root of your domain. That means you can point the "naked" version of your domain (like example.com ) to a hostname (like mycdn.com). 
**This is like a virtual CNAME and a non-standard DNS record**.

5. **Mail eXchanger record (MX Record)** - specifies an SMTP email server for the domain.

6. **Name Server records (NS Record)** - specifies that a DNS Zone, such as "*hello.com*" is delegated to a specific Authoritative Name Server, and provides the address of the name server.

7. **Reverse-lookup Pointer records (PTR Record)** - allows a DNS resolver to provide an IP address and receive a hostname (reverse DNS lookup).

8. **Certificate record (CERT Record)** - stores encryption certificates such as PKIX, SPKI, PGP, and so on.

9. **Service Location (SRV Record)** - a service location record, like MX but for other communication protocols.

10. **Text Record (TXT Record) **- typically carries machine-readable data such as opportunistic encryption, sender policy framework, etc.

11. **Start of Authority (SOA Record)** - this record appears at the beginning of a DNS zone file, and indicates the Authoritative Name Server for the current DNS zone, contact details for the domain administrator, domain serial number, etc.

***

### Bonus cheatsheet 🎉
Here is a bonus cheat sheet for all the important records for you to remember.

{% include pictures.html img="1_EsgHQALk21bxbCPQKXY44g" alt="DNS Query resolution" %}
*Figure 3: **A cheat sheet for important DNS records***


***

### Issues and Concerns 🐛
DNS cache poisoning is the act of entering false information into a DNS cache so that DNS queries return an incorrect response and users are directed to the wrong websites. DNS cache poisoning is also known as 'DNS spoofing.' 

IP addresses are the 'room numbers' of the Internet, enabling web traffic to arrive in the right places. DNS resolver caches are the 'campus directory,' and when they store faulty information, traffic goes to the wrong places until the cached information is corrected.

Because there is typically no way for DNS resolvers to verify the data in their caches, incorrect DNS information remains in the cache until the **TTL** expires, or until it is removed manually. 

A more secure DNS protocol called [DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions){:target="_blank"} aims to solve some of these problems, but it has not been widely adopted yet.

***

### References:

1. [Cloudflare - what is DNS](https://www.cloudflare.com/learning/dns/what-is-dns/){:target="_blank"} 
2. [Seobility - DNS records and types](https://www.seobility.net/en/wiki/DNS_Server){:target="_blank"} 

***

Thanks for reading this article! Feel free to leave your comments and let us know what you think. Please feel free to drop any comments to improve this article.  

Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com), Have a great day!

  