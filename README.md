# Pihole + Unbound - DNS Stack

<p align="center">
    <a href="https://pi-hole.net/">
      <img src="https://pi-hole.github.io/graphics/Vortex/Vortex_with_Wordmark.svg" width="150" height="260" alt="Pi-hole">
    </a>
    <br>
    <a href="">
      <img src="https://www.docker.com/sites/default/files/d8/styles/role_icon/public/2019-07/Docker-Logo-White-RGB_Vertical-BG_0.png" width="auto" height="150" alt="Unbound">
    </a>
    <br>
    <a href="">
      <img src="https://nlnetlabs.nl/static/logos/Unbound/Unbound_FC_Shaded_cropped.svg" width="auto" height="150" alt="Unbound">
    </a>
</p>

## Why, How & What

### Why

[DNS Security](https://www.cloudflare.com/learning/dns/dns-security/) is the practice of protecting DNS infrastructure from cyberattacks in order to keep it performing quickly and reliably.
Most of the DNS queries in the internet are **unencrypted**. It means that your network administrator or ISP provider can see exactly what websites you visit every day. Usually, only one DNS resolver is used so the provider knows all domains that you visit.
Security & Privacy might be at risk. Do you trust your ISP or DNS provider?

### How

We are going block malicious websites & ads by predefined blocklists and also cache DNS entries for faster lookups in the future.

### What

In order to accomplish our goals, we use:
- [Pihole](https://github.com/pi-hole/pi-hole) is a [DNS sinkhole](https://en.wikipedia.org/wiki/DNS_sinkhole) that protects your devices from unwanted content without installing any client-side software. It block malicious websites and ads.
- [Unbound](https://github.com/NLnetLabs/unbound) is a validating, recursive, and caching DNS resolver.

## Performance
```shell
            test1   test2   test3   test4   test5   test6   test7   test8   test9   test10  Average
pihole      2 ms    3 ms    2 ms    3 ms    2 ms    2 ms    3 ms    2 ms    2 ms    2 ms      2.30
quad9       12 ms   12 ms   11 ms   12 ms   11 ms   10 ms   11 ms   11 ms   11 ms   11 ms     11.20
google      11 ms   11 ms   11 ms   13 ms   11 ms   12 ms   11 ms   11 ms   11 ms   11 ms     11.30
cloudflare  22 ms   22 ms   22 ms   22 ms   22 ms   22 ms   20 ms   30 ms   34 ms   20 ms     23.60
```
## Support

Leave a ⭐ if you find this project helpful!

<a href="https://www.buymeacoffee.com/tjay.dev" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-red.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

---

## Prerequisites

### Docker & Docker Compose installed
```shell
docker -v
docker-compose -v
```

### Clone the repo
```shell
git clone https://github.com/tomajask/pihole-unbound-dns-stack.git
cd pihole-unbound-dns-stack
```

## Setting up Environmental Variables and config files

### Generate a password for Pihole Admin panel:
```shell
cp .env.example .env
echo "WEBPASSWORD=`openssl rand -base64 32`" >> .env
chmod 600 .env
```

### Generate a log file for pihole:
```shell
mkdir -p ./pihole
touch ./pihole/pihole.log
```

### Copy Unbound config files:
```shell
mkdir -p ./unbound
cp unbound-config-examples/a-records.example.conf ./unbound/a-records.conf
cp unbound-config-examples/srv-records.example.conf ./unbound/srv-records.conf
cp unbound-config-examples/unbound.example.conf ./unbound/unbound.conf
```
There are 2 options for Unbound:
1. **Recursive mode** - Unbound will act as a Recursive DNS server, the DNS traffic will not be encrypted
```shell
touch ./unbound/forward-records.conf
```
2. **Fowarding mode** - Unbound will act as a forwarding DNS server, it will use DNS over TLS when querying upstream DNS servers
```shell
cp unbound-config-examples/forward-records.example.conf ./unbound/forward-records.conf
```
If you are not sure which one to use, see discussions: [here](https://discourse.pi-hole.net/t/recursive-dns-server-unbound-or-dns-over-https/11890/9), [here](https://discourse.pi-hole.net/t/unbound-using-tls-not-working-as-recursive-dns-server-anymore/31796), [here](https://www.netmeister.org/blog/doh-dot-dnssec.html)
### Check whether port `53` if available:
```shell
sudo lsof -i -P -n | grep ':53 (LISTEN)'
```
If the result is empty then it's ready to go. If not, then most probably it's `systemd-resolve`'s DNSStubListener running.
To disable the `DNSStubListener` & `Cache`, run:
```shell
sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf
sudo sed -r -i.orig 's/#?Cache=yes/Cache=no/g' /etc/systemd/resolved.conf
```
Restart `systemd-resolve`:
```shell
sudo systemctl restart systemd-resolved
sudo systemctl status systemd-resolved
```
Check whether port `53` is freed now:
```shell
sudo lsof -i -P -n | grep ':53 (LISTEN)'
```

## Installation

```shell
docker-compose up -d
```

You should be able to access now the Pihole Admin panel on `<ip_address_of_the_server>/admin` (e.g. http://192.168.1.20/admin).


## Advanced

### Additional blocklists

The default list of blocked domain provided by Pihole is quite small. If you'd like to add more domains, then visit https://firebog.net/ & https://phishing.army/ and add them in the Pihole Admin panel: `Panel -> Group Management -> Adlists -> Add a new addlist`.

### Custom Upstream DNS Resolvers

By default, as upstream DNS resolvers for Unbound are used Cloudflare's and Quad 9's server. They can be adjusted in `forward-records.conf` config file.

### Local DNS entries

If you need to setup some local DNS entries, you can add them in Pihole or Unbound:
- Pihole: `Panel -> Local DNS -> DNS Records` or `Panel -> Local DNS -> CNAME Records`
- Unbound: add new entries to `a-records.conf` config file.

### DNS over TLS

By default, Unbound is configured to use DNS over TLS for upstream DNS resolvers. See `forward-records.conf` config file.
In order to check whether DoT (DNS over TLS) is enabled, you can visit https://1.1.1.1/help.

### Performance

If you are curious how performant your Pihole with Unbound is you can test it with [dnsperftest](https://github.com/cleanbrowsing/dnsperftest).

To run the test following [instructions](https://github.com/cleanbrowsing/dnsperftest#utilization) or:
- install prerequisites
```shell
sudo apt install bc dnsutils -y
```
- clone the repo
```shell
git clone --depth=1 https://github.com/cleanbrowsing/dnsperftest/
cd dnsperftest
```
- insert your Pihole's instance IP address for tests
```shell
sed -i '/^PROVIDERS="/a 192.168.1.20#pihole' dnstest.sh
```
- run the test a few times to allow cache to build itself
```shell
./dnstest.sh
./dnstest.sh
./dnstest.sh | sort -k 22 -n
```
Ideally, `pihole` should result on average in a very low latency

### Troubleshooting

#### Debugging Unbound via logs

In order to see how logs should look like, you can visit https://unboundtest.com and test a domain.
To achieve the same log level in this stack you need to adjust the `verbosity` level for unbound:
```shell
sed -i 's/verbosity: \d/verbosity: 2/g' ./unbound/unbound.conf
```
Resolve a domain (replace the IP of the Pihole, adjust the port if needed):
```shell
dig github.com @192.168.1.20 -p 53
```

#### Slow performance

See: https://www.reddit.com/r/pihole/comments/d9j1z6/unbound_as_recursive_dns_server_slow_performance/
