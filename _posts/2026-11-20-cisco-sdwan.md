---
title: Cisco Catalyst SD-WAN
date: 2026-11-20 20:30:00 +0700
categories: [Software Defined Networking, Cisco SD-WAN]
tags: [SD-WAN]
---

**Cisco Catalyst SD-WAN** (formerly Cisco SD-WAN / Viptela) is an enterprise-grade Software-Defined Wide Area Network architecture designed to decouple network hardware from control mechanisms, centralize orchestration, and dynamically optimize application traffic across diverse transport links (MPLS, broadband Internet, LTE/5G).

The architecture separates network functionality into four discrete planes:

1. **Management Plane (Catalyst SD-WAN Manager / vManage)**: Centralized single-pane-of-glass Network Management System (NMS) providing multi-tenant GUI, configuration management via feature/device templates, real-time monitoring, telemetry, troubleshooting, and software upgrades.
2. **Orchestration Plane (Catalyst SD-WAN Validator / vBond)**: The gatekeeper and orchestrator of the entire SD-WAN fabric. It authenticates and authorizes all incoming controllers and WAN edge routers via PKI certificates and whitelist serial/chassis numbers, facilitates NAT traversal, and distributes initial control-plane connection information.
3. **Control Plane (Catalyst SD-WAN Controller / vSmart)**: The brain of the SD-WAN overlay. It establishes secure DTLS/TLS control connections with WAN edge routers and utilizes the **Overlay Management Protocol (OMP)** to exchange routing reachability, transport locators (TLOCs), encryption keys, and centralized data/routing policies without requiring a full mesh of BGP/OSPF peerings across the edge.
4. **Data Plane (Catalyst SD-WAN WAN Edge / cEdge & vEdge)**: Physical or virtual appliances (such as the Cisco Catalyst 8000V Edge Software Router) deployed at branch offices, campuses, and data centers. They establish auto-keyed IPsec data tunnels between one another across underlay transports based on control policies received from vSmart.

In this comprehensive deployment guide, we build a multi-site Cisco Catalyst SD-WAN fabric from scratch: deploying controllers, establishing an Enterprise CA PKI trust model, onboarding WAN Edge routers (Catalyst 8000V) via Cisco Smart Account / Plug and Play (PnP), creating modular Feature and Device Templates, validating OMP routing and BFD session establishment, and configuring Direct Internet Access (DIA) with NAT and inter-VPN route leaking.

---

## Topology & Architecture

Our lab fabric consists of three control plane nodes residing in the controller subnet, two transport underlays representing dual-ISP connectivity, and multiple branch sites running Cisco Catalyst 8000V virtual edge routers:

- **Control Plane**:
  - **vManage (Manager)**: System IP `1.1.1.1`, Site ID `1`, VPN 0 IP `172.16.0.91/24`, VPN 512 Mgmt IP `198.18.129.91/18`
  - **vBond (Validator)**: System IP `1.1.1.2`, Site ID `1`, VPN 0 IP `172.16.0.92/24`, VPN 512 Mgmt IP `198.18.129.92/18`
  - **vSmart (Controller)**: System IP `1.1.1.3`, Site ID `1`, VPN 0 IP `172.16.0.93/24`, VPN 512 Mgmt IP `198.18.129.93/18`
- **Transport Underlays (WAN Transports)**:
  - **`biz-internet`**: Subnet `172.17.0.0/24` (Gateway: `172.17.0.1`)
  - **`public-internet`**: Subnet `172.18.0.0/24` (Gateway: `172.18.0.1`)
- **WAN Edge Sites**:
  - **Site 11 (Jakarta - `c8k-11-jkt`)**: System IP `10.11.1.1`, Site ID `11`, WAN1 (Gig1) `172.17.0.11/24`, WAN2 (Gig2) `172.18.0.11/24`, Service LAN (VPN 1 Gig3) `10.11.0.1/24`
  - **Site 12 (Surabaya - `c8k-12-sby`)**: System IP `10.12.1.1`, Site ID `12`, WAN1 (Gig1) `172.17.0.12/24`, WAN2 (Gig2) `172.18.0.12/24`, Service LAN (VPN 1 Gig3) `10.12.0.1/24`
  - **Site 13 (Denpasar - `c8k-13-dps`)**: System IP `10.13.1.1`, Site ID `13`, WAN1 (Gig1) `172.17.0.13/24`, WAN2 (Gig2) `172.18.0.13/24`, Service LAN (VPN 1 Gig3) `10.13.0.1/24`

