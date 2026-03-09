
# 11. Domain Name System (DNS)

## Introduction to DNS

The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities. Most prominently, it translates more readily memorized domain names to the numerical IP addresses needed for locating and identifying computer services and devices with the underlying network protocols. By providing a worldwide, distributed directory service, the Domain Name System is an essential component of the functionality of the Internet.

## How DNS Works

When you type a domain name into your web browser, your computer first checks its local DNS cache to see if it already has the IP address for that domain name. If it does, it will use that IP address to connect to the website. If it does not, it will send a DNS query to a recursive DNS server.

The recursive DNS server will then send a query to a root DNS server. The root DNS server will respond with the IP address of a top-level domain (TLD) DNS server. The recursive DNS server will then send a query to the TLD DNS server. The TLD DNS server will respond with the IP address of an authoritative DNS server. The recursive DNS server will then send a query to the authoritative DNS server. The authoritative DNS server will respond with the IP address of the website. The recursive DNS server will then send the IP address of the website to your computer. Your computer will then use that IP address to connect to the website.

## DNS Hierarchy

The DNS hierarchy is a tree-like structure. At the top of the hierarchy are the root DNS servers. There are 13 root DNS servers in the world. The root DNS servers are responsible for the top-level domains (TLDs), such as .com, .org, and .net.

Below the root DNS servers are the TLD DNS servers. The TLD DNS servers are responsible for the second-level domains, such as google.com and wikipedia.org.

Below the TLD DNS servers are the authoritative DNS servers. The authoritative DNS servers are responsible for the individual domain names, such as www.google.com and en.wikipedia.org.

## DNS Records

DNS records are used to store information about a domain name. There are many different types of DNS records, including:

- **A Record:** An A record maps a domain name to an IPv4 address.
- **AAAA Record:** An AAAA record maps a domain name to an IPv6 address.
- **CNAME Record:** A CNAME record is an alias for another domain name.
- **MX Record:** An MX record specifies the mail server for a domain name.
- **NS Record:** An NS record specifies the name server for a domain name.
- **TXT Record:** A TXT record can be used to store any text-based information.

## DNS Caching

DNS caching is the process of storing DNS records locally. This can be done on your computer, on your router, or on your ISP's DNS server. DNS caching can improve the performance of DNS by reducing the number of DNS queries that need to be sent.
