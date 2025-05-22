# NoteSlash
Quick Slash with Cheatsheet

---
Reference From TryHackMe~
---
# 🔰 Initial Access (From: Breaching Active Directory)

Two popular methods for gaining access to that first set of AD credentials is Open Source Intelligence (OSINT) and Phishing.

## NTLM (New Technology LAN Manager):

~ Suite of security protocols used to authenticate users' identities in AD.

~ NTLM can be used for authentication by using a challenge-response-based scheme called NetNTLM.

~ Heavily used by services on a network.

~ NetNTLM, also often referred to as Windows Authentication or just NTLM Authentication.

~ Allows the application to play the role of a middle man between the client and AD. All authentication material is forwarded to a Domain Controller in the form of a challenge, and if completed successfully, the application will authenticate the user.

This means that the application is authenticating on behalf of the user and not authenticating the user directly on the application itself. This prevents the application from storing AD credentials, which should only be stored on a DC.

![image](https://github.com/user-attachments/assets/ab62dc18-43b4-4bb4-842f-0bff8494c12a)

---

## LDAP (Lightweight Directory Access Protocol):

~ Default port of LDAP is 389

~ LDAP authentication is similar to NTLM authentication. However, with LDAP authentication, the application directly verifies the user's credentials. The application has a pair of AD credentials that it can use first to query LDAP and then verify the AD user's credentials.

~ A popular mechanism with third-party (non-Microsoft) applications that integrate with AD. These include applications and systems such as:
  Gitlab,  Jenkins,  Custom-developed web applications,  Printers,  VPNs.

![image](https://github.com/user-attachments/assets/4e54b4f1-6094-4cf2-b5a9-adcbbbc42411)


**# LDAP Pass-back:**

LDAP Pass-back attack is a common attack against network devices, such as printers, when you have gained initial access to the internal network, such as plugging in a rogue device in a boardroom.

LDAP Pass-back attacks can be performed when we gain access to a device's configuration where the LDAP parameters are specified. This can be, for example, the web interface of a network printer. Usually, the credentials for these interfaces are kept to the default ones, such as admin:admin or admin:password. Here, we won't be able to directly extract the LDAP credentials since the password is usually hidden. However, we can alter the LDAP config, such as the IP or hostname of the LDAP server. In an LDAP Pass-back attack, we can modify this IP to our IP and then test the LDAP configuration, which will force the device to attempt LDAP authentication to our rogue device. We can intercept this authentication attempt to recover the LDAP credentials.

![image](https://github.com/user-attachments/assets/ec0f24fd-2829-4235-9505-74e5676072b5)

**# Hosting a Rogue LDAP Server:**

sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd

sudo dpkg-reconfigure -p low slapd

![image](https://github.com/user-attachments/assets/274e6d6a-4e2b-4559-a11b-136a1d3e383c)

sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart

Capturing LDAP Credentials:  
sudo tcpdump -SX -i breachad tcp port 389

---

## Authentication Relays:

In Windows networks, there are a significant amount of services talking to each other, allowing users to make use of the services provided by the network.

These services have to use built-in authentication methods to verify the identity of incoming connections.

~ Here, focus on NetNTLM authentication used by SMB.

**# SMB (Server Message Block):**

SMB protocol allows clients (like workstations) to communicate with a server (like a file share). In networks that use Microsoft AD, SMB governs everything from inter-network file-sharing to remote administration. Even the "out of paper" alert your computer receives when you try to print a document is the work of the SMB protocol.

**# LLMNR, NBT-NS, and WPAD**

In this task, we will take a bit of a look at the authentication that occurs during the use of SMB. We will use Responder to attempt to intercept the NetNTLM challenge to crack it. There are usually a lot of these challenges flying around on the network. Some security solutions even perform a sweep of entire IP ranges to recover information from hosts. Sometimes due to stale DNS records, these authentication challenges can end up hitting your rogue device instead of the intended host.

Responder allows us to perform Man-in-the-Middle attacks by poisoning the responses during NetNTLM authentication, tricking the client into talking to you instead of the actual server they wanted to connect to. On a real LAN, Responder will attempt to poison any  Link-Local Multicast Name Resolution (LLMNR),  NetBIOS Name Service (NBT-NS), and Web Proxy Auto-Discovery (WPAD) requests that are detected. On large Windows networks, these protocols allow hosts to perform their own local DNS resolution for all hosts on the same local network. Rather than overburdening network resources such as the DNS servers, hosts can first attempt to determine if the host they are looking for is on the same local network by sending out LLMNR requests and seeing if any hosts respond. The NBT-NS is the precursor protocol to LLMNR, and WPAD requests are made to try and find a proxy for future HTTP(s) connections.

Since these protocols rely on requests broadcasted on the local network, our rogue device would also receive these requests. Usually, these requests would simply be dropped since they were not meant for our host. However, Responder will actively listen to the requests and send poisoned responses telling the requesting host that our IP is associated with the requested hostname. By poisoning these requests, Responder attempts to force the client to connect to our AttackBox. In the same line, it starts to host several servers such as SMB, HTTP, SQL, and others to capture these requests and force authentication.

**# Intercepting NetNTLM Challenge**

One thing to note is that Responder essentially tries to win the race condition by poisoning the connections to ensure that you intercept the connection. This means that Responder is usually limited to poisoning authentication challenges on the local network. Since we are connected via a VPN to the network, we will only be able to poison authentication challenges that occur on this VPN network. For this reason, we have simulated an authentication request that can be poisoned that runs every 30 minutes. This means that you may have to wait a bit before you can intercept the NetNTLM challenge and response.

Although Responder would be able to intercept and poison more authentication requests when executed from our rogue device connected to the LAN of an organisation, it is crucial to understand that this behaviour can be disruptive and thus detected. By poisoning authentication requests, normal network authentication attempts would fail, meaning users and services would not connect to the hosts and shares they intend to. Do keep this in mind when using Responder on a security assessment.

https://github.com/lgandx/Responder. We will set Responder to run on the interface connected to the VPN: 
sudo responder -I breachad

![image](https://github.com/user-attachments/assets/a42e2803-c056-48f4-a30f-022d81628bc1)

Use hashtype 5600, which corresponds with NTLMv2-SSP for hashcat.

**# Relaying the challenge***

![image](https://github.com/user-attachments/assets/9be38888-3720-414a-8f3b-7a07e6679de2)

---

## Microsoft Deployment Toolkit (MDT)

~ A Microsoft service that assists with automating the deployment of Microsoft Operating Systems (OS). Large organisations use services such as MDT to help deploy new images in their estate more efficiently since the base images can be maintained and updated in a central location.

~ Usually, MDT is integrated with Microsoft's System Center Configuration Manager (SCCM), which manages all updates for all Microsoft applications, services, and operating systems. MDT is used for new deployments. Essentially it allows the IT team to preconfigure and manage boot images. Hence, if they need to configure a new machine, they just need to plug in a network cable, and everything happens automatically.

~ SCCM can be seen as almost an expansion and the big brother to MDT. What happens to the software after it is installed? Well, SCCM does this type of patch management. It allows the IT team to review available updates to all software installed across the estate. The team can also test these patches in a sandbox environment to ensure they are stable before centrally deploying them to all domain-joined machines. It makes the life of the IT team significantly easier.

**# Preboot Ex
ecution Environment (PXE) boot**

![image](https://github.com/user-attachments/assets/5dbaacef-3bd8-4f1e-8b87-dcc8fc723d01)

**# PXE Boot Image Retrieval**

![image](https://github.com/user-attachments/assets/5c7de561-a25f-443e-bee7-77654e4b0660)
ssh thm@THMJMP1.za.tryhackme.com and the password of Password1@.

A BCD (Boot Configuration Data) file is a configuration file used by Windows to determine how to boot. It stores information about which operating systems to load, their locations, and boot parameters.

![image](https://github.com/user-attachments/assets/854f4978-9966-40ce-8f63-086fdeaf3697)

**# Recovering Credentials from a PXE Boot Image**

![image](https://github.com/user-attachments/assets/baa55d4d-16b6-47f4-bc09-1f3de143dd02)

---

## Config Files




---

# 🔰 Initial Access (From: Breaching Active Directory)