![x](/static/2026-11-20-cisco-sdwan/00.png)

<br>

---

## Installation & Bootstrap Configuration

We prepare the virtual appliance installation OVA images for **Catalyst SD-WAN Manager** (`viptela-vmanage-20.18.5-genericx86-64.ova`), **Catalyst SD-WAN Validator** (`viptela-bond-20.18.5-genericx86-64.ova`), and **Catalyst SD-WAN Controller** (`viptela-smart-20.18.5-genericx86-64.ova`).

![x](/static/2026-11-20-cisco-sdwan/01.png)

<br>

---

Upon powering on each controller appliance, we access the console using the default credentials (`admin`/`admin`). The system prompts us to establish a secure initial administrator password and select the vManage operating persona (`1) COMPUTE_AND_DATA`).

![x](/static/2026-11-20-cisco-sdwan/02.png)

<br>

---

Next, we configure the base bootstrap parameters on each controller node via CLI.

### vManage Bootstrap Configuration

```
config terminal
 system
  host-name vmanage
  system-ip 1.1.1.1
  site-id 1
  organization-name helena
  vbond 172.16.0.92
  ntp server 172.16.0.1
   version 4
  exit
 !
 vpn 0
  interface eth0
   ip address 172.16.0.91/24
   no shutdown
   tunnel-interface
    allow-service all
   exit
  exit
  ip route 0.0.0.0/0 172.16.0.1
 !
 vpn 512
  interface eth1
   ip address 198.18.129.91/18
   no shutdown
  exit
  ip route 0.0.0.0/0 198.18.128.1
 !
commit and-quit
```

### vBond Bootstrap Configuration

```
config terminal
 system
  host-name vbond
  system-ip 1.1.1.2
  site-id 1
  organization-name helena
  vbond 172.16.0.92 local
  ntp server 172.16.0.1
   version 4
  exit
 !
 vpn 0
  interface ge0/0
   ip address 172.16.0.92/24
   no shutdown
   tunnel-interface
    encapsulation ipsec
    allow-service all
   exit
  exit
  ip route 0.0.0.0/0 172.16.0.1
 !
 vpn 512
  interface eth0
   ip address 198.18.129.92/18
   no shutdown
  exit
  ip route 0.0.0.0/0 198.18.128.1
 !
commit and-quit
```

### vSmart Bootstrap Configuration

```
config terminal
 system
  host-name vsmart
  system-ip 1.1.1.3
  site-id 1
  organization-name helena
  vbond 172.16.0.92
  ntp server 172.16.0.1
   version 4
  exit
 !
 vpn 0
  interface eth0
   ip address 172.16.0.93/24
   no shutdown
   tunnel-interface
    allow-service all
   exit
  exit
  ip route 0.0.0.0/0 172.16.0.1
 !
 vpn 512
  interface eth1
   ip address 198.18.129.93/18
   no shutdown
  exit
  ip route 0.0.0.0/0 198.18.128.1
 !
commit and-quit
```

<br>

---

With the network interfaces and routing established, the vManage web UI becomes accessible via HTTPS. On the initial welcome screen, we select **Standard** setup mode to configure system settings and onboard network devices.

![x](/static/2026-11-20-cisco-sdwan/03.png)

<br>

---

## Initial Configuration & Enterprise PKI

Under **Administration → Settings → Organization Name**, we configure our organization name as `helena`. This parameter must match identically across all controllers and edge routers for mutual authentication.

![x](/static/2026-11-20-cisco-sdwan/04.png)

<br>

---

