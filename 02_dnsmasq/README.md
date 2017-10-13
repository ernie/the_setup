# The Setup for dnsmasq

## The Goal

In [the last step](../01_nginx/), we configured nginx to listen on ports 8080
and 8443 of our loopback interface. The only problem is that the TLDs we want
to use (`.devel` and `.staging`) are completely made up. We need to make
anything that ends in `.devel` or `.staging` point at our loopback, so that
the DNS names we set up for our SSL certificate will work, among other things.

## The Install

    brew install dnsmasq # You may need `sudo mkdir /usr/local/sbin` and chown
    cp dnsmasq.conf /usr/local/etc
    sudo brew services start dnsmasq

## The Configuration

There's not a ton to do on this step, really. The configuration is mostly
handled by the `dnsmasq.conf` we placed in `/usr/local/etc`. However, we do need
to tell OS X to use our dnsmasq server to resolve the TLDs we configured. That's
as easy as:

    sudo mkdir -p /etc/resolver
    echo nameserver 127.0.0.1 | \
      sudo tee /etc/resolver/devel /etc/resolver/staging

Now, when OS X tries to look up anything at either of these two TLDs, it'll use
the local nameserver provided by dnsmasq.

Onward to [pfctl configuration](../03_pfctl/)!
