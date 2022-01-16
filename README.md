# LDAP Relay Scan 
A tool to check Domain Controllers for LDAP server protections regarding the relay of NTLM authentication. If you're interested in the specifics of the error-based enumeration, see [below](blah).
## Summary
There are unique, server-side protections when attempting to relay NTLM authentication LDAP on Domain Controllers. The LDAP protections this tools attempts to enumerate include:
 - LDAPS - [channel binding](https://support.microsoft.com/en-us/topic/use-the-ldapenforcechannelbinding-registry-entry-to-make-ldap-authentication-over-ssl-tls-more-secure-e9ecfa27-5e57-8519-6ba3-d2c06b21812e)
 - LDAP - [server signing requirements](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/domain-controller-ldap-server-signing-requirements)

The enforcement of channel binding for LDAP over SSL/TLS can be determined from an **unauthenticated** perspective. This is because the error associated with an LDAP client lacking the ability to conduct channel binding properly will occur before credentials are validated during the LDAP bind process. 

However, to determine if the server-side protection of standard LDAP is enforced, server signing integrity requirements, the clients credentials must first be validated during the LDAP bind. The potential error identifying the enforcement of this protection is identified from an **authenticated** perspective.



#### TL;DR - LDAPS can be checked unauthenticated, but checking LDAP requires authentication.

## Usage

The tool has two methods, **LDAPS** (the default), and **BOTH**. LDAPS only requires a domain controller IP address, because this check can be preformed unauthenticated. The BOTH method will require a username and password or NT hash. The Active Directory domain is not required, it will be determine via anonymous LDAP bind.

## Examples

```
python3.9 LdapRelayScan.py -method LDAPS -dc-ip 10.0.0.20
python3.9 LdapRelayScan.py -method BOTH -dc-ip 10.0.0.20 -u domainuser1 
python3.9 LdapRelayScan.py -method BOTH -dc-ip 10.0.0.20 -u domainuser1 -p badpassword2
python3.9 LdapRelayScan.py -method BOTH -dc-ip 10.0.0.20 -u domainuser1 -nthash e6ee750a1feb2c7ee50d46819a6e4d25
```

## Error-Based Enumeration Specifics

### LDAP - Server Signing Requirements
On a Domain Controller, the policy called ```Domain Controller: LDAP server signing requirements``` is set to `None`, `Require signing`, or it's just not defined. When not defined, it defaults to not requiring signing (at the time of writing this). The error which identifies this protection as required is when a [sicily NTLM](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/e7d814a5-4cb5-4b0d-b408-09d79988b550) or [simple](https://ldapwiki.com/wiki/Simple%20Authentication) bind attempt responds with a [resultCode of 8](https://ldap.com/ldap-result-code-reference-core-ldapv3-result-codes/#rc-strongerAuthRequired), signifying `strongerAuthRequired`. This will only occur if credentials during the LDAP bind are validated. 

*INSERT IMAGE*

### LDAPS - Channel Binding Token Requirements
On a Domain Controller that has been patched since [CVE-2017-8563](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2017-8563) the capability to enforce LDAPS channel binding has existed. The specific policy is called `Domain Controller: LDAP server channel binding token requirements` and can be set to either `Never`, `When supported`, or `Always`. This is also [not required by default](https://msrc.microsoft.com/update-guide/en-us/vulnerability/ADV190023) (at the time of writing this). 
