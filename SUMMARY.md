# Table of contents

* [Welcome](README.md)

## ✈️ Exposed Services

* [DNS 53](exposed-services/dns-53.md)
* [KRB 88](exposed-services/krb-88.md)
* [LDAP](exposed-services/ldap.md)
* [SMB 139,445](exposed-services/smb-139-445.md)
* [SNMP 161](exposed-services/snmp-161.md)
* [MSSQL](exposed-services/mssql.md)
* [NFS 2049](exposed-services/nfs-2049.md)
* [RPC](exposed-services/rpc.md)

## 🕸️ Web

* [Server Side Attacks](web/server-side-attacks/README.md)
  * [Authentication](web/server-side-attacks/authentication.md)
  * [Command Injection](web/server-side-attacks/command-injection.md)
  * [File Upload](web/server-side-attacks/file-upload.md)
  * [Race Conditions](web/server-side-attacks/race-conditions.md)
  * [SSRF - Server Side Request Forgery](web/server-side-attacks/ssrf-server-side-request-forgery.md)
  * [XXE Injection](web/server-side-attacks/xxe-injection.md)
  * [HTTP Smuggling](web/server-side-attacks/http-smuggling/README.md)
    * [HTTP/2](web/server-side-attacks/http-smuggling/http-2.md)
* [Client Side Attack](web/client-side-attack/README.md)
  * [WebSocket](web/client-side-attack/websocket.md)
  * [CSRF](web/client-side-attack/csrf.md)

## 🪟 Windows

* [Active Directory](windows/active-directory/README.md)
  * [Enumeration](windows/active-directory/enumeration/README.md)
    * [Domain Information](windows/active-directory/enumeration/domain-information.md)
    * [BloodHound](windows/active-directory/enumeration/bloodhound.md)
    * [User Hunting](windows/active-directory/enumeration/user-hunting.md)
  * [Lateral Movement](windows/active-directory/lateral-movement.md)
    * [Pass The \*](windows/active-directory/lateral-movement/pass-the/README.md)
      * [Pass The Hash](windows/active-directory/lateral-movement/pass-the/pass-the-hash.md)
      * [Pass The Ticket](windows/active-directory/lateral-movement/pass-the/pass-the-ticket.md)
      * [Pass The Cert](windows/active-directory/lateral-movement/pass-the/pass-the-cert.md)
    * [Open other sessions](windows/active-directory/lateral-movement/open-other-sessions.md)
    * [File transfer](windows/active-directory/lateral-movement/file-transfer.md)
  * [Exploitation / Abuse](windows/active-directory/exploitation-abuse.md)
    * [Kerberoasting](windows/active-directory/exploitation-abuse/kerberoasting.md)
    * [Groups](windows/active-directory/exploitation-abuse/groups.md)
    * [DACL](windows/active-directory/exploitation-abuse/dacl/README.md)
      * [DCSync](windows/active-directory/exploitation-abuse/dacl/dcsync.md)
      * [GenericALL](windows/active-directory/exploitation-abuse/dacl/genericall.md)
      * [Delegations](windows/active-directory/exploitation-abuse/dacl/delegations.md)
      * [Trust](windows/active-directory/exploitation-abuse/dacl/trust.md)
      * [gMSA](windows/active-directory/exploitation-abuse/dacl/gmsa.md)
    * [Mimi Tools](windows/active-directory/exploitation-abuse/mimi-tools.md)
    * [ADCS](windows/active-directory/exploitation-abuse/adcs/README.md)
      * [ESC1](windows/active-directory/exploitation-abuse/adcs/esc1.md)
      * [ESC3](windows/active-directory/exploitation-abuse/adcs/esc3.md)
      * [ESC4](windows/active-directory/exploitation-abuse/adcs/esc4.md)
      * [ESC6](windows/active-directory/exploitation-abuse/adcs/esc6.md)
    * [Shadow Credentials](windows/active-directory/exploitation-abuse/shadow-credentials.md)
    * [LAPS](windows/active-directory/exploitation-abuse/laps.md)
  * [Azure AD Integrations](windows/active-directory/azure-ad-integrations.md)
* [Persistence](windows/active-directory/persistence.md)
  * [Ticket Forging](windows/persistence/ticket-forging.md)
  * [ACLs](windows/persistence/acls.md)
  * [Golden gMSA](windows/persistence/golden-gmsa.md)
  * [Skeleton Key](windows/persistence/skeleton-key.md)
  * [DSMR](windows/persistence/dsmr.md)
* [Local](windows/local/README.md)
  * [Enumeration](windows/local/enumeration.md)
  * [Privescs](windows/local/privescs/README.md)
    * [Privileges](windows/local/privescs/privileges.md)
    * [Groups](windows/local/privescs/groups.md)
    * [Weak Registry Permissions](windows/local/privescs/weak-registry-permissions.md)
    * [Services Misconfiguration](windows/local/privescs/services-misconfiguration.md)
  * [Download Files](windows/local/download-files.md)
* [Evasion](windows/evasion/README.md)
  * [Payload Delivery](windows/evasion/payload-delivery.md)
  * [EDR](windows/evasion/edr.md)
  * [WDAC](windows/evasion/wdac.md)
* [MSSQL](windows/mssql.md)

## 🐧 Linux

* [TODO](linux/todo.md)

## 🛜 Wi-Fi

* [Page 1](wi-fi/page-1.md)
* [Linux Wireless Tools, Drivers, and Stacks](wi-fi/linux-wireless-tools-drivers-and-stacks.md)
* [Wireshark](wi-fi/wireshark.md)
* [AirCrack-ng](wi-fi/aircrack-ng.md)
* [Cracking Auth Hash](wi-fi/cracking-auth-hash.md)
* [Attaccking WPS](wi-fi/attaccking-wps.md)
* [Rogue Access Point](wi-fi/rogue-access-point.md)
* [Attacking captive portals](wi-fi/attacking-captive-portals.md)
* [Attacking WPA Enterprise](wi-fi/attacking-wpa-enterprise.md)
* [Bettercap](wi-fi/bettercap.md)
* [Manual Network Connections](wi-fi/manual-network-connections.md)

## 🚊 Binary Exploitation

* [Tools and Misc](binary-exploitation/tools-and-misc/README.md)
  * [Assembly](binary-exploitation/tools-and-misc/assembly.md)
  * [Reversing assembly](binary-exploitation/tools-and-misc/reversing-assembly.md)
  * [PWN-Tools](binary-exploitation/tools-and-misc/pwn-tools.md)
  * [Defences](binary-exploitation/tools-and-misc/defences/README.md)
    * [aslr/ pie](binary-exploitation/tools-and-misc/defences/aslr-pie.md)
* [Buffer OverFlow](binary-exploitation/buffer-overflow/README.md)
  * [Stack Based](binary-exploitation/buffer-overflow/stack-based/README.md)
    * [Variables](binary-exploitation/buffer-overflow/stack-based/variables.md)
    * [Call Function](binary-exploitation/buffer-overflow/stack-based/call-function.md)
    * [Shellcode](binary-exploitation/buffer-overflow/stack-based/shellcode.md)

## 📃 Write Ups

* [CRTP Lab](write-ups/crtp-lab.md)
* [WiFi Challenge](write-ups/wifi-challenge.md)

## 🧠 AI

* [Introduction](ai/introduction.md)
* [Prompt Engineering](ai/prompt-engineering.md)
* [Output Attacks](ai/output-attacks.md)