Under **Administration → Settings → Validator**, we enter the vBond Validator IPv4 address (`172.16.0.92`) and listening port `12346`.

![x](/static/2026-11-20-cisco-sdwan/05.png)

<br>

---

Because we manage our own Public Key Infrastructure, under **Administration → Settings → Trust and Privacy → Certificate settings**, we switch both Control component and WAN edge cloud certificate authorizations from Cisco PKI to **Enterprise**. We paste our Enterprise Root CA certificate chain into the field to establish the fabric root of trust.

![x](/static/2026-11-20-cisco-sdwan/06.png)

<br>

---

Navigating to **Configuration → Devices → Control components**, vManage is listed as our sole control device with status `No certificate installed`. We proceed to generate and sign its identity certificate via the **Certificate management** menu.

![x](/static/2026-11-20-cisco-sdwan/07.png)

<br>

---

Under the certificate management workflow, we click **Generate CSR**, download the generated Certificate Signing Request, sign it with our Enterprise CA, and upload the signed certificate (`signed_vmanage.cer`).

![x](/static/2026-11-20-cisco-sdwan/08.png)

<br>

---

Once installed, vManage's operational state updates to `vBond Updated` with its certificate serial number active and valid.

![x](/static/2026-11-20-cisco-sdwan/09.png)

<br>

---

## Adding Controllers (vBond & vSmart)

Under **Configuration → Devices → Control components**, we click **Add control component** and choose **Validator**. We enter the vBond management IP (`172.16.0.92`), administrator credentials, and select **Manual** certificate renewal.

![x](/static/2026-11-20-cisco-sdwan/10.png)

<br>

---

We download the vBond CSR, sign it with our Enterprise CA, and upload the resulting `signed_vbond.cer` certificate.

![x](/static/2026-11-20-cisco-sdwan/11.png)

<br>

---

Next, we click **Add control component** and select **Controller** to add the vSmart controller, entering its management IP (`172.16.0.93`), credentials, and manual renewal mode.

![x](/static/2026-11-20-cisco-sdwan/12.png)

<br>

---

We download the vSmart CSR, sign it with our Enterprise CA, and upload `signed_vsmart.cer`.

![x](/static/2026-11-20-cisco-sdwan/13.png)

<br>

---

Returning to **Configuration → Devices → Control components**, all three controllers (**vManage**, **vBond**, and **vSmart**) are online, synchronized, and authenticated with valid enterprise certificates.

![x](/static/2026-11-20-cisco-sdwan/14.png)

<br>

---

## Onboarding Edge Routers

To onboard our edge routers, we log into **Cisco Software Central** (`software.cisco.com`) and navigate to **Network Plug and Play → Plug and Play Connect**.

![x](/static/2026-11-20-cisco-sdwan/15.png)

<br>

---

Under Controller Profiles, we click **Add Profile** and select Controller Type **VBOND**.

![x](/static/2026-11-20-cisco-sdwan/16.png)

<br>

---

We name the profile `HELENA`, set the deployment type to **Customer Hosted**, specify our Organization Name `helena`, and enter the primary controller vBond address `172.16.0.92` with protocol `DTLS` on port `12346`.

![x](/static/2026-11-20-cisco-sdwan/17.png)

<br>

---

Under Devices, we click **Add Software Devices**, select Base PID **C8000V**, specify quantity **5**, and associate them with our `HELENA` Controller Profile.

![x](/static/2026-11-20-cisco-sdwan/18.png)

<br>

---

The portal generates 5 virtual C8000V serial numbers and chassis IDs in **Provisioned** state.

![x](/static/2026-11-20-cisco-sdwan/19.png)

<br>

---

Under Controller Profiles, we download the WAN Edge provisioning file (`provisioning_file.viptela` / `serialFile.viptela`).

![x](/static/2026-11-20-cisco-sdwan/20.png)

<br>

---

Back in vManage under **Configuration → Devices → WAN Edge list**, we click **Add WAN edges** and select **Upload WAN Edge List**.

![x](/static/2026-11-20-cisco-sdwan/21.png)

<br>

---

