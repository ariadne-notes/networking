# Rocky Linux, Certbot, Let's Encrypt, DNS and Snap

This setup means a device can have a valid SSL certificate and still be inaccessible from the Internet, so `https://host.example.com` works internally without SSL warnings.

[Let's Encrypt](https://letsencrypt.org/about/) is a [Certificate Authority](https://en.wikipedia.org/wiki/Certificate_authority#Providers) provided by the non-profit [Internet Security Research Group](https://www.abetterinternet.org/) as a free service.

This is a partial set of instructions to get valid SSL certificates via Let's Encrypt via certbot. It doesn't include autorenew. I did this on Rocky Linux but other instructions exist for [other platforms.](https://snapcraft.io/docs/installing-snapd)

These instructions follow [RFC 8555#section-8.4](https://datatracker.ietf.org/doc/html/rfc8555#section-8.4) -> **DNS Challenge**.

I'm using cloudflare with a domain I own, but there is a good sized list of [supported DNS plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins).



## Instructions
1. Remove the older certbot
   
   `sudo dnf remove certbot`

1. Update the package list
   
   `sudo dnf update`

1. Install the EPEL repository
   
   `sudo dnf install epel-release`

1. Install snapd, via the EPEL repository
   
   `sudo dnf install snapd`

1. Enable the snap socket
   
   `sudo systemctl enable --now snapd.socket`

1. Enable Classic Snap
   
   `sudo ln -s /var/lib/snapd/snap /snap`

1. Install Classic Certbot, via Snap
   
   `sudo snap install --classic certbot`

1. Link it like a regular binary.
   
   `sudo ln -s /snap/bin/certbot /usr/bin/certbot`

1. Tell Certbot it can have root
   
   `sudo snap set certbot trust-plugin-with-root=ok`

1. Obtain the cloudflare plugin
   
   `sudo snap install certbot-dns-cloudflare`

1. Re-establish connection to box, to refresh binary paths
   
   `<exit>`
   
   `<reconnect>`

1. Get an API token from cloudflare.
   
   * Limit permissions to `Zone - DNS - Edit`
   * Limit the Zone to `Include - Specific Zone - <domain>`

1. Create a `cloudflare.key` file with the API token
   
   `dns_cloudflare_api_token =  <token here>`

1. Set the permissions on the key to be restrictive
   
   `sudo chmod o-rwx cloudflare.key`

1. Get the certificates
   
   ```
   sudo certbot certonly \
     --dns-cloudflare \
     --dns-cloudflare-credentials /opt/certbot/cloudflare.key \
     -d host.example.com
   ```

1. Move `cloudflare.key` into the new `/etc/letsencrypt/` directory.
   
   `sudo mv /etc/letsencrypt/cloudflare-api-key cloudflare.key`

1. Check work
   
   `ls -la /etc/letsencrypt/`

# References
[EFF - Install Certbot via Snap](https://certbot.eff.org/instructions?ws=other&os=snap&tab=standard)

[Snapcraft - Installing Snap or Rocky Linux](https://snapcraft.io/docs/installing-snap-on-rocky)

[Read The Docs - Certbot - DNS Plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins)

