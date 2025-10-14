# Network

* The CLI parser is converting the IP into binary, then comparing it to the wild mask.
* The CLI parser will only save the matched bits of the IP.
* The CLI parser will not save the zeroth network, anything starting with 0.
* The CLI parser will only save the matched bits of an IP if if finds bits that are "on"
* Using the "all" mask of 255.255.255.255 creates this statement 'network 0.0.0.0' and matches everything.
* Using the "unique-ip" mask of 0.0.0.0 means "match this single address"
* The wildcard mask only accepts contiguous numbers "Discontiguous mask is not supported."

192.0.2.5 127.255.255.255 - becomes 128.0.0.0, the rest of the bits get dropped.