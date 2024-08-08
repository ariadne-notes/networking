network:
    version: 2
    ethernets:
        ens2:
            addresses:
                - 10.0.2.100/24
            macaddress: 8F:46:1E:7B:72:52
            routes:
                - to: default
                  via: 10.0.2.10
                  metric: 100
                  on-link: true