In the Quick Connect wizard, we select **Upload file with serial numbers** and upload `serialFile.viptela`.

![x](/static/2026-11-20-cisco-sdwan/22.png)

<br>

---

We select all 5 imported C8000V chassis entries and click **Skip configuration** to bring them into the device inventory.

![x](/static/2026-11-20-cisco-sdwan/23.png)

<br>

---

We review the device summary and click **Onboard** to commit the whitelist to vManage and distribute it to vBond and vSmart.

![x](/static/2026-11-20-cisco-sdwan/24.png)

<br>

---

Under **Configuration → Devices → WAN Edge list**, all 5 authorized C8000V chassis appear in the inventory awaiting bootstrap activation.

![x](/static/2026-11-20-cisco-sdwan/25.png)

<br>

---

## Installing & Provisioning Edge Routers

We prepare the Cisco Catalyst 8000V Edge Software Router virtual appliance installer (`c8000v-universalk9.17.18.05.ova`).

![x](/static/2026-11-20-cisco-sdwan/25a.png)

<br>

---

We boot the virtual router for Site 11 (`c8k-11-jkt`) and apply the initial base bootstrap configuration via CLI:

```
config-transaction
 system
  system-ip 10.11.1.1
  site-id 11
  admin-tech-on-failure
  organization-name helena
  vbond 172.16.0.92
 exit
 !
 ntp server 172.17.0.1
 ntp server 172.18.0.1
 !
 interface GigabitEthernet1
  no shutdown
  ip address 172.17.0.11 255.255.255.0
 exit
 !
 interface GigabitEthernet2
  no shutdown
  ip address 172.18.0.11 255.255.255.0
 exit
 !
 ip route 0.0.0.0 0.0.0.0 172.17.0.1
 ip route 0.0.0.0 0.0.0.0 172.18.0.1
 !
 interface Tunnel1
  no shutdown
  ip unnumbered GigabitEthernet1
  tunnel source GigabitEthernet1
  tunnel mode sdwan
 exit
 !
 interface Tunnel2
  no shutdown
  ip unnumbered GigabitEthernet2
  tunnel source GigabitEthernet2
  tunnel mode sdwan
 exit
 !
 sdwan
  interface GigabitEthernet1
   tunnel-interface
    encapsulation ipsec
    color biz-internet
    allow-service all
   exit
  exit
  interface GigabitEthernet2
   tunnel-interface
    encapsulation ipsec
    color public-internet
    allow-service all
   exit
  exit
 exit
commit
```

We install our Enterprise Root CA certificate chain into the router bootflash to enable trust establishment with the controllers:

```
edge-11-jkt#request platform software sdwan root-cert-chain install bootflash:CA.cer
```

![x](/static/2026-11-20-cisco-sdwan/26.png)

<br>

---

In vManage under **Configuration → Devices → WAN Edge list**, we locate the unmanaged device entry and click **Generate Bootstrap Configuration** from the context menu.

![x](/static/2026-11-20-cisco-sdwan/27.png)

<br>

---

The bootstrap dialog generates the UUID and One-Time Password (OTP) activation token:
- **UUID / Chassis ID**: `C8K-7C396478-00F8-9657-D6F8-72884AF95985`
- **OTP Token**: `bb9288e019bb4a01a59269e1a1dd9d29`

![x](/static/2026-11-20-cisco-sdwan/28.png)

<br>

---

On the router console, we activate the chassis using the generated credentials:

```
edge-11-jkt#request platform software sdwan vedge_cloud activate chassis-number C8K-7C396478-00F8-9657-D6F8-72884AF95985 token bb9288e019bb4a01a59269e1a1dd9d29
```

The router establishes a temporary control connection to vBond, receives validation, and authenticates Netconf over SSH from vManage (`1.1.1.1`).

![x](/static/2026-11-20-cisco-sdwan/29.png)

<br>

---

In vManage, the router appears in the WAN Edge list with certificate state `CSR Generated` and reachability green. We click **Certificate management** to sign the router's identity certificate.

