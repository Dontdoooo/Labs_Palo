# 1. Initial Firewall Configuration

**Skills demonstrated:**

- Connected to firewall via management interface (192.168.1.1)
- Configured management network settings (static IP, DNS, NTP)
- Navigated the web GUI: Dashboard, Policies, Objects, Network, Device panels
- Understood running configuration vs. candidate configuration model
- Configured dynamic content updates for application signatures and threat definitions
- Created local administrative accounts with role-based access

**Key configurations:**

```panos
set deviceconfig system hostname PA-VM-Lab
set deviceconfig system dns-setting servers primary 4.2.2.2
set deviceconfig system dns-setting servers secondary 8.8.8.8
set deviceconfig system ntp-servers primary-ntp-server ntp-server-address pool.ntp.org
set deviceconfig administrators setup-admin permissions role-based superuser yes
```

**Verification:**

```panos
# Verification
show system info | match hostname
#   → Expected: hostname: PA-VM-Lab

show interface management
#   → Expected: ip: 192.168.1.254/24, mtu: 1500, speed: [auto]

show system software status
#   → Expected: PAN-OS version 10.x with "running" state
```

**Challenge & Solution:**

> **Challenge:** Initial management access failed — browser connection to 192.168.1.1 timed out.
> **Root cause:** Default management interface was on a different subnet than the client workstation.
> **Solution:** Connected via console/SSH first, verified management IP with `show interface management`, corrected subnet configuration to match lab topology.


---
# 2. Security Zones & Policies

**Skills demonstrated:**

- Created Trust, Untrust, and DMZ security zones
- Configured Ethernet interfaces and assigned them to zones
- Built security policy rules controlling inter-zone traffic flow
- Understood policy rule matching: source/destination zone, address, port, application
- Configured intra-zone, inter-zone, and universal rule types
- Tested traffic flow and verified policy enforcement via traffic logs

**Key configurations:**

```panos
set zone Trust network layer3 ethernet1/2
set zone Untrust network layer3 ethernet1/1
set zone DMZ network layer3 ethernet1/3
set network interface ethernet ethernet1/2 layer3 ip 192.168.1.254/24
set network interface ethernet ethernet1/1 layer3 ip 203.0.113.20/24
set network interface ethernet ethernet1/3 layer3 ip 172.16.1.1/24
set rulebase security rules Allow-Outbound from Trust to Untrust application web-browsing action allow
set rulebase security rules Allow-Outbound log-end yes
```

**Verification:**

```panos
# Verification
show routing route
#   → Expected: 3 connected routes (192.168.1.0/24, 203.0.113.0/24, 172.16.1.0/24)

show running security-policy
#   → Expected: Allow-Outbound rule (Trust→Untrust, app: web-browsing, action: allow)

show session all filter state active
#   → Expected: Active sessions with correct zone pairs (Trust-to-Untrust) confirming traffic flow
```

**Challenge & Solution:**

> **Challenge:** Inter-zone traffic from Trust to Untrust was being denied despite an allow rule.
> **Root cause:** Security policy rule was configured with incorrect zone direction — source and destination zones were swapped.
> **Solution:** Reviewed traffic logs (`Monitor > Logs > Traffic`), identified the "deny" action was hitting the implicit interzone-default rule. Corrected source/destination zone assignment and committed.


---
# 3. Content-ID Configuration

**Skills demonstrated:**

- Created and applied all 7 security profile types:
  - Antivirus (malware scanning)
  - Anti-Spyware (C2 and spyware detection)
  - Vulnerability Protection (exploit prevention)
  - URL Filtering (web access control)
  - File Blocking (restricting dangerous file types)
  - Data Loss Prevention (sensitive data controls)
  - Service Protection (L3/L4 protocol attacks)
- Configured security profile groups for policy attachment
- Verified Content-ID inspection via threat logs

**Key configurations:**

```panos
set profiles virus Strict-AV decoder http action reset-both
set profiles spyware Strict-AS botnet-domains lists default-paloalto-dns action sinkhole
set profiles vulnerability Strict-VP rules simple-client-critical action reset-both
set profiles url-filtering Strict-URL action block category malware
set profiles file-blocking Block-EXE rules block-executables application any file-type exe direction both action block
set profile-group Strict-Security virus Strict-AV spyware Strict-AS vulnerability Strict-VP url-filtering Strict-URL file-blocking Block-EXE
set rulebase security rules Allow-Outbound profile-setting group Strict-Security
```

