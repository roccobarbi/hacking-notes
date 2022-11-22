# Active Directory: breaching the perimeter

## TL;DR

The first step for breaching an AD environment is a lot of OSINT. In particular, look for leaked credentials for users from the AD network: even if they are for other services, they may have reused them on AD (maybe with some minor and obvious change) and they could give you a foothold into the system. Remember that there's always a good chance that, if the admins force periodic password changes, users may simply add the year and month, or the year and a progressive number, at the end of their favourite password. Try to poke at the perimeter of the network and see if there's some less secure internet-facing service that you can breach more easily: chances are that it will have some configuration file with a set of AD credentials, or that it will let you tunnel into the network, where you can proceed with lateral movement and privesc. If you can get phisical access to the network (e.g. via a network port in some accessible room without strong NAC in place), see if you can reach some less-secure service (e.g. a printer) that you can leverage to obtain valid AD credentials.

## OSINT and Phishing

These techniques can't easily be practiced against a lab environment, but they're often the most effective ways that a hacker can gain the initial foothold into an AD network.

OSINT helps by uncovering breached credentials, which could be used either directly (breached company credentials that haven't been changed) or indirectly (breached credentials that reveal a password that an employee reused on the company's network).

It could also yield leaked credentials (e.g. on github repositories, issues, stackoverflow questions...), default credentials for new users (which may or may not have changed them), lists of usernames or other usable information on attackable systems that may provide direct access to the AD network.

## NetNTLM

NetNTLM is a challenge-response system that is based on the NTLM authentication protocol. It is often used by applications that are exposed to the web, such as self-hosted exchange servers with OWA authentication pages.

NetNTLM can be attacked with password spraying. Let's assume, for example, that you have uncovered a default password for new users during information gathering: you can write a script to test that password on a list of usernames. This method may trigger an alert on the target due to the high number of failed login attempts, but it also may not (and it should not alert individual users by locking them out) because each user will have failed to login only once. It could help if the script isn't too fast, ideally with varying times between attempts and using (e.g. through proxies) different ips.

NetNTLM can also be used to test breached/leaked credentials to check if they're still valid. The same caveats as above apply.

## LDAP

LDAP is often used by non-Microsoft applications that need to integrate with Active Directory. LDAP applications often receive credentials from individual users, then use a different set of credentials of their own to contact AD and authenticate the user. In other cases (e.g. network printers), they simply use their own credentials to connect to the network, then they receive jobs/connections/data from the rest of the network.

If an endpoint that uses LDAP can be hacked into, it is often possible to find AD credentials in cleartext in some config file.

### LDAP Passback Attacks

A special case is that of network printers that connect to AD networks using LDAP. Such printers often have accessible/default configuration pages that can be leveraged to access their AD username directly, and their AD password using a technique called LDAP Passback.

In a few words, the technique consists of setting up an endpoint on the attacking machine, then pointing the printer configuration to contact it instead of the lDAP server, thus receiving the password.

Depending on how the printer connects to LDAP, it may be possible to leverage this technique:
- using a simple netcat endpoint;
- using openldap or a similar tool to simulate a full LDAP environment;
- scripting a custom endpoint.