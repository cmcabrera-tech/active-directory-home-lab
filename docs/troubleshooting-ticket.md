# Troubleshooting Ticket

## Ticket Summary

**Title:** CLIENT01 cannot resolve the internal Active Directory domain  
**Category:** Network / DNS  
**Priority:** Medium  
**Affected system:** CLIENT01  
**Domain:** cabreralab.test  

## Reported Issue

The user could not access domain resources from CLIENT01. A lookup of the
internal domain failed.

## Investigation

1. Confirmed that CLIENT01 had network connectivity to DC01.
2. Reviewed the client configuration with `ipconfig /all`.
3. Identified that CLIENT01 was using a public DNS server instead of the domain
   controller.
4. Verified that the public DNS server could not resolve
   `cabreralab.test`.

## Root Cause

Active Directory relies on its internal DNS namespace and service records.
CLIENT01 was pointed to a public DNS resolver that had no records for the
private domain.

## Resolution

1. Returned the DNS setting to automatic configuration.
2. Confirmed that DHCP supplied `10.10.10.10` as the DNS server.
3. Flushed the client DNS cache.
4. Released and renewed the DHCP lease.
5. Verified successful resolution of `cabreralab.test`.

```powershell
ipconfig /flushdns
ipconfig /release
ipconfig /renew
nslookup cabreralab.test
```

## Result

CLIENT01 successfully resolved the internal domain and regained access to
domain resources. The ticket was closed after validation.

