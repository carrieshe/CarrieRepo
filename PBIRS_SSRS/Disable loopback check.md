# Disable Loopback Check for PBIRS

When the virtual server name differs from the actual server name, you may encounter a 401 error when authenticating to the virtual server's HTTPS RS URL. Disabling the loopback check is necessary in such scenarios, especially when configuring network load balancers (NLB) where the NLB URL  probably differs from the actual server name.

---

## Methods to Disable Loopback Check

### Method 1 (Recommended)

1. **Set DisableStrictNameChecking registry entry to 1**
   - Registry path:  
     `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\LanmanServer\Parameters`
   - Value name: `DisableStrictNameChecking`
   - Data type: `REG_DWORD`
   - Base: `Decimal`
   - Value: `1`

2. **Create Local Security Authority host names for NTLM authentication**
   - Registry path:  
     `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\MSV1_0`
   - Value name: `BackConnectionHostNames`
   - Data type: `Multi-String`
   - Value:  
     - VirtualServerFQDNName  
     - ActualServerFQDNName  
     - ActualServerShortName

---

### Method 2

1. **Set DisableStrictNameChecking registry entry to 1**
   - Registry path:  
     `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\LanmanServer\Parameters`
   - Value name: `DisableStrictNameChecking`
   - Data type: `REG_DWORD`
   - Base: `Decimal`
   - Value: `1`

2. **Disable loopback check**
   - Registry path:  
     `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa`
   - Value name: `DisableLoopbackCheck`
   - Data type: `REG_DWORD`
   - Base: `Decimal`
   - Value: `1`

---

**Note:**  You must restart the server for these changes to take effect.

For more details, refer to:  
[Error message when you try to access a server locally by using its FQDN or its CNAME alias after you install Windows Server 2003 Service Pack 1 - Windows Server | Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/accessing-server-locally-with-fqdn-cname-alias-denied)
