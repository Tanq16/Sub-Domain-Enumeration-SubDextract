# Sub Directory Enumeration - SubDextract

This tool is meant to emulate the functionality of many available sub directory enumeration tools, with one of the best known ones being **Sublist3r**. However, there are two major classes of problems one can encounter with already available tools -

1. Long enumeration times.
2. Poorly formatted or short outputs (# subdomains).

To fix these problems, SubDextract uses a number of different functions/methods to enumerate the subdomains while also using strict regex matches to ensure only well-formatted outputs for ease of use. All functions are executed by different threads for max efficiency. For more details on each of the functions, refer below in their respective sections. The tool aims at striking a balance between enumeration time and the number and quality of results produced. It also introduces a webpage content dictionary method which may prove useful to many bug bounty hunters/red teamers in controlled or resticted local environments.

---

## Installation and Use

Installation is very simple - just install requirements and ensure required version of python.

> Tested and recommended for use on python version 3.6.9+

```bash
$ git clone https://github.com/Tanq16/Sub-Domain-Enumeration.git
Cloning into 'Sub-Domain-Enumeration'...
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 14 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (14/14), 6.05 KiB | 413.00 KiB/s, done.
$ cd 'Sub-Domain-Enumeration'
$ pip3 install -r requirements.txt
Collecting colorama==0.4.3 (from -r requirements.txt (line 1))
......
$ python sub_domain_enum.py domain.tld --save
```

The `--save` option allows the subdomains to be written line wise to a file which is named in the format *domain-tld*.

---

## List of functions/checks

1. CT logs server query
2. VirusTotal subdomain api query
3. ThreatCrowd subdomain api query
4. Subject Alternate Name query
5. Search engines (Yahoo, Netcraft) queries
6. DNS MX query
7. Web Content recursive dictionary level 2 breadth first regex match

Each of these functions/checks are used in parallel by the script on a given domain to search for the subdomains. Some servers (example VirusTotal) may not always respond or some checks (example SAN) may not always return a value. Required exception handling/script continuity constructs allow the script to return a significant subset of the subdomains despite one or two erroneous queries.

### **CT logs server query**

Crt.sh is a web interface to the distributed database of CT (certificate transparency) logs. CT is an internet security standard and open source framework for monitoring and auditing digital certificates, which creates a system of public logs which record all certificates issued by publicly trusted CAs (Certificate Authorities).

CT ensures that a certificate cannot be issued for a domain without the domain owner knowing. CT depends on verifiable CT logs. A log appends new certificate to an ever growing Merkle hash tree. CT monitors are clients to log servers which omnitor and check the correctness of the behavior of logs. CT auditors are also clients which use partial information about logs against other partial information they possess, to verify the logs.

The tool sends a query to the crt.sh web server and uses regular expressions to extract all well ormatted sub domains. It is subject to failing periodically if queried too many times.

> Reference: [Wikipedia - Certificate Transparency](https://en.wikipedia.org/wiki/Certificate_Transparency)

### **VirusTotal and TheratCrowd subdomain api queries**

VirusTotal maintains a mapping of DNS resolutions performed by it when executing samples submitted by users i.e., a passive DNS record replication. Subsequenty, subdomain resolutions are also stored.

The tool, like many others, uses the public api of VirusTotal to extract these subdomains. However, due to the implementation of the api, this is limited to the first 40 subdomains listed by the api. For load balancing and avaoiding a DoS, VirusTotal may reject requests if queried too many times.

The limitation of 40 results only does not pose a disadvantage because the other methods provide a significant addition to the list of subdomains.

Just like VirusTotal, ThreatCrowd also maintains data from different crawls it performs on the internet. The search API has DNS resolution data as well as a list of subdomains ThreatCrowd encountered, all in the form of a json.

> References: [VirusTotal Public API](https://virustotal.com/ui/domains/domain.tld/subdomains?limit=40), [ThreatCrowd API](https://github.com/AlienVault-OTX/ApiV2)

### **SAN and DNS mail server queries**

Subject Alternative Name (SAN) is an extension to the X.509 specification that allows users to specify additional host names for a single SSL certificate. Thus, SANs can be extracted from X.509 certificates to search for alternate names or subdomains.

DNS MX records can be queried to obtain the mail server names for the given domain, which again is an important vector to consider when enumerating subdomains.

> Reference: [SAN](https://en.wikipedia.org/wiki/Subject_Alternative_Name)

### **Search Engine queries**

Search engines such as Google, Bing and Yahoo index various web pages which are often served at particular subdomains for any given domain. Thus, extracting relevant information from the results page of Search Engines would prove really useful in providing subdomains. Dorks such as `site:` can be used in these search engines to gather results which have the input int their URL.

Two different search engines are used by the tool - Yahoo and Netcraft. Yahoo is a search engine like Google and Netcraft is an uptime monitoring company which also stores information such as OS detection results about its hosts. All result pages (<=10>) of Yahoo results are queried in parallel and all pages of Netcraft results (<=500 results) are matched against required regular expression compilations to extract subdomains.

Use of these web servers provide a major chunk of results. Not many online tools include these methods in their subdomain enumeration scripts. Sublist3r also querie these servers however, most of the times, the results are not well formatted due to complex regex used. This tool solves this issue as well.

> References: [Netcraft DNS Search](https://searchdns.netcraft.com/), [Yahoo Search](https://search.yahoo.com/)

### **Webpage Content dictionary**

This is a method that most of the available tools ignore. The method involves using the webpage of a given domain to search for regex matches and store them in a queue. The queue is then searched (breadth first) by matching the regex with contents of each entry to find further subdomains. This tool uses a 2 level BFS to find subdomains which proves to be sufficient to produce significant results, especially for domains meant for reaching web services. The code can be easily modified to use multiple levels of search.

This method is very useful in testing scenarios. For example, given a case where a Red Team must enumerate subdomains in an environment still under maintenance. If the internet is unreachable and the environment only consists of the local network, then finding subdomains by using only existing local DNS servers will be fruitless. Therefore, crawling content from the web services in the environment would help enuumerate a significant chunk of subdomains.

The method also proves useful in contrained enumeration when querying servers like netcraft, crt.sh, virustotal is infeasible. As an example, this method spat out a total 135 unique subdomains when run on domain `stanford.edu`. This method can be and should be used by bug bounty hunters for testing web services.

---

## Results

The tools uses many methods, each of which are succint in providing a significantly sized array of subdomains. However, when used in conjuction, as done by the tool, the result is a large list of subdomains in a very short amount of time. The time of the tool is managed by running each of the methods on separate threads. As an example, a complete run on the domain `stanford.edu` resulted in a total of 1714 subdomains in a short span of `50.30 seconds`. Unlike, many other tools available, this one does not have any formatting issue with the results.

## Miscellaneous

This section is for all the methods that could have been a part of the tool but aren't.

1. **More search engines** - When considering multiple search engines like Google, Yahoo, Bing, Ask, etc. it is important to note that most of the results will overlap given the fact that approximately 10 pages of results are crawled. The proportion of the overlap tends to be approximately 100% for domains which have more than 200 subdomains. Thus, to save time at the expense of losing tens of subdomains out of a few hundreds is  more efficient choice.
2. **DNS dumpster** is an online tool that tries to do the same function, but is limited in the functionality as a lot of information needs to be processed and rendered on a web interface on a public server. However, the results from DNS dumpster can also be crawled to expand the list of subdomains further. However, again the overlap is huge and to save time and limit to valid and useful results, this option hasn't been implemented.
3. **Multi-level traversal of content dictionary** - As stated in its section, a 2-level BFS proved to be a tight balance between time to search and number of results.
4. **VirusTotal data** - The premium (paid) API of VirusTotal provides access to much more than just 40 subdomains to be included in the json. This, if included in code, can significantly increase the number of results.
5. **More web databases of certificate logs** - Many web databses such as censys.io exist and could have also been a part of the tool. However, to maintain balance between number of subdomains and the time required to produce results, the possibility hasn't been considered.

---