![x](/static/2026-11-20-cisco-sdwan/30.png)

<br>

---

Under WAN Edges certificate management, we download the CSR for `edge-11-jkt`, sign it with our Enterprise CA, and upload `signed_edge-11-jkt.cer`.

![x](/static/2026-11-20-cisco-sdwan/31.png)

<br>

---

With the signed certificate installed, `edge-11-jkt` shows certificate valid with a green checkmark, operating version `17.18.05.0.86`, and reachability active.

![x](/static/2026-11-20-cisco-sdwan/32.png)

<br>

---

We repeat the activation and certificate signing procedure for the Site 12 edge router (`edge-12-sby` / `10.12.1.1`). Both branch routers are now recognized as valid WAN edge members of the fabric.

![x](/static/2026-11-20-cisco-sdwan/33.png)

<br>

---

## Creating Feature & Device Templates

To standardize configuration across the fabric, we build reusable, modular **Feature Templates** and bundle them into a **Device Template**.

### System Template

Under **Configuration → Templates → Feature Templates**, we click **Add Template**, select device type **C8000v**, and choose **Cisco System** (`C8K-System`). We parameterize site-specific values as device variables:
- **Site ID**: `[system_site_id]`
- **System IP**: `[system_system_ip]`
- **Hostname**: `[system_host_name]`

![x](/static/2026-11-20-cisco-sdwan/34.png)

<br>

---

### VPN 0 (Transport VPN) Templates

Under Feature Templates, we create a **Cisco VPN** template named `C8K-VPN0` with VPN ID `0` and name `TRANSPORT-VPN`.

![x](/static/2026-11-20-cisco-sdwan/35.png)

<br>

---

Under IPv4 Route, we add default routes pointing to our transport gateways: `172.17.0.1` (`biz-internet`) and `172.18.0.1` (`public-internet`).

![x](/static/2026-11-20-cisco-sdwan/36.png)

<br>

---

We create an interface feature template for WAN 1 named `C8K-VPN0-WAN1` bound to `GigabitEthernet1` with variable IP `[vpn0_wan1_if_ipv4_address]`.

![x](/static/2026-11-20-cisco-sdwan/37.png)

<br>

---

Under Tunnel Interface, we enable the tunnel and set the color to `biz-internet`.

![x](/static/2026-11-20-cisco-sdwan/38.png)

<br>

---

Under Allow Service, we enable `All` to allow control connections and management protocols.

![x](/static/2026-11-20-cisco-sdwan/39.png)

<br>

---

We create the WAN 2 interface template named `C8K-VPN0-WAN2` bound to `GigabitEthernet2` with variable IP `[vpn0_wan2_if_ipv4_address]`.

![x](/static/2026-11-20-cisco-sdwan/40.png)

<br>

---

Under Tunnel Interface, we set the color to `public-internet`.

![x](/static/2026-11-20-cisco-sdwan/41.png)

<br>

---

Under Allow Service, we enable `All`.

![x](/static/2026-11-20-cisco-sdwan/42.png)

<br>

---

### VPN 1 (Service VPN) Templates

We create the Service VPN template named `C8K-VPN1` with VPN ID `1` and name `SERVIC-LAN`.

![x](/static/2026-11-20-cisco-sdwan/43.png)

<br>

---

Under Advertise OMP, we configure the router to redistribute `Connected` and `Static` routes into OMP so service subnets are advertised across the fabric.

![x](/static/2026-11-20-cisco-sdwan/44.png)

<br>

---

We create the Service LAN interface template named `C8K-VPN1-LAN` bound to `GigabitEthernet3` with variable IP `[vpn1_lan_if_ipv4_address]`.

![x](/static/2026-11-20-cisco-sdwan/45.png)

<br>

---

### AAA Template

Under Feature Templates, we create a **Cisco AAA** template named `C8K-AAA` configuring local admin user credentials (`admin` and `helena`) with privilege level 15.

![x](/static/2026-11-20-cisco-sdwan/46.png)

<br>

---

