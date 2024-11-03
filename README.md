# Vxlan-Control-Plane


![image](https://github.com/user-attachments/assets/9665d605-19c8-42dd-a8da-cf161d91d876)



# إعدادات أجهزة NXOS-1 و NXOS-2



## 1. إعدادات اسم الجهاز ( hostname )
ملاحظة : عندما تري اي امر يوجد امامه  NXOS-1 / NXOS-2  اعرف انه يوجد اختلاف بين الجهازين . اي بمعنى تسجله بجهاز الاول hostname NXOS-1 و تسجله بالجهاز الثاني hostname NXOS-2
### NXOS-1 و NXOS-2

```plaintext

hostname NXOS-1 / NXOS-2
```

## 2. تفعيل الميزات
```

nv overlay evpn
feature ospf
feature bgp
feature pim
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay
```
الوصف: تفعيل الميزات الأساسية مثل OSPF وBGP وPIM وميزات أخرى لدعم بنية الشبكة الافتراضية باستخدام EVPN وVXLAN.

## 3.إعدادات fabric forwarding
```plaintext

fabric forwarding anycast-gateway-mac 0000.2222.3333
```
الوصف: تفعيل خاصية anycast مما يسمح باستخدام عنوان MAC مشترك .

## 4. إعدادات VLAN و VN-segment
```plaintext
vlan 1,101-102,900,1000-1001
vlan 101
  vn-segment 900001
vlan 102
  vn-segment 900002
vlan 900
  vn-segment 5000
vlan 1000
  vn-segment 6000
vlan 1001
  vn-segment 5005
```
الوصف: تخصيص VLANs وربط كل منها بـ VN-segment للمساعدة في تشغيل VXLAN

##  4. إعداد VRF
```plaintext
vrf context Tenant-1
  vni 900001
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context Tenant-2
  vni 900002
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
```
الوصف: إعداد VRF لتمكين فصل حركة المرور بين جداول routing منفصله عن بعضها

##  5- إعداد واجهة NVE
```plaintext
interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback0
  member vni 5000
    suppress-arp
    ingress-replication protocol bgp
  member vni 5005
    ingress-replication protocol bgp
  member vni 6000
    ingress-replication protocol bgp
  member vni 900001 associate-vrf
  member vni 900002 associate-vrf
```
الوصف: إعداد nve1 لدعم EVPN وVXLAN من خلال استخدام BGP كوسيلة للتواصل مع الشبكة الافتراضية.

## 6. إعدادات المنافذ
```plaintext
interface Ethernet1/1
  switchport access vlan 1000 (على NXOS-1)
  switchport access vlan 1001 (على NXOS-2)

interface Ethernet1/2
  switchport access vlan 900

interface Ethernet1/3
  no switchport
  ip address 10.10.10.1/30 (على NXOS-1)
  ip address 10.10.10.2/30 (على NXOS-2)
  ip router ospf 10 area 0.0.0.0
  no shutdown
```
الوصف: تخصيص منافذ VLAN وإنشاء منفذ OSPF بين الجهازين.

##  7. إعداد واجهة Loopback
```plaintext
interface loopback0
  ip address 1.1.1.1/32 (على NXOS-1)
  ip address 2.2.2.2/32 (على NXOS-2)
  ip router ospf 10 area 0.0.0.0

```
الوصف: تخصيص واجهة Loopback، وهي مهمة كمعرف للجهازين في الشبكة عند استخدام BGP وOSPF.

## 8. إعداد BGP

```plaintext
router bgp 65535
  router-id 1.1.1.1 (على NXOS-1)
  router-id 2.2.2.2 (على NXOS-2)
  neighbor 2.2.2.2  (على NXOS-1)
  neighbor 1.1.1.1  (على NXOS-2)
    remote-as 65535
    update-source loopback0
    address-family l2vpn evpn
      send-community
      send-community extended
  vrf Tenant-2
    address-family ipv4 unicast
      advertise l2vpn evpn

```
الوصف: تكوين جلسات BGP مع جار باستخدام Loopback وتفعيل تبادل المعلومات بين VNIs باستخدام EVPN.

## 9. إعدادات EVPN
```plaintext
evpn
  vni 5000 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 5005 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 6000 l2
    rd auto
    route-target import auto
    route-target export auto

```
الوصف: إعداد VNIs داخل EVPN لتسهيل تبادل البيانات بين مختلف الشبكات الافتراضية.

## تمنياتي لكم بالتوفيق 
