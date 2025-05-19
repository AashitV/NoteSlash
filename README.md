# NoteSlash
Quick Slash with Cheatsheet

---
Reference From TryHackMe~
---
## 🔰 Initial Access (From: Breaching Active Directory)

Two popular methods for gaining access to that first set of AD credentials is Open Source Intelligence (OSINT) and Phishing.

# NTLM (New Technology LAN Manager):

~ Suite of security protocols used to authenticate users' identities in AD.

~ NTLM can be used for authentication by using a challenge-response-based scheme called NetNTLM.

~ Heavily used by services on a network.

~ NetNTLM, also often referred to as Windows Authentication or just NTLM Authentication.

~ Allows the application to play the role of a middle man between the client and AD. All authentication material is forwarded to a Domain Controller in the form of a challenge, and if completed successfully, the application will authenticate the user.

This means that the application is authenticating on behalf of the user and not authenticating the user directly on the application itself. This prevents the application from storing AD credentials, which should only be stored on a DC.

![image](https://github.com/user-attachments/assets/ab62dc18-43b4-4bb4-842f-0bff8494c12a)
---

# LDAP (Lightweight Directory Access Protocol):

~ LDAP authentication is similar to NTLM authentication. However, with LDAP authentication, the application directly verifies the user's credentials. The application has a pair of AD credentials that it can use first to query LDAP and then verify the AD user's credentials.

~ A popular mechanism with third-party (non-Microsoft) applications that integrate with AD. These include applications and systems such as:
  Gitlab,  Jenkins,  Custom-developed web applications,  Printers,  VPNs.

![image](https://github.com/user-attachments/assets/4e54b4f1-6094-4cf2-b5a9-adcbbbc42411)


# LDAP Pass-back:

LDAP Pass-back attack is a common attack against network devices, such as printers, when you have gained initial access to the internal network, such as plugging in a rogue device in a boardroom.


LDAP Pass-back attacks can be performed when we gain access to a device's configuration where the LDAP parameters are specified. This can be, for example, the web interface of a network printer. Usually, the credentials for these interfaces are kept to the default ones, such as admin:admin or admin:password. Here, we won't be able to directly extract the LDAP credentials since the password is usually hidden. However, we can alter the LDAP configuration, such as the IP or hostname of the LDAP server. In an LDAP Pass-back attack, we can modify this IP to our IP and then test the LDAP configuration, which will force the device to attempt LDAP authentication to our rogue device. We can intercept this authentication attempt to recover the LDAP credentials.





