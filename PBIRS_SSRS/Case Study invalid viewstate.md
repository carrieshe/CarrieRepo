# Case Study: Invalid View State in PBIRS

## Symptom

When using Power BI Report Server (PBIRS) in a scale-out or network load balancing (NLB) environment, users may encounter intermittent errors when interacting with paginated reports. Typically, the report loads successfully at first, but after several minutes, further interactions (such as refreshing or viewing the report) result in the following exception:

```
Sys.WebForms.PageRequestManagerServerErrorException: An unknown error occurred while processing the request on the server. The status code returned from the server was: 500
```

## Root Cause

This issue is commonly caused invalid viewstate in NLB deployment. In an NLB cluster scenario, there are multiple service instances and web service identities that run on different computers. Because the service identity varies for each node, you can't rely on a single process identity to perform the validation. To work around this issue, we can generate an arbitrary validation key to support view state validation, and then manually configure each report server node to use the same key. typically we could notice below exception in the PBIRS log files.

```
System.Web.HttpException (0x80004005): Validation of viewstate MAC failed. If this application is hosted by a Web Farm or cluster, ensure that <machineKey> configuration specifies the same validationKey and validation algorithm. AutoGenerate cannot be used in a cluster.
System.Web.UI.ViewStateException: Invalid viewstate.
```

## Resolution Steps

### 1. Generate a Consistent Machine Key

On one PBIRS server, use the following PowerShell script to generate a `<machineKey>` element. This key must be used on all PBIRS nodes in the deployment.

```powershell
function Generate-MachineKey {
  [CmdletBinding()]
  param (
    [ValidateSet("AES", "DES", "3DES")]
    [string]$decryptionAlgorithm = 'AES',
    [ValidateSet("MD5", "SHA1", "HMACSHA256", "HMACSHA384", "HMACSHA512")]
    [string]$validationAlgorithm = 'SHA1'
  )
  process {
    function BinaryToHex {
        [CmdLetBinding()]
        param($bytes)
        process {
            $builder = new-object System.Text.StringBuilder
            foreach ($b in $bytes) {
              $builder = $builder.AppendFormat([System.Globalization.CultureInfo]::InvariantCulture, "{0:X2}", $b)
            }
            $builder
        }
    }
    switch ($decryptionAlgorithm) {
      "AES" { $decryptionObject = new-object System.Security.Cryptography.AesCryptoServiceProvider }
      "DES" { $decryptionObject = new-object System.Security.Cryptography.DESCryptoServiceProvider }
      "3DES" { $decryptionObject = new-object System.Security.Cryptography.TripleDESCryptoServiceProvider }
    }
    $decryptionObject.GenerateKey()
    $decryptionKey = BinaryToHex($decryptionObject.Key)
    $decryptionObject.Dispose()
    switch ($validationAlgorithm) {
      "MD5" { $validationObject = new-object System.Security.Cryptography.HMACMD5 }
      "SHA1" { $validationObject = new-object System.Security.Cryptography.HMACSHA1 }
      "HMACSHA256" { $validationObject = new-object System.Security.Cryptography.HMACSHA256 }
      "HMACSHA385" { $validationObject = new-object System.Security.Cryptography.HMACSHA384 }
      "HMACSHA512" { $validationObject = new-object System.Security.Cryptography.HMACSHA512 }
    }
    $validationKey = BinaryToHex($validationObject.Key)
    $validationObject.Dispose()
    [string]::Format([System.Globalization.CultureInfo]::InvariantCulture,
      "<machineKey validationKey=`"{3}`" decryptionKey=`"{1}`" validation=`"{2}`" decryption=`"{0}`"/>",
      $decryptionAlgorithm.ToUpperInvariant(), $decryptionKey,
      $validationAlgorithm.ToUpperInvariant(), $validationKey)
  }
}
Generate-MachineKey
```

### 2. Update RSReportServer.config on All Nodes

- Open `RSReportServer.config` (typically located at `C:\Program Files\Microsoft Power BI Report Server\PBIRS\ReportServer\RSReportServer.config`).
- In the `<Configuration>` section, paste the generated `<machineKey>` element.
- Example:
  ```xml
  <machineKey ValidationKey="[your key here]" DecryptionKey="[your key here]" Validation="SHA1" Decryption="AES"/>
  ```
- Save the file and restart the PBIRS service.



### 3. Ensure Consistency Across All Servers

Repeat step 2 for every PBIRS node in the scale-out or NLB deployment.  
**All nodes must use the exact same `<machineKey>` configuration.**

For further details, refer to:  
[Configure a report server on a network load balancing cluster - SQL Server Reporting Services (SSRS) | Microsoft Learn](https://learn.microsoft.com/en-us/sql/reporting-services/report-server/configure-a-report-server-on-a-network-load-balancing-cluster?view=sql-server-ver16#ViewState)


