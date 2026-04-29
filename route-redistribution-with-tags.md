# Mutual Route-Redistribution

* Tag EIGRP as 100
* TAG OSPF as 1

- Route maps should take the form DENY -> PERMIT.
- Routes are tagged when they are advertised.

Route tags appear on-the-wire and can be read by other routers.
`ospf.lsa.asext.extrttag == 100`

In this example, EIGRP becomes a Type-5 OSPF update, with a route-tag of 100. If we look for these tags can exclude them in redistribution updates.

```
route-map ospf-into-eigrp deny 10
 description previously tagged EIGRP traffic
 match tag 100
!
route-map ospf-into-eigrp permit 20
 match source-protocol ospf 1 ospfv3 1
 set tag 1
!
route-map eigrp-into-ospf deny 10
 description previously tagged OSPF traffic
 match tag 1
!
route-map eigrp-into-ospf permit 20
 match source-protocol eigrp 100
 set tag 100
!
router eigrp 100
 redistribute ospf 1 metric 1000000 100 255 1 1500 route-map ospf-into-eigrp
!
router ospf 1
 redistribute eigrp 100 subnets route-map eigrp-into-ospf
```