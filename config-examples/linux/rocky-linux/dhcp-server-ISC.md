Part of getting DDNS (Dynamic DNS) working for FreeIPA.

This is for Rocky Linux, I did this on RL 9.3.

## Setup the ISC DHCP Server

1. Install the ISC DHCP Server

   `sudo dnf install dhcp-server`

1. Create a DHCP Server config, this is an example, modify as appropriate

    `-a` append, not clobber
    
    ```
    sudo mkdir /etc/dhcp
    sudo touch /etc/dhcp/dhcpd.conf
    cat <<EOF | sudo tee -a /etc/dhcp/dhcpd.conf > /dev/null
    
    default-lease-time 3600;
    max-lease-time 86400;
    authoritative;
    subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.200 192.168.1.253;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option domain-search "example.com";
    option domain-name-servers 192.168.1.1;
    }
    EOF
    ```

1. Enable the Server
 
   `sudo systemctl enable --now dhcpd`
   
1. Check the Status

   `sudo systemctl status dhcpd`

## Setup the access for the DDNS_KEY

1. Create a very specific looking file with a sha512 key called "DDNS_KEY"

   This is what it needs to look like, if we use *hmac-sha512*.

   ```
   key DDNS_UPDATE {
     algorithm hmac-sha512;
     secret <exactly-eight-six-charaters-go-here-no-more-no-less-needs-the-equals-signs-and-the-semi-colon>==;
   };
   ```

    This command rolls for a random key.
    
    `openssl rand -base64 64 | tr -d '\n'`
    
    This command rolls for a random key, and makes the file, and puts the key into the file at the same time.

   `key=$(openssl rand -base64 64 | tr -d '\n'); echo -e "key DDNS_UPDATE {\n  algorithm hmac-sha512;\n  secret $key;\n};" | sudo tee /etc/dhcp/ddns.key > /dev/null` 
   
1. Fix the permissions

   `sudo chmod 440 /etc/dhcp/ddns.key`

1. Append DDNS_UPDATE information to the DHCP config

    ```
    cat <<EOF | sudo tee -a /etc/dhcp/dhcpd.conf > /dev/null
    
    # this file includes definition of DDNS_UPDATE key
    include "/etc/dhcp/ddns.key";
    
    # replace xxxxxxx as appropriate
    zone xxxxxxx.com. {
      primary 127.0.0.1;
      key DDNS_UPDATE;
    }
    
    # replace 2.0.10 with your reverse ip
    zone 2.0.10.in-addr.arpa. {
      primary 127.0.0.1;
      key DDNS_UPDATE;
    }
    EOF
    ```

1. Restart DHCP

   ``sudo systemctl restart dhcpd`

# Resources
[Linux Techi - Configure DHCP Server on RHEL Rocky Linux](https://www.linuxtechi.com/configure-dhcp-server-on-rhel-rockylinux/)

[FreeIPA - ISC DHCPd and DDNS](https://www.freeipa.org/page/Howto/ISC_DHCPd_and_Dynamic_DNS_update)