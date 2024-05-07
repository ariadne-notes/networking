This setup enables Let's Encrypt SSL certificates without exposing the lab box to internet traffic, or needing to open port 80.

The license for this file is MIT, it can be merged into the official documentation.

These instructions will automate certificate renewal and there is verification at the bottom.

I used cloudflare, but it should work with other services.

1. Update the system

   `sudo apt update`

1. Install Snap

   `sudo apt install snapd`

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

1. Obtain the DNS Challenge cloudflare plugin
   
   `sudo snap install certbot-dns-cloudflare`
   
1. Re-establish connection to box, to refresh binary paths
   
   `<exit>`
   
   `<reconnect>` 
   
1. Get an API token from cloudflare, via the website
   
   * Limit permissions to `Zone - DNS - Edit`
   * Limit the Zone to `Include - Specific Zone - <domain>`

1. Create a `cloudflare.key` file with the API token

1. Make the directory, if it doesn't exist
   
   `sudo mkdir -p /etc/certbot/`

1. Make the file to contain the key
   
   `sudo nano /etc/certbot/cloudflare.key`
   
1. The file should contain ...
   
   `dns_cloudflare_api_token =  <put-key-here>`

1. Set the permissions on the key to be restrictive
   
   `sudo chmod o-rwx /etc/certbot/cloudflare.key`

1. Get the certificates
   
   ```
   sudo certbot certonly \
     --dns-cloudflare \
     --dns-cloudflare-credentials /opt/certbot/cloudflare.key \
     -d host.example.com
   ```
 
1. Stop Apache

`systemctl stop apache2`

1. Update Apache to use the new certs

```
CERT=$(find /etc/letsencrypt/live/ -name fullchain*)
KEY=$(find /etc/letsencrypt/live/ -name priv*)

sed -i --follow-symlinks /etc/apache2/sites-enabled/eveng-ssl.conf -Ee 's,(\s+SSLCertificateFile\s+).+,\1'$CERT',g'
sed -i --follow-symlinks /etc/apache2/sites-enabled/eveng-ssl.conf -Ee 's,(\s+SSLCertificateKeyFile\s+).+,\1'$KEY',g'
```

1. Restart Apache2

`systemctl start apache2`

1. Test

1. Verify Certbot has installed the renewal service

   `systemctl list-timers | grep certbot`



# References

[Install Certbot on Ubuntu](https://snapcraft.io/install/certbot/ubuntu)

[Read The Docs - Certbot - DNS Plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins)

[Enable SSL EVE Pro with Let's Encrypt](https://www.eve-ng.net/index.php/documentation/howtos/howto-enable-ssl-eve-pro-with-lets-encrypt/)