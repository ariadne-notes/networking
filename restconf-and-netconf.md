# Have a valid user with AAA new-model turned on


```
conf t
aaa new-model
aaa authentication login default local
aaa authorization exec default local
username admin privilege 15 secret cisco123
```

# Restconf

1. RESTCONF uses HTTP or HTTPS, so turn on the webserver
```
conf t
ip http secure-server
```

1. Turn on RESTCONF
```
conf t
restconf
```

1. Validate

RESTCONF relies on DMI and nginx
```
restconf-router# show platform software yang-management process
confd            : Running    
nesd             : Running    
syncfd           : Running    
ncsshd           : Running    
dmiauthd         : Running    
nginx            : Running    
ndbmand          : Running    
pubd             : Running  
```


## Get an IP Address

This is done from the linux commandline via curl

`--insecure` is added because Cisco generates it's own self-signed certificates.

```
ariadne@tesseract:~$ curl --insecure --user admin:cisco123 \
   -H "Accept: application/yang-data+json" \
   https://192.168.52.199/restconf/data/Cisco-IOS-XE-native:native/interface/Loopback=0

{
  "Cisco-IOS-XE-native:Loopback": {
    "name": 0,
    "ip": {
      "address": {
        "primary": {
          "address": "1.1.1.1",
          "mask": "255.255.255.255"
        }
      }
    }
  }
}
```


## Set an IP Address

Also done from the linux commandline via curl, just with a PATCH message.

```
ariadne@tesseract:~$ curl --insecure --user admin:cisco123 \
   -X PATCH \
   -H "Accept: application/yang-data+json" \
   -H "Content-Type: application/yang-data+json" \
   https://192.168.52.199/restconf/data/Cisco-IOS-XE-native:native/interface/Loopback=0 \
   -d '{
     "Cisco-IOS-XE-native:Loopback": {
       "name": 0,
       "ip": {
         "address": {
           "primary": {
             "address": "2.2.2.2",
             "mask": "255.255.255.255"
           }
         }
       }
     }
   }'

```



# Use NETCONF-YANG

1. Ensure a Valid user with AAA new-model is turned on, and available (see above)

1. Turn on NETCONF-YANG
```
conf t
netconf-yang
```

1. Validate
```
restconf-router#show netconf-yang status 
netconf-yang: enabled
netconf-yang ssh port: 830
netconf-yang candidate-datastore: disabled
```

I performed this lab inside a linux virtual environment.

1. Load a python virtual environment
```
python3 -m venv ~/netconf-lab
```

1. Activate it
```
source ~/netconf-lab/bin/activate
```

1. Install ncclient
```
pip install ncclient
```

1. Enter the python shell
```
python
```

1. Connect to device:
```
>>> conn = manager.connect(
    host="192.168.52.199",
    port=830,
    username="admin",
    password="cisco123",
    hostkey_verify=False,
    device_params={"name": "iosxe"}
)
```

1. Paste in a payload, follow the XML
```
>>> payload = """
<config>
  <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
    <interface>
      <Loopback>
        <name>5</name>
        <ip>
          <address>
            <primary>
              <address>5.5.5.5</address>
              <mask>255.255.255.255</mask>
            </primary>
          </address>
        </ip>
      </Loopback>
    </interface>
  </native>
</config>
"""
>>> conn.edit_config(target="running", config=payload)
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:5edcd8ca-3e51-4581-8bce-87f7eb939735" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0"><ok/></rpc-reply>
```

# Reference
[Programmability Configuration Guide, Cisco IOS XE 17.17.x](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1717/b_1717_programmability_cg/m_1717_prog_restconf.html)