**Verification:**

```panos
# Verification
show running security-policy
#   → Expected: Allow-Outbound rule shows "profile: Strict-Security" (profile group attached)

show running threat-prevention
#   → Expected: Active profiles listed — Strict-AV, Strict-AS, Strict-VP, Strict-URL, Block-EXE

test security-policy-match source 192.168.1.100 destination 1.1.1.1 protocol 6 destination-port 443
#   → Expected: "Allow-Outbound" with profile group "Strict-Security" — confirms Content-ID inspection active
```

**Challenge & Solution:**

> **Challenge:** Content-ID profiles were configured but threats were not being blocked.
> **Root cause:** Security profiles were created but not attached to the security policy rule via a profile group.
> **Solution:** Created a Security Profile Group containing all 7 profile types, then attached it to the Allow-Outbound rule under `Actions > Profile Setting > Group`.

---
# 5. Blocking Threats from Known Bad Sources

**Skills demonstrated:**

- Configured External Dynamic Lists sourcing threat intelligence feeds
- Created security policies referencing EDLs for automated blocking
- Implemented URL filtering profiles with cloud-based categorization database
- Configured safe search enforcement for web traffic
- Tested blocking of traffic to/from known bad IP addresses and URLs
- Understood Palo Alto's URL categorization system and database updates

**Key configurations:**

```panos
set address Malicious-IP-1 ip-netmask 198.51.100.0/24 description "Known malicious network"
set address-group Known-Bad-IPs static [ Malicious-IP-1 ]
set external-list Palo-Alto-Known-Malicious-IP type ip recurring five-minute url https://panwdbl.appspot.com/lists/mdl.txt
set rulebase security rules Block-EDL-IPs from any to any source Palo-Alto-Known-Malicious-IP action deny
set profiles url-filtering URL-Filter-Strict action block category command-and-control
```

**Verification:**

```panos
# Verification
request system external-list show type ip name Palo-Alto-Known-Malicious-IP
#   → Expected: Status "Success", entries loaded (e.g., 500+ IPs from threat feed)

show running url-filtering-profile
#   → Expected: URL-Filter-Strict active with "block" action on command-and-control, malware, phishing categories
```

**Challenge & Solution:**

> **Challenge:** External Dynamic List was configured but not blocking traffic to known-bad IPs.
> **Root cause:** EDL had not completed its initial download — status showed "pending" in the EDL object.
> **Solution:** Manually triggered EDL refresh via `request system external-list show name <edl-name>`, confirmed entries loaded, then verified blocking via test traffic and threat logs.

---
# 7. Blocking Threats with User-ID

**Skills demonstrated:**

- Configured User-ID agent for Active Directory integration
- Established LDAP connection to domain controller for user/group mapping
- Implemented user-to-IP address mapping via AD security event logs
- Created security policies based on user identity and group membership
- Verified user identification in traffic and threat logs
- Compared user-based policies vs. IP-based policies for granular control

**Key configurations:**

```panos
set user-id-agent server-monitor windows-server type active-directory host 192.168.1.10
set user-id-agent ldap-proxy server ad-server address 192.168.1.10 port 389 base-dn DC=lab,DC=local
set zone Acquisition enable-user-identification yes
set rulebase security rules Allow-Mktg-Apps from Acquisition to Untrust source-user "lab.local\marketing" application [ facebook-base youtube-base ] action allow
set rulebase security rules Deny-All-Others from Acquisition to Untrust application any action deny
```

**Verification:**

```panos
# Verification
show user ip-user-mapping all
#   → Expected: IP-to-user bindings (e.g., 192.168.1.100 → lab\marketing-user, Src: AD)

show user group list
#   → Expected: lab.local\marketing, lab.local\engineering groups populated from AD

show session all filter source-user CORP\\marketing-user
#   → Expected: Sessions showing user identity, matched rule "Allow-Mktg-Apps", app: facebook-base
```

**Challenge & Solution:**

> **Challenge:** User-ID mappings were not populating — traffic logs showed IP addresses instead of usernames.
> **Root cause:** The User-ID agent was configured but User Identification was not enabled on the zone.
> **Solution:** Enabled User-ID on the Acquisition zone (`Network > Zones > Acquisition > Enable User Identification`), committed, and verified user-to-IP mappings appeared in `show user ip-user-mapping all`.

---
