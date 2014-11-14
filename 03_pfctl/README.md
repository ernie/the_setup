# The Setup for pfctl

## The Goal

So, now we have a [web server](../01_nginx/) and [DNS](../02_dnsmasq/) all set
up, but we still have a problem. We run nginx as a regular user, which means it
binds to unprivileged ports. We want to be able to go to URLs like
http://my-app.local.devel and https://my-app.local.devel without appending a
port number to the URL, and without changing nginx from its default Homebrew
unprivileged port setup. Thankfully, OS X has a packet filter that can take care
of this for us. The `pfctl` command is the one that manages this packet filter.

## The Installation

1. There is no step one. `pfctl` is already installed!

## The Configuration

The BSD packet filter has some really powerful configuration capabilities, but a
correspondingly large number of concepts with which to familiarize ourselves.
Thankfully, we aren't doing anything too complex.

The basic idea is that we want to define an "anchor" (BSD packet filter
terminology for a named set of rules) called "devel.local" in our `pf.conf` file
and set up our rules inside that anchor, then ensure that the packet filter
gets loaded on startup.

We can completely cargo-cult the default "com.apple" anchor's format in our
configuration file. Feel free to copy the [pf.conf](pf.conf) from this
repository...

    sudo cp pf.conf /etc/pf.conf

If you'd like to do this manually, it's simply copying each line in the
existing com.apple anchor point, replacing com.apple with devel.local. Note that
these lines are order-dependent. While you might like to create a second section
commented as "devel.local anchor point" after the "com.apple anchor point"
section, you can't. `pfctl` will complain when you run it.

Then, copy the anchor contents:

    sudo cp pf.anchors/* /etc/pf.anchors/

The interesting lines are in `devel.local.forwarding`, which consists of:

```
rdr pass on lo0 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080
rdr pass on lo0 inet proto tcp from any to any port 443 -> 127.0.0.1 port 8443
```

Here we set up forwarding for the loopback interface (lo0), which handles
traffic to 127.0.0.1, so that any TCP traffic to port 8080 gets forwarded to
port 80 instead, and any traffic to port 8443 gets forwarded to port 443.

That's all we need to do! Now, to make sure it gets activated right now (and at
each startup):

    sudo cp devel.local.pfctl.plist /Library/LaunchDaemons/
    sudo chown root:wheel /Library/LaunchDaemons/devel.local.pfctl.plist
    sudo launchctl load /Library/LaunchDaemons/devel.local.pfctl.plist

This will wait for network services to complete their configuration, then run
`/sbin/pfctl -e -f /etc/pf.conf`, which starts up the packet filter with our
rules.

At this point, if you've been following along with the guide until this point,
you should be able to start up a (for instance) Rails app listening on port
3000, and access it at http://myapp.devel.local *and* https://myapp.devel.local!

Onward to [Foreman setup](../04_foreman/)!
