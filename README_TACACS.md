# Cisco ISE TACACS+ Device Administration Lab

A hands-on lab implementing **role-based device administration** for network infrastructure using **Cisco ISE (Identity Services Engine)** and **TACACS+**, so that switch/router admin access is centrally authenticated, authorized per command, and fully accounted for — instead of relying on shared local passwords.

---

## 📌 Project Objective

Move CLI login for network devices away from local usernames/passwords to a centralized **TACACS+** server on Cisco ISE, enforce **role-based command authorization** (full admin vs. read-only), and prove the model works with live authentication, authorization, and command-accounting logs — including a deliberate failure test.

---

## 🖧 Environment Overview

| Component | Details |
|---|---|
| AAA Server | Cisco ISE — Device Administration Work Center (TACACS+) |
| Network Device | Cisco Catalyst switch (SW1), AAA + TACACS+ client configured |
| Admin user | `net.admin` — member of `NetAdmins` group — full CLI access (Priv 15, all commands) |
| Read-only user | `net.viewer` — member of `NetReadOnly` group — show commands only, config commands denied |

---

## ⚙️ Step 1 — Enable Device Administration on ISE

Enabled the TACACS+ Device Administration service on Cisco ISE and confirmed the application services were running before building any policy.

![Device Administration Service Enabled](images/01-Cisco-ISE-Device-Administration-Service-Enabled.png)
![Application Services Running](images/02-Cisco-ISE-Application-Services-Running.png)

---

## ⚙️ Step 2 — Identities and Network Devices

Created two identity groups (`NetAdmins`, `NetReadOnly`) with matching internal user accounts, and registered the Catalyst switch as a AAA client in ISE.

![TACACS User Identity Groups](images/03-Cisco-ISE-TACACS-User-Identity-Groups.png)
![TACACS Internal User Accounts](images/04-Cisco-ISE-TACACS-Internal-User-Accounts.png)
![TACACS Network Device Registrations](images/05-Cisco-ISE-TACACS-Network-Device-Registrations.png)

---

## ⚙️ Step 3 — Shell Profiles and Command Sets (Policy Elements)

Built the building blocks of role-based access: a shell profile granting Privilege 15, and two command sets — one permitting everything, one permitting only `show` commands.

![Priv15 TACACS Shell Profile](images/06-Cisco-ISE-Priv15-TACACS-Shell-Profile.png)
![PermitAll TACACS Command Set](images/07-Cisco-ISE-PermitAll-TACACS-Command-Set.png)
![ShowOnly TACACS Command Set](images/08-Cisco-ISE-ShowOnly-TACACS-Command-Set.png)

---

## ⚙️ Step 4 — Device Admin Policy Set

Created a Device Administration policy set scoped to the network device's IP, with an authentication policy and three authorization rules mapping identity group → command set + shell profile.

| Rule | Identity Group | Command Set | Shell Profile |
|---|---|---|---|
| Admin | `NetAdmins` | PermitAll | Priv15 |
| ReadOnly | `NetReadOnly` | ShowOnly | Priv15 |
| Default | — | DenyAllCommands | Deny All Shell Profile |

![Network Device Admin Policy Set](images/09-Cisco-ISE-Network-Device-Admin-Policy-Set.png)
![TACACS Authentication Policy](images/10-Cisco-ISE-TACACS-Authentication-Policy.png)
![TACACS Role-Based Authorization Rules](images/11-Cisco-ISE-TACACS-Role-Based-Authorization-Rules.png)

---

## ⚙️ Step 5 — Catalyst Switch AAA/TACACS+ Configuration

Configured the switch to point at the ISE server for AAA, set the TACACS+ source interface, secured VTY lines behind TACACS+ authentication, and confirmed a test login worked.

![Catalyst AAA and TACACS Server Configuration](images/12-Catalyst-AAA-and-TACACS-Server-Configuration.png)
![Catalyst TACACS Source Interface and VTY Configuration](images/13-Catalyst-TACACS-Source-Interface-and-VTY-Configuration.png)
![Catalyst TACACS User Authentication Test](images/14-Catalyst-TACACS-User-Authentication-Test.png)

---

## ✅ Step 6 — Verification: Role-Based Command Authorization

### Read-only user: `show` allowed, `configure` denied
Logged in as `net.viewer` over SSH — `show ip route` succeeded, but `configure terminal` was correctly blocked with **"Command authorization failed."**

![NetViewer Show Command Allowed and Configure Denied](images/17-NetViewer-Show-Command-Allowed-and-Configure-Denied.png)

### ISE-side confirmation for both roles
Cross-checked in ISE that `net.admin` was authorized for full command access and `net.viewer` authenticated successfully but was correctly restricted.

![NetAdmin Command Authorization Success](images/18-Cisco-ISE-NetAdmin-Command-Authorization-Success.png)
![NetAdmin Authorization Protocol Details](images/19-Cisco-ISE-NetAdmin-Authorization-Protocol-Details.png)
![NetViewer Authentication Success](images/20-Cisco-ISE-NetViewer-Authentication-Success.png)
![NetViewer Authentication Protocol Details](images/21-Cisco-ISE-NetViewer-Authentication-Protocol-Details.png)
![NetViewer Configure Command Denied](images/22-Cisco-ISE-NetViewer-Configure-Command-Denied.png)
![NetViewer Denial Protocol Details](images/23-Cisco-ISE-NetViewer-Denial-Protocol-Details.png)

### Live logs and command accounting
Reviewed ISE's live authentication logs showing both roles' results side by side, then confirmed **command accounting** was logging every command run on the device — full traceability of who ran what.

![TACACS Live Logs Role-Based Results](images/24-Cisco-ISE-TACACS-Live-Logs-Role-Based-Results.png)
![TACACS Command Accounting Event](images/25-Cisco-ISE-TACACS-Command-Accounting-Event.png)
![TACACS Command Accounting Details](images/26-Cisco-ISE-TACACS-Command-Accounting-Details.png)
![TACACS Command Accounting Report](images/27-Cisco-ISE-TACACS-Command-Accounting-Report.png)

### Negative test — wrong password
Deliberately tested a login with an incorrect password to confirm ISE correctly rejects invalid credentials and logs the failure with full protocol detail.

![TACACS Wrong Password Live Log](images/28-Cisco-ISE-TACACS-Wrong-Password-Live-Log.png)
![TACACS Wrong Password Failure Details](images/29-Cisco-ISE-TACACS-Wrong-Password-Failure-Details.png)
![TACACS Wrong Password Protocol Attributes](images/30-Cisco-ISE-TACACS-Wrong-Password-Protocol-Attributes.png)

---

## ✅ Results

- Centralized network device admin authentication on **Cisco ISE** using **TACACS+**, replacing local device credentials.
- Implemented **role-based command authorization**: full admin access for `NetAdmins`, `show`-only access for `NetReadOnly`, with `configure terminal` explicitly blocked for read-only users.
- Verified enforcement live on the switch (`Command authorization failed.`) and cross-confirmed the same result in ISE's live logs.
- Confirmed **command accounting** logs every command executed per user for full audit traceability.
- Validated failure handling with a deliberate wrong-password test, confirming ISE correctly rejects and logs invalid login attempts.

---

## 🛠️ Skills Demonstrated

`Cisco ISE` `TACACS+` `AAA (Authentication, Authorization, Accounting)` `Role-Based Access Control` `Device Administration` `Cisco Catalyst Switch Configuration` `Command Authorization` `Security Auditing / Logging` `Network Troubleshooting`
