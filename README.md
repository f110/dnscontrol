# DNSControl

[![Build Status](https://travis-ci.org/StackExchange/dnscontrol.svg?branch=master)](https://travis-ci.org/StackExchange/dnscontrol)
[![Gitter chat](https://badges.gitter.im/dnscontrol/Lobby.png)](https://gitter.im/dnscontrol/Lobby)
[![Google Group chat](https://img.shields.io/badge/google%20group-chat-green.svg)](https://groups.google.com/forum/#!forum/dnscontrol-discuss)

DNSControl is a system for maintaining DNS zones.  It has two parts:
a domain specific language (DSL) for describing DNS zones plus
software that processes the DSL and pushes the resulting zones to
DNS providers such as Route53, CloudFlare, and Gandi.  It can talk
to Microsoft ActiveDirectory and it generates the most beautiful
BIND zone files ever.  It runs anywhere Go runs (Linux, macOS,
Windows). The provider model is extensible, so more providers can be added.

Currently supported DNS providers:
 - Active Directory
 - BIND
 - CloudFlare
 - Digitalocean
 - DNSimple
 - Gandi
 - Google
 - HEXONET
 - Linode
 - Namecheap
 - Name.com
 - NS1
 - Route 53
 - SoftLayer
 - Vultr
 - OVH

At Stack Overflow, we use this system to manage hundreds of domains
and subdomains across multiple registrars and DNS providers.

You can think of it as a DNS compiler.  The configuration files are
written in a DSL that looks a lot like JavaScript.  It is compiled
to an intermediate representation (IR).  Compiler back-ends use the
IR to update your DNS zones on services such as Route53, CloudFlare,
and Gandi, or systems such as BIND and ActiveDirectory.

# An Example

`dnsconfig.js`:

```js
// define our registrar and providers
var namecom = NewRegistrar("name.com", "NAMEDOTCOM");
var r53 = NewDnsProvider("r53", "ROUTE53")

D("example.com", namecom, DnsProvider(r53),
  A("@", "1.2.3.4"),
  CNAME("www","@"),
  MX("@",5,"mail.myserver.com."),
  A("test", "5.6.7.8")
)
```

Running `dnscontrol preview` will talk to the providers (here name.com as registrar and route 53 as the dns host), and determine what changes need to be made.

Running `dnscontrol push` will make those changes with the provider and my dns records will be correctly updated.

See [Getting Started](https://stackexchange.github.io/dnscontrol/getting-started) page on documentation site.

# Benefits

* Editing zone files is error-prone.  Clicking buttons on a web
page is irreproducible.
* Switching DNS providers becomes a no-brainer.  The DNSControl
language is vendor-agnostic.  If you use it to maintain your DNS
zone records, you can switch between DNS providers easily. In fact,
DNSControl will upload your DNS records to multiple providers, which
means you can test one while switching to another. We've switched
providers 3 times in three years and we've never lost a DNS record.
* Adopt CI/CD principles to DNS!  At StackOverflow we maintain our
DNSControl configurations in Git and use our CI system to roll out
changes.  Keeping DNS information in a VCS means we have full
history.  Using CI enables us to include unit-tests and system-tests.
Remember when you forgot to include a "." at the end of an MX record?
We haven't had that problem since we included a test to make sure
Tom doesn't make that mistake... again.
* Variables save time!  Assign an IP address to a constant and use
the variable name throughout the file. Need to change the IP address
globally? Just change the variable and "recompile."
* Macros!  Define your SPF records, MX records, or other repeated
data once and re-use them for all domains.
* Control CloudFlare from a single location.  Enable/disable
Cloudflare proxying (the "orange cloud" button) directly from your
DNSControl files.
* Keep similar domains in sync with transforms and other features.
If one domain is supposed to be the same
* It is extendable!  All the DNS providers are written as plugins.
Writing new plugins is very easy.

# Installation

## From source

DNSControl can be built with Go version 1.7 or higher. To install, simply run 

`go get github.com/StackExchange/dnscontrol`

dnscontrol should be installed in $GOPATH/bin

## Via packages

Get prebuilt binaries from [github releases](https://github.com/StackExchange/dnscontrol/releases/latest)

## Via [docker](https://hub.docker.com/r/stackexchange/dnscontrol/)

```
docker run --rm -it -v $(pwd)/dnsconfig.js:/dns/dnsconfig.js -v $(pwd)/creds.json:/dns/creds.json stackexchange/dnscontrol dnscontrol preview
```