We review our completed suite of 7 custom Feature Templates in the list view:
- `C8K-System` (Cisco System)
- `C8K-VPN0` (Cisco VPN)
- `C8K-VPN0-WAN1` (Cisco VPN Interface Ethernet)
- `C8K-VPN0-WAN2` (Cisco VPN Interface Ethernet)
- `C8K-VPN1` (Cisco VPN)
- `C8K-VPN1-LAN` (Cisco VPN Interface Ethernet)
- `C8K-AAA` (Cisco AAA)

![x](/static/2026-11-20-cisco-sdwan/47.png)

<br>

---

### Device Template Assembly & Push

Under **Configuration → Templates → Device Templates**, we click **Create Template → From Feature Template** for model **C8000v** and name it `C8K-DualWAN`. We attach `C8K-System` and `C8K-AAA`.

![x](/static/2026-11-20-cisco-sdwan/48.png)

<br>

---

Under Transport & Management VPN, we attach `C8K-VPN0` along with its sub-templates `C8K-VPN0-WAN1` and `C8K-VPN0-WAN2`.

![x](/static/2026-11-20-cisco-sdwan/49.png)

<br>

---

Under Service VPN, we add VPN 1 attaching `C8K-VPN1` and interface sub-template `C8K-VPN1-LAN`.

![x](/static/2026-11-20-cisco-sdwan/50.png)

<br>

---

From the Device Templates list, we click the action menu for `C8K-DualWAN` and select **Attach Devices**.

![x](/static/2026-11-20-cisco-sdwan/51.png)

<br>

---

We move `edge-11-jkt` and `edge-12-sby` into the Selected Devices pane and click **Attach**.

![x](/static/2026-11-20-cisco-sdwan/52.png)

<br>

---

We input the site-specific variable values for `edge-11-jkt`:
- **Hostname**: `edge-11-jkt`
- **System IP**: `10.11.1.1`
- **Site ID**: `11`
- **VPN 0 WAN 1 IP**: `172.17.0.11/24`
- **VPN 0 WAN 2 IP**: `172.18.0.11/24`
- **VPN 1 LAN IP**: `10.11.0.1/24`

![x](/static/2026-11-20-cisco-sdwan/53.png)

<br>

---

We input the parameters for `edge-12-sby`:
- **Hostname**: `edge-12-sby`
- **System IP**: `10.12.1.1`
- **Site ID**: `12`
- **VPN 0 WAN 1 IP**: `172.17.0.12/24`
- **VPN 0 WAN 2 IP**: `172.18.0.12/24`
- **VPN 1 LAN IP**: `10.12.0.1/24`

![x](/static/2026-11-20-cisco-sdwan/54.png)

<br>

---

We review the compiled configuration preview and diff before deploying.

![x](/static/2026-11-20-cisco-sdwan/55.png)

<br>

---

We click **Configure Devices** to push the templates. Both devices successfully pull and commit the configuration via Netconf.

![x](/static/2026-11-20-cisco-sdwan/56.png)

<br>

---

Under **Configuration → Devices → WAN Edge list**, both routers now reflect **Managed by**: `Template C8K-DualWAN` with status `In Sync`.

![x](/static/2026-11-20-cisco-sdwan/57.png)

<br>

---

Under **Monitor → Devices**, all five fabric devices (**vmanage**, **vbond**, **vsmart**, **edge-11-jkt**, and **edge-12-sby**) report green health status, active reachability, and fully established control connections.

![x](/static/2026-11-20-cisco-sdwan/58.png)

<br>

---

## Verification & Data Plane Testing

On `edge-11-jkt`, we verify control connections and local properties:

```
edge-11-jkt#show sdwan control local-properties | include personality|organization-name|certificate-status|site-id|domain-id|system-ip
personality               vedge
sp-organization-name      helena
organization-name         helena
certificate-status        Installed
site-id                   11
domain-id                 1
system-ip                 10.11.1.1

edge-11-jkt#show sdwan control connections
```

The output confirms active DTLS control connections to **vSmart** (`1.1.1.3`) over both transports (`biz-internet` and `public-internet`), to **vBond** (`172.16.0.92`), and to **vManage** (`1.1.1.1`).

