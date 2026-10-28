# Packet Capture Investigation: DNS Tunnelling, Port Scan and Plaintext Login

A hands-on analysis of a network capture from a lab exercise (Cybera
Academy Cybersecurity Foundation Programme, Week 2). I was asked to
find three suspicious things hidden in normal traffic and report on
them. I used Wireshark in Kali Linux.

## What I found

### 1. DNS tunnelling
Filtering on `dns` showed the computer at 10.0.2.15 sending dozens of
lookups with long, random-looking subdomains of one domain,
`sync-update-cdn.info`. Every lookup resolved to the same address,
10.10.10.10. A normal lookup in the same capture, for `github.com`,
resolved to a real public address. The filter `dns.qry.name.len > 40`
picked out the unusually long names.

This matches DNS tunnelling, where stolen data is hidden inside DNS
lookups, because DNS is usually allowed out of a network.

### 2. Port scan
Filtering on `tcp.flags.syn==1` showed 10.0.2.15 sending SYN packets
to 15 different ports on 10.0.2.20 in about half a second. The ports
were tried roughly in order, which suggests a deliberate scan. Only
three ports replied with a SYN-ACK: 22 (SSH), 80 (HTTP) and 3389 (RDP).
This is reconnaissance: the scanner learned which services were running.

### 3. Plaintext login
Filtering on `http` and following the HTTP stream showed a login sent to
an internal portal, `intranet-portal.info`, over plain HTTP. The
username and password were readable in the request, and the server
replied with a redirect to `/dashboard`, so the login worked.
(I have left the password out of this write-up.)

## My reasoning

**Most serious: the plaintext login.** It was a working login for an
internal system that anyone watching the network could read. The scan
only reveals targets, and I could not tell from the capture how much
data the DNS tunnelling carried out.

**OSI link.** HTTP is an Application layer (Layer 7) protocol that sends
everything as readable text. The missing encryption is the direct reason
the login was exposed.

**Would HTTPS have helped?** It would have stopped the plaintext login.
It would not have stopped the DNS tunnelling, because the lookup happens
before any HTTPS connection starts, or the port scan, because a scan
only checks whether ports answer.

## Recommendations

1. Use HTTPS on every internal login page, so credentials cannot be
   read on the network. Zero Trust ideas (verify every request, not
   just the network edge) would add protection over time.
2. Add an IDS rule that alerts on many connection attempts to different
   ports from one machine with no completed handshakes, the pattern the
   scan produced.
3. Most urgent: reset the password for the exposed account, because it
   may already have been seen.

## What is fact and what is my inference

The captures show what happened on the network. What the person behind
them intended is my inference. I did not decode the data inside the DNS
names.

## Tools

Kali Linux, Wireshark.

## How I worked

I did the investigation and wrote up my findings myself. I used an AI
assistant to explain concepts and to check my wording.

## Repo layout

```
README.md
```

All the data comes from a lab exercise, so none of these addresses or
accounts are real.
