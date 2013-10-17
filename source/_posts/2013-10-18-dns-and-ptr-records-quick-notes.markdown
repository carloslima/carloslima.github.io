---
layout: post
title: "dns and ptr records quick notes"
date: 2013-10-18 03:18
comments: true
categories: note-to-self dns
---

Every once in a blue moon I get to troubleshoot, help someone with or explain how reverse dns works but somehow it keeps getting misplaced inside my brain.

So, for my future self, here it is:

As example, lets pick github's IP.
    [carlos@multi ~]$ host github.com
    github.com has address 192.30.252.129

The key thing to remember is that it is the same as forward dns, just on a "funny" zone name.

In our example, the reverse dns record is: `129.252.30.192.in-addr.arpa`  
(and the zone `252.30.192.in-addr.arpa`)

## Find the authority over that zone:
    [carlos@multi ~]$ dig 129.252.30.192.in-addr.arpa
    
    ; <<>> DiG 9.8.1-P1 <<>> 129.252.30.192.in-addr.arpa
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21414
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0
    
    ;; QUESTION SECTION:
    ;129.252.30.192.in-addr.arpa.   IN      A
    
    ;; AUTHORITY SECTION:
    252.30.192.in-addr.arpa. 60     IN      SOA     ns1.p16.dynect.net. ops.github.com. 7 3600 600 604800 60
    
    ;; Query time: 72 msec
    ;; SERVER: 127.0.1.1#53(127.0.1.1)
    ;; WHEN: Fri Oct 18 03:25:41 2013
    ;; MSG SIZE  rcvd: 113

## Get the PTR record for that ip:
    [carlos@multi ~]$ dig PTR 129.252.30.192.in-addr.arpa
    
    ; <<>> DiG 9.8.1-P1 <<>> PTR 129.252.30.192.in-addr.arpa
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29787
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
    
    ;; QUESTION SECTION:
    ;129.252.30.192.in-addr.arpa.	IN	PTR
    
    ;; ANSWER SECTION:
    129.252.30.192.in-addr.arpa. 3189 IN	PTR	ip1b-lb3-prd.iad.github.com.
    
    ;; Query time: 29 msec
    ;; SERVER: 127.0.1.1#53(127.0.1.1)
    ;; WHEN: Fri Oct 18 03:28:44 2013
    ;; MSG SIZE  rcvd: 86

### Directly from authority
And if you're trying to troubleshoot whether your ISP correctly updated the PTR records you requested, you can ask the authority directly and avoid waiting for propagation:

    [carlos@multi ~]$ dig PTR 129.252.30.192.in-addr.arpa @ns1.p16.dynect.net
    
    ; <<>> DiG 9.8.1-P1 <<>> PTR 129.252.30.192.in-addr.arpa @ns1.p16.dynect.net
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51495
    ;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 4, ADDITIONAL: 0
    ;; WARNING: recursion requested but not available
    
    ;; QUESTION SECTION:
    ;129.252.30.192.in-addr.arpa.   IN      PTR
    
    ;; ANSWER SECTION:
    129.252.30.192.in-addr.arpa. 3600 IN    PTR     ip1b-lb3-prd.iad.github.com.
    
    ;; AUTHORITY SECTION:
    252.30.192.in-addr.arpa. 86400  IN      NS      ns1.p16.dynect.net.
    252.30.192.in-addr.arpa. 86400  IN      NS      ns3.p16.dynect.net.
    252.30.192.in-addr.arpa. 86400  IN      NS      ns2.p16.dynect.net.
    252.30.192.in-addr.arpa. 86400  IN      NS      ns4.p16.dynect.net.
    
    ;; Query time: 476 msec
    ;; SERVER: 2001:500:90:1::16#53(2001:500:90:1::16)
    ;; WHEN: Fri Oct 18 03:31:39 2013
    ;; MSG SIZE  rcvd: 172
    
## Just simple resolution
Now, for completeness sake, if all you want is the reverse dns, remember that `host` works on both ip and name.

    [carlos@multi ~]$ host github.com
    github.com has address 192.30.252.130
    github.com mail is handled by 30 ASPMX3.GOOGLEMAIL.com.
    github.com mail is handled by 30 ASPMX2.GOOGLEMAIL.com.
    github.com mail is handled by 20 ALT1.ASPMX.L.GOOGLE.com.
    github.com mail is handled by 10 ASPMX.L.GOOGLE.com.
    github.com mail is handled by 20 ALT2.ASPMX.L.GOOGLE.com.
    [carlos@multi ~]$ host 192.30.252.130
    130.252.30.192.in-addr.arpa domain name pointer ip1c-lb3-prd.iad.github.com.
    [carlos@multi ~]$ 
    