![x](/static/2026-11-20-cisco-sdwan/59.png)

<br>

---

We inspect OMP peerings and routing updates for VPN 1:

```
edge-11-jkt#show sdwan omp peers
edge-11-jkt#show sdwan omp routes vpn 1 | b TENANT
edge-11-jkt#show ip route vrf 1 | b Gateway
```

The router receives the remote Site 12 LAN prefix (`10.12.0.0/24`) via OMP from vSmart (`1.1.1.3`), installed into the VRF 1 routing table via next hop `10.12.1.1` using `Sdwan-system-intf`.

![x](/static/2026-11-20-cisco-sdwan/60.png)

<br>

---

We verify Bidirectional Forwarding Detection (BFD) sessions between sites:

```
edge-11-jkt#show sdwan bfd sessions
```

Four active IPsec BFD tunnels are up between Site 11 and Site 12 (`10.12.1.1`), forming a full mesh across both transports:
- `biz-internet` → `biz-internet`
- `biz-internet` → `public-internet`
- `public-internet` → `biz-internet`
- `public-internet` → `public-internet`

We test data-plane connectivity by pinging the remote Site 12 gateway (`10.12.0.1`) from `GigabitEthernet3` in VRF 1 with 100% success rate:

```
edge-11-jkt#ping vrf 1 10.12.0.1 source GigabitEthernet3
```

![x](/static/2026-11-20-cisco-sdwan/61.png)

<br>

---

From the end-host workstations on the LAN segments, we verify end-to-end site-to-site communication:
- Client at Site 11 (`10.11.0.21`) successfully pings Client at Site 12 (`10.12.0.22`) with `time=2ms`.
- Client at Site 12 (`10.12.0.22`) successfully pings Client at Site 11 (`10.11.0.21`).

![x](/static/2026-11-20-cisco-sdwan/62.png)

<br>

---

## Enabling Direct Internet Access (DIA)

By default, branch traffic inside Service VPN 1 cannot exit directly to the public Internet through the local WAN interfaces because VPN 1 lacks an external default route and Network Address Translation (NAT) is not enabled on the transport interfaces.

To enable Direct Internet Access (DIA):

Under Feature Templates for `C8K-VPN0-WAN1` and `C8K-VPN0-WAN2`, we navigate to the **NAT** tab and switch **NAT** to **On** with **NAT Type** set to **Interface**.

![x](/static/2026-11-20-cisco-sdwan/63.png)

<br>

---

Next, in the `C8K-VPN1` template under **Route Leak**, we configure inter-VPN route leaking between the Global VPN (VPN 0) and Service VPN (VPN 1). Under **Add New Route Leak from Global VPN to Service VPN**, we select protocol **static** to leak the default routes (`0.0.0.0/0`) from VPN 0 into VPN 1.

![x](/static/2026-11-20-cisco-sdwan/64.png)

<br>

---

After updating and pushing the templates, we verify the routing table in VRF 1 on `edge-11-jkt`:

```
edge-11-jkt#show ip route vrf 1 | b Gateway
Gateway of last resort is 172.18.0.1 to network 0.0.0.0

S* + 0.0.0.0/0 [1/0] via 172.18.0.1
               [1/0] via 172.17.0.1
```

Default routes pointing to both Internet underlay gateways (`172.18.0.1` and `172.17.0.1`) are now leaked directly into VRF 1.

![x](/static/2026-11-20-cisco-sdwan/65.png)

<br>

---

Finally, from the LAN workstation (`10.11.0.21`), we verify outbound Internet connectivity:
- Ping to `google.com` (`142.251.175.100`) succeeds with 0% packet loss.
- Traceroute to Google DNS (`8.8.8.8`) demonstrates traffic exiting the local default gateway (`10.11.0.1`), translating via the local WAN underlay (`172.18.0.1`), and reaching the public destination in 3 hops without backhauling across the SD-WAN overlay.

![x](/static/2026-11-20-cisco-sdwan/66.png)
