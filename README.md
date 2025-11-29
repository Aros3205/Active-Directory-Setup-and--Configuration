# On-Prem Active Directory Lab – Project 1  
**Core Domain Controller & Windows 8 Client Setup (VirtualBox)**

This project is the **first part** of my on-prem Active Directory lab series.  
Here I focus on:

- Building the **network foundation** in VirtualBox  
- Configuring **Windows Server 2022** as a Domain Controller (AD DS + DNS)  
- Putting **Windows 8 client(s)** on the same internal subnet  
- Testing connectivity with **ping** and preparing for domain join  

Project 2 will build on this by adding **OUs, groups, users, and role-based access (IAM).**

---

## 🧱 Lab Topology (Project 1 Scope)

**Hypervisor:** Oracle VirtualBox  
**VMs in this project:**

| VM Name              | Role                                   |
|----------------------|----------------------------------------|
| `Magic_Windows_Server_02` | Domain Controller (AD DS + DNS)        |
| `Windows_8_Wizard`   | Windows 8 client (future domain member) |

**Network Design:**

- **Adapter 1 (Server) → NAT**  
  - Used as “Internet router” going outside
- **Adapter 2 (Server) → Internal Network (`intnet`)**  
  - Used as the **private LAN** for all lab machines  
- **Windows 8 clients → Internal Network (`intnet`) only**  
  - They talk to the server **through Adapter 2** only

You can think of it like this:

> NAT = router to the outside world  
> Internal Network (`intnet`) = internal company LAN where the server and clients live

---

## 1️⃣ Configure Server Network in VirtualBox

On the **Windows Server VM** (`Magic_Windows_Server_02`):

- **Adapter 1**: NAT (for Internet)  
- **Adapter 2**: Internal Network (`intnet`) – for lab LAN

![Server – VirtualBox Network Adapters](screenshot/Screenshot%20(52).png)

This gives the server **two NICs**:
- One for the **Internet**
- One for the **internal AD network**

---

## 2️⃣ Configure Internal IP on the Server

Inside the **Windows Server** VM:

1. Open **ncpa.cpl** (Network Connections).
2. Identify the **internal NIC** (connected to `intnet`).
3. Set a **static IPv4** address, e.g.:

   - IP address: `193.168.10.1`  
   - Subnet mask: `255.255.255.0`  
   - Default gateway: *(empty or self, depending on your setup)*  
   - Preferred DNS server: `193.168.10.1` (itself, because it will run DNS)

*(Screenshots below show the network configuration work in progress.)*

![Server – Internal NIC Properties](screenshot/Screenshot%20(61).png)

![Server – IPv4 Static Configuration](screenshot/Screenshot%20(62).png)

---

## 3️⃣ Install Active Directory Domain Services (AD DS) & DNS

With the base network ready, I prepared the server to become a Domain Controller:

1. Open **Server Manager** → **Add roles and features**.
2. Role-based installation on this server.
3. Select:
   - **Active Directory Domain Services**
   - **DNS Server** (if not auto-checked)
4. Proceed through the wizard and install.

![Server Manager – Add Roles and Features](screenshot/Screenshot%20(63).png)

![Selecting AD DS Role](screenshot/Screenshot%20(64).png)

![Adding AD DS & DNS](screenshot/Screenshot%20(65).png)

![Installation Progress](screenshot/Screenshot%20(66).png)

After the roles were installed, **Server Manager** showed a yellow flag with a post-deployment task:  

> *Promote this server to a domain controller*

---

## 4️⃣ Promote Server to Domain Controller (Create `rg.local`)

Next, I promoted the server to a **new forest**:

1. In **Server Manager**, click the **flag notification** →  
   **Promote this server to a domain controller**.
2. Choose **Add a new forest**.
3. Use domain name:

   ```text
   rg.local
Set a strong DSRM password.

Accept the default options (DNS, GC, etc.), then continue.

Review and click Install.




When the promotion finished, the server restarted and came back up as the Domain Controller for rg.local.

5️⃣ Configure Windows_8_Wizard Network
Now I prepared the first Windows 8 client: Windows_8_Wizard.

5.1 VirtualBox Network
In VirtualBox, for Windows_8_Wizard:

Adapter 1: Internal Network → intnet

(No NAT needed for this project on the client)


5.2 IPv4 Settings Inside Windows 8
Inside the guest OS:

Run ncpa.cpl → open Ethernet properties.

Set static IPv4:

IP address: 193.168.10.10

Subnet mask: 255.255.255.0

Default gateway: 193.168.10.1

Preferred DNS server: 193.168.10.1


Then I verified with:

bash
Copy code
ipconfig


6️⃣ Test Connectivity: Client ↔ Server
To confirm that the Windows 8 client could “see” the Domain Controller:

From Windows_8_Wizard:

bash
Copy code
ping 193.168.10.1

After fixing a small typo in the IP address, the ping succeeded:


This confirmed:

Same subnet

Correct gateway

Correct DNS (pointing at the server)

No firewall issues blocking basic ICMP between server and client

7️⃣ Prepare for Domain Join (rg.local)
Once connectivity was tested, I prepared the client to join the rg.local domain:

On Windows 8, open:

This PC → Properties → Change settings → Change…

Select Domain and type:

text
Copy code
rg.local

When prompted for credentials, use a domain account on the server (for example Administrator of rg.local).

8️⃣ Domain Join & Restart (Conceptual Step – For Next Project)
Because this repository is focused on Project 1: Core Setup, I stop right after:

The Domain Controller is fully configured

The client is correctly networked

The client can resolve and ping the DC (193.168.10.1)

The domain name rg.local is recognized at the join dialog

The actual join and employee logons (e.g. Babatunde Raji) are part of Project 2: IAM & Role Assignment, which will live in a separate documentation.

Screenshots below capture the point where the domain is found and the system is ready to complete the join:



✅ What I Achieved in Project 1
By the end of this project, I achieved:

✅ Built an on-prem lab environment in VirtualBox

✅ Configured dual-homed Windows Server with:

NAT for Internet

Internal Network (intnet) for LAN

✅ Installed and configured Active Directory Domain Services (AD DS) & DNS

✅ Promoted the server to a Domain Controller for rg.local

✅ Put Windows 8 client on the same internal subnet with:

Static IP: 193.168.10.10

Gateway: 193.168.10.1

DNS: 193.168.10.1

✅ Verified end-to-end connectivity using ping

This gives me a solid, realistic foundation for the next stage:
IAM, OUs, groups, users, and proper role-based access.

🔮 Next Steps (Project 2 – IAM & Role Assignment)
In the next project/repo, I will:

Complete the domain join for Windows_8_Wizard and the second Windows 8 VM

Design Organizational Units (OUs) for UK / US / Nigeria

Create security groups (Sales, IT, HR, Marketing, etc.)

Create user accounts (employees) and map them into the right groups

Demonstrate logons and permissions from client machines

That project will be documented as Project 2 and linked from this README once completed.
