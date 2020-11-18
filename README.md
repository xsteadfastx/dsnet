dsnet is a simple configuration tool to manage a centralised wireguard VPN.
Think wg-quick but quicker. From scratch:

![dsnet add](https://raw.githubusercontent.com/naggie/dsnet/master/etc/init+add.png)

The server peer is listening, and a client peer config has been generated and
added to the server peer:

![wg](https://raw.githubusercontent.com/naggie/dsnet/master/etc/wg2.png)

More client peers can be added with `dsnet add`. They can connect immediately after!

It works on AMD64 based linux and also ARMv5.

    Usage: dsnet <cmd>

    Available commands:

    	init   : Create /etc/dsnetconfig.json containing default configuration + new keys without loading. Edit to taste.
    	add    : Add a new peer + sync
    	up     : Create the interface, run pre/post up, sync
    	report : Generate a JSON status report to the location configured in /etc/dsnetconfig.json.
    	remove : Remove a peer by hostname provided as argument + sync
    	down   : Destroy the interface, run pre/post down
    	sync   : Update wireguard configuration from /etc/dsnetconfig.json after validating


Quick start (AMD64 linux) -- install wireguard, then, after making sure `/usr/local/bin` is in your path:

    sudo wget https://github.com/naggie/dsnet/releases/download/v0.1/dsnet-linux-amd64 -O /usr/local/bin/dsnet
    sudo chmod +x /usr/local/bin/dsnet
    sudo dsnet init
    # edit /etc/dsnetconfig.json to taste
    sudo dsnet up
	sudo dsnet add banana > dsnet-banana.conf
	sudo dsnet add apple > dsnet-apple.conf

Copy the generated configuration file to your device and connect!

To send configurations, ffsend (with separately transferred password) or a
local QR code generator may be used.

The peer private key is generated on the server, which is technically not as
secure as generating it on the client peer and then providing the server the
public key; there is provision to specify a public key in the code when adding
a peer to avoid the server generating the private key. The feature will be
added when requested.

# Configuration overview

dsnetconfig.json is the only file the server needs to run the VPN. It contains
the server keys, peer public/shared keys and IP settings. **A working version is
automatically generated by `dsnet init` which can be modified as required.**

Currently its location is fixed as all my deployments are for a single network.
I may add a feature to allow setting of the location via environment variable
in the future to support multiple networks on a single host.

Main (automatically generated) configuration example:


    {
        "ExternalIP": "198.51.100.2",
        "ExternalIP6": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
        "ListenPort": 51820,
        "Domain": "dsnet",
        "InterfaceName": "dsnet",
        "Network": "10.164.236.0/22",
        "Network6": "fd00:7b31:106a:ae00::/64",
        "IP": "10.164.236.1",
        "IP6": "fd00:d631:74ca:7b00:a28:11a1:b821:f013",
        "DNS": "",
        "Networks": [],
        "ReportFile": "/var/lib/dsnetreport.json",
        "PrivateKey": "uC+xz3v1mfjWBHepwiCgAmPebZcY+EdhaHAvqX2r7U8=",
        "PostUp": "",
        "PostDown" "",
        "Peers": [
            {
                "Hostname": "test",
                "Owner": "naggie",
                "Description": "Home server",
                "IP": "10.164.236.2",
                "IP6": "fd00:7b31:106a:ae00:44c3:29c3:53b1:a6f9",
                "Added": "2020-05-07T10:04:46.336286992+01:00",
                "Networks": [],
                "PublicKey": "altJeQ/V52JZQrGcA9RiKcpZusYU6zMUJhl7Wbd9rX0=",
                "PresharedKey": "GcUtlze0BMuxo3iVEjpOahKdTf8xVfF8hDW3Ylw5az0="
            }
        ]
    }


See [CONFIG.md](CONFIG.md) for an explanation of each field.


# Report file overview

An example report file, generated by `dsnet report` to
`/var/lib/dsnetreport.json` by default:

    {
        "ExternalIP": "198.51.100.2",
        "InterfaceName": "dsnet",
        "ListenPort": 51820,
        "Domain": "dsnet",
        "IP": "10.164.236.1",
        "Network": "10.164.236.0/22",
        "DNS": "",
        "PeersOnline": 4,
        "PeersTotal": 13,
        "ReceiveBytes": 32517164,
        "TransmitBytes": 85384984,
        "ReceiveBytesSI": "32.5 MB",
        "TransmitBytesSI": "85.4 MB",
        "Peers": [
            {
                "Hostname": "test",
                "Owner": "naggie",
                "Description": "Home server",
                "Online": false,
                "Dormant": true,
                "Added": "2020-03-12T20:15:42.798800741Z",
                "IP": "10.164.236.2",
                "ExternalIP": "198.51.100.223",
                "Networks": [],
                "Added": "2020-05-07T10:04:46.336286992+01:00",
                "ReceiveBytes": 32517164,
                "TransmitBytes": 85384984,
                "ReceiveBytesSI": "32.5 MB",
                "TransmitBytesSI": "85.4 MB"
            }

            <...>
        ]
    }

Fields mean the same as they do above, or are self explanatory. Note that some
data is converted into human readable formats in addition to machine formats --
this is technically redundant but useful with Hugo shortcodes and other site generators.

The report can be converted, for instance, into a HTML table as below:

![dsnet report table](https://raw.githubusercontent.com/naggie/dsnet/master/etc/report.png)

See
[etc/README.md](https://github.com/naggie/dsnet/blob/master/contrib/report_rendering/README.md)
for hugo and PHP code for rendering a similar table.

# Generating other config files

dsnet currently supports the generation of `wg-quick` configuration by default.
It can also generate VyOS/Vyatta configuration for EdgeOS/Unifi devices such as
the Edgerouter 4 using the
[wireguard-vyatta](https://github.com/WireGuard/wireguard-vyatta-ubnt) package.

To change the config file format, set the following environment variables:

* `DSNET_OUTPUT=vyatta`
* `DSNET_OUTPUT=wg-quick`

Example vyatta output:

    configure
    set interfaces wireguard wg0 address 10.165.52.3/22
    set interfaces wireguard wg0 address fd00:7b31:106a:ae00:f7bb:bf31:201f:60ab/64
    set interfaces wireguard wg0 route-allowed-ips true
    set interfaces wireguard wg0 private-key cAtj1tbjGGmVoxdY78q9Sv0EgNlawbzffGWjajQkLFw=
    set interfaces wireguard wg0 description dsnet

    set interfaces wireguard wg0 peer PjxQM7OwVYvOJfORA1EluLw8CchSu7jLq92YYJi5ohY= endpoint 123.123.123.123:51820
    set interfaces wireguard wg0 peer PjxQM7OwVYvOJfORA1EluLw8CchSu7jLq92YYJi5ohY= persistent-keepalive 25
    set interfaces wireguard wg0 peer PjxQM7OwVYvOJfORA1EluLw8CchSu7jLq92YYJi5ohY= preshared-key w1FtOKoMEdnhsjREtSvpg1CHEKFzFzJWaQYZwaUCV38=
    set interfaces wireguard wg0 peer PjxQM7OwVYvOJfORA1EluLw8CchSu7jLq92YYJi5ohY= allowed-ips 10.165.52.0/22
    set interfaces wireguard wg0 peer PjxQM7OwVYvOJfORA1EluLw8CchSu7jLq92YYJi5ohY= allowed-ips fd00:7b31:106a:ae00::/64
    commit; save

Replace `wg0` with an unused interface name in the range `wg0-wg999`.

# FAQ

> Does dsnet support IPv6?

Yes! By default since version 0.2, a random ULA subnet is generated with a 0
subnet ID. Peers are allocated random addresses when added. Existing IPv4
configs will not be updated -- add a `Network6` subnet to the existing config
to allocate addresses to new peers.

Like IPv4, it's up to you if you want to provide NAT IPv6 access to the
internet; alternatively (and preferably) you can allocate a a real IPv6 subnet
such that all peers have a real globally routeable IPv6 address.

Upon initialisation, the server IPv4 and IPv6 external IP addresses are
discovered on a best-effort basis. Clients will have configuration configured
for the server IPv4 preferentially. If not IPv4 is configured, IPv6 is used;
this is to give the best chance of the VPN working regardless of the dodgy
network you're on.

> Is dsnet production ready?

Absolutely, it's just a configuration generator so your VPN does not depend on
dsnet after adding peers. I use it in production at 2 companies so far.

Note that before version 1.0, the config file schema may change. Changes will
be made clear in release notes.

> Why are there very few issues?

I'm tracking development elsewhere using
[dstask](https://github.com/naggie/dstask). I keep public initiated issues on
github though, and will probably migrate issues over if this gains use outside
of what I'm doing.

> Client private keys are generated on the server. Can I avoid this?

Allowing generation of the pub/priv keypair on the client is not yet supported,
but will be soon as provision exists within the code base. Note that whilst
client peer private keys are generated on the server, they are never stored.


> How do I get dsnet to bring the (server) interface up on startup?

Assuming you're running a systemd powered linux distribution (most of them are):

1. Copy
   [etc/dsnet.service](https://github.com/naggie/dsnet/blob/master/etc/dsnet.service)
   to `/etc/systemd/system/`
2. Run `sudo systemctl daemon-reload` to get systemd to see it
3. Then run `sudo systemctl enable dsnet` to enable it at boot

> How can I generate the report periodically?

Either with cron or a systemd timer. Cron is easiest:

    echo '* * * * * root /usr/local/bin/dsnet report | sudo tee /etc/cron.d/dsnetreport'

Note that whilst report generation requires root, consuming the report does not
as it's just a world-readable file. This is important for web interfaces that
need to be secure.

This is also why dsnet loads its configuration from a file -- it's possible to
set permissions such that dsnet synchronises the config generated by a non-root
user. Combined with a periodic `dsnet sync` like above, it's possible to build
a secure web interface that does not require root. A web interface is currently
being created by a friend; it will not be part of dstask, rather a separate
project.
