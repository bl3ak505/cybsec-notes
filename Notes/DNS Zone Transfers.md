
![[Pasted image 20260424080121.jpg]]

**_Quick Definition:_** _DNS Zone transfers are used by primary and secondary DNS servers to update zone files and records. There are two types of DNS zone transfers, full (AXFR) and incremental (IXFR), which are both prone to attack._ 

If you’ve ever used a subway system to travel, you know it's easier to plan your trip using a map than hopping on a random train and riding around until you arrive at your location. It’s also easier to remember a station name than the physical address for that station.

The [Domain Name System (DNS)](https://www.cbtnuggets.com/it-training/skills/microsoft-windows-server-2016-implement-domain-name-system-dns-70-743) works sort of like the subway map of the internet. Rather than keeping a list of IP addresses for every site you visit (or might want to visit!), you need to type the domain name, and the computer does the rest, guiding your traffic along the railway of the internet until your request reaches its destination. That’s the simple explanation, anyway.

In reality, DNS requests contain multiple steps to translate your device’s IP address, send it through various servers until it reaches the [IP address](https://www.cbtnuggets.com/blog/technology/networking/networking-basics-how-ip-and-mac-addresses-work) of the domain name you want to visit, and you can finally view your desired website.

This article discusses the types of DNS zone transfers, the benefits of each, and when we might use one instead of another. We’ll also discuss how DNS transfers work and how to configure them.

Finally, we'll look at how DNSSEC protects against zone transfer attacks, DNS zone transfer best practices, and how to troubleshoot DNS zone transfer issues.

## What are DNS Zone Transfers?

Now, let’s imagine you get to the subway station, and the screen showing the map is unavailable. How do you know where to go? That subway station likely has multiple map screens as a redundancy method.

Multiple screens ensure no map is overwhelmed with viewers, and additional maps are available if one is down. When updates are required, all the screens can display those updates simultaneously. 

DNS zone transfers work similarly by copying data from one DNS zone ([known as a zone file](https://www.cbtnuggets.com/blog/technology/system-admin/dns-records-explained-with-examples)) to another. This helps keep data accurate and up to date in case a DNS zone is unreachable.

![What-are-DNS-zone-transfers-Diagram](https://images.ctfassets.net/aoyx73g9h2pg/1PGX1tDUWI2LvZaMfcuLLD/b99767377630902316d2bd6421cfcb34/What-are-DNS-zone-transfers-Diagram.jpg "What-are-DNS-zone-transfers-Diagram")

Where our subway analogy begins to differ from proper DNS terminology is the sensitivity of data contained in a zone file. Taking a copy of the subway map doesn’t get you much information other than travel routes.

However, a copy of a DNS zone file may give you access to [IP addresses](https://www.cbtnuggets.com/it-training/skills/ip-addressing-subnetting-concepts) and hostnames of devices you shouldn’t necessarily have, as well as information about any mail server(s) in the DNS zone and other information an attacker might use with malicious intent. 

## Types of Zone Transfers

There are two types of zone transfers: full zone transfers (referred to as AXFR) and incremental zone transfers (referred to as IXFR). A full DNS zone transfer is exactly what it sounds like — a full copy of the DNS zone file. Incremental DNS zone transfers are also aptly named as they are a copy of the most recent changes to a zone file. 

Returning to our subway map analogy, imagine two stations were recently renamed. You could get a whole new map (AXFR) or write in the new station names on your old map. Either method successfully updates your map.

A full DNS zone transfer is a more time-consuming and bandwidth-intensive process. Unless most of the DNS records have changed, an incremental zone transfer is a more efficient option. You might also use a full zone transfer when adding additional DNS servers. 

An incremental zone transfer ensures you have the most up-to-date information faster, using less [network bandwidth](https://www.cbtnuggets.com/blog/technology/programming/network-throughput-vs-bandwidth-the-difference). Incremental zone transfers can also be used more frequently since they only update changed information. 

## DNS Zone Transfer Process

So, how does a DNS zone transfer work on a technical level? The process differs depending on whether a DNS zone has been replicated before or if this is the first request.

The process we'll look at involves a secondary server requesting a full zone transfer (AXFR) from a primary DNS server. 

[![Default Image Link](https://www.cbtnuggets.com/_next/image?url=%2Fassets%2F_next%2Fstatic%2Fc2271f7aad83500835d667b64ccf7d96.png&w=3840&q=75 "Default Image Link")](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/images/cc781340.d52bb70e-085f-4623-9fe3-9c3b17e7e47f\(ws.10\).gif)

The first step involves the secondary server sending an AXFR request to the primary DNS server, which is a request for the full DNS zone file. The primary DNS server then responds with the full DNS zone and a serial number associated with the version. 

This version contains the [Start of Authority (SOA)](https://www.cbtnuggets.com/blog/technology/networking/forward-vs-reverse-dns-lookup-zones-do-you-need-both) Resource Record (RR), which provides a specified refresh interval. This refresh interval lets the secondary server know when to request potential updates from the primary DNS server, usually around 15 minutes. 

Upon expiration of the refresh interval, the secondary server sends an SOA query to the primary server for updates. The primary server responds with the serial number of the latest SOA RR, which the secondary server compares with the last serial number it received. The secondary server will restart the refresh interval if the two serial numbers match.

If the two serial numbers do not match, the secondary server knows updates have occurred and will request an incremental DNS zone transfer, along with the serial number of its most recent zone file.

**The primary server will then inspect the supplied serial number and do one of two things:**

1. If the primary DNS server is capable of IXFR, the server will begin sending any changes between the two zone files to the secondary server.
    
2. If the primary DNS server cannot IXFR, it will respond with a full zone transfer.
    

This process will continue so long as the two servers can communicate to ensure DNS zone files are properly replicated across all DNS servers.

## Configuring Zone Transfers

Before initiating DNS zone transfers, you must set up at least two DNS servers: primary and secondary. [Configuring a DNS server](https://www.cbtnuggets.com/it-training/microsoft-windows-server/implement-dns) will look different depending on what systems you use.

Still, the main tasks involve choosing a server and installing server software (common in [Linux systems](https://www.cbtnuggets.com/it-training/linux)) or adding DNS server functionality from within Server Manager (common in Windows systems). For specific configuration instructions, consult user guides specific to your operating system.

Windows operating systems provide two options to configure DNS zones when configuring a primary zone: Active Directory (AD) and file-based. The first few steps are similar when configuring both. 

1. Navigate to the **Start** menu. 
    
2. Click on **Windows Administrative Tools**, then click **DNS**. 
    
3. Click on **New Zone**, then click **Next**. 
    
4. Click **Primary Zone**, and either ensure the box is checked or unchecked for the **Store the zone in Active Directory** option. 
    

There will be additional steps that will differ based on whether you want an AD-integrated DNS zone.

To configure a secondary zone in Windows Server: 

1. Navigate to the **Start** menu again, clicking on **Windows Administrative Tools**, then clicking **DNS**. 
    
2. Click on **New Zone** and **Next**. 
    
3. Click **Secondary Zone**, and specify the name for this zone. 
    

To configure DNS zone transfers, navigate to the **Start** menu again, click on **Windows Administrative Tools**, then click **DNS**. Expand the DNS server and DNS zone you wish to configure, then click on the **Zone Transfers** tab and either check or clear the box that says **Allow zone transfers**. If you want to allow zone transfers, you will need to specify one of the following options: 

- **To any server**
    
- **Only to servers listed on the name servers tab**
    
- **Only to the following servers**
    

The [most secure option](https://www.cbtnuggets.com/it-training/skills/implement-dns-zones-and-security) would be to select **only the following servers** since you want to ensure only specific devices can access zone files. You'll specify the IP address(es) for the server(s) you want to be able to perform zone transfers. You can implement other security features, such as TSIG (transaction signature) and DNSSEC (domain name system security extensions). 

TSIG provides a method of [authentication](https://www.cbtnuggets.com/blog/technology/system-admin/3-ways-to-authenticate-a-user-beyond-a-password) between two DNS servers via a shared secret. Using TSIG means packets shared between primary and secondary DNS servers are cryptographically signed, reducing the likelihood of a malicious device getting in the way of the transaction. In addition to authentication, TSIG timestamps all messages, further reducing the risk of packets being intercepted and altered. 

## DNSSEC (DNS Security Extensions) and Zone Transfers

DNSSEC is another method of authentication. The main risks DNSSEC protects against are DNS forgery and DNS cache poisoning. By signing all communication, DNSSEC ensures that forged messages from malicious devices are easily identified. Both TSIG and DNSSEC protect the integrity of data, not confidentiality. 

TSIG and DNSSEC serve different purposes. TSIG protects the integrity of traffic between primary and secondary DNS servers. In contrast, DNSSEC protects the integrity of communication between DNS servers and other devices using DNS. 

## Troubleshooting Zone Transfer Issues

As with any technology, DNS servers sometimes require [troubleshooting](https://www.cbtnuggets.com/blog/technology/networking/how-to-troubleshoot-external-network-issues). Some of the more common issues will center around one server being unable to communicate with another due to authoritative or network issues. 

### Check Server Logs

If a zone transfer fails, check the server logs for any error messages which may indicate the issue. If you cannot determine the cause using logs, move on to additional troubleshooting steps. 

### Verify Zone Transfer Settings

Check zone transfer settings on your primary DNS server. Is the IP address of the secondary server authorized to request zone transfers? 

### Review TTL Settings 

Did the TTL (time to live) time out on one of the servers? TTL is the amount of time a request is valid and able to elicit a response, so if the TTL expired, the subsequent information would never be sent... If the TTL did expire, check for network latency issues between devices. 

## Zone Transfers In a Multi-Server Environment

Depending on the scale of your organization, you likely use multiple authoritative DNS servers. When managing multiple authoritative [DNS servers](https://www.cbtnuggets.com/it-training/skills/install-configure-dns-servers), ensuring they are configured with optimization and security in mind is important. 

If one server becomes unavailable, you need other servers to respond to the additional requests. Depending on your DNS setup, there may be native tools that allow you to manage multiple authoritative DNS servers, and there are third-party tools to help. 

## Conclusion

DNS zone transfers are crucial to keeping records updated and traffic flowing smoothly. A DNS zone transfer is when a primary DNS server sends some or all of its [zone files](https://www.cbtnuggets.com/it-training/skills/implement-dns-zones-and-security) to a secondary server.

As mentioned, these are known as incremental zone transfers (IXFR) and full zone transfers (AFXR). DNS zone transfers are a prime target for cyber-attacks because of the type and amount of information that can be gained. 

Thankfully, there are security features and best practices that reduce the risk of attacks. First, you can configure primary DNS servers to only communicate with secondary servers based on their IP addresses.

You can also implement TSIG and DNSSEC. TSIG helps protect the integrity of traffic between DNS servers, and DNSSEC helps protect data integrity between a DNS server and other devices.
