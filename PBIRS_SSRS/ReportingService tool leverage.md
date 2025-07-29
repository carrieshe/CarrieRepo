# Leveraging ReportingServicesTools PowerShell Module

This guide demonstrates how to use the [ReportingServicesTools PowerShell module](https://github.com/microsoft/ReportingServicesTools) to automate the management against PBIRS as well as SSRS. Several examples such as managing permissions, gettting cache refresh plans, and subscriptions are shared below.

---

## 1. Install and Import the Module

To get started, install and import the ReportingServicesTools module:

```powershell
Invoke-Expression (Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/ReportingServicesTools/master/Install.ps1)
Install-Module -Name ReportingServicesTools -RequiredVersion 0.0.6.4

# Import the module if installed manually
# Import-Module C:\RStools\ReportingServicesTools\ReportingServicesTools
```


## 2. Example a- Grant User Permissions

You can grant catalog item permissions to users using PowerShell:

```powershell
# Specify credentials
$cred = Get-Credential -Message 'Enter your credentials'

# Grant access to a catalog item
Grant-AccessOnCatalogItem -Identity 'dongshe\catest01' -RoleName 'Content Manager' -Path '/CATEST/sub_1/sub_2' -ReportServerUri 'http://vm7/ReportServer' -Credential $cred
```

---

## 3. Example b- Get Cache Refresh Plans and Subscriptions

### For Power BI Reports

Retrieve cache refresh plans for Power BI reports:

```powershell
Get-RsRestCacheRefreshPlan -RsReport "/SQL_TIMEZONE_import"
Get-RsRestCacheRefreshPlan -RsReport "/SQL_TIMEZONE_import" -ReportPortalUri "http://client/Reports"
```

### For RDL Reports

Retrieve subscription details for paginated (RDL) reports:

```powershell
Get-RsSubscription -RsItem "/CountrySalesPerformance"
Get-RsSubscription -ReportServerUri 'http://client/reportserver' -RsItem '/CountrySalesPerformance'
```

---

By leveraging these PowerShell cmdlets, you can efficiently automate report server management tasks, including permission assignment, cache refresh monitoring, and subscription auditing.
