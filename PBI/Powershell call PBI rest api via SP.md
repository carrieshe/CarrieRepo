# Calling Power BI REST API via PowerShell and Service Principal

In addition to using client tools such as Power Query or Postman, you can also invoke the Power BI REST API using PowerShell. Below is a sample PowerShell script demonstrating how to authenticate with a service principal and call the API.

---

## Sample PowerShell Script

```powershell
# Install the Power BI Management module (run as administrator if needed)
Install-Module -Name MicrosoftPowerBIMgmt

# Define the service principal credentials
$tenantId = "your-tenant-id"
$clientId = "your-client-id"
$clientSecret = "your-client-secret"

# Create a PSCredential object
$secureClientSecret = ConvertTo-SecureString $clientSecret -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($clientId, $secureClientSecret)

# Authenticate to Power BI using the service principal
Connect-PowerBIServiceAccount -ServicePrincipal -Credential $credential -Tenant $tenantId

# Define the API endpoint
$url = "https://api.powerbi.com/v1.0/myorg/groups"

# Invoke the REST API
$response = Invoke-PowerBIRestMethod -Url $url -Method Get

# Convert the response to JSON and output
$response | ConvertTo-Json
```

---

*This approach allows for automation and scripting of Power BI REST API operations using PowerShell and service principal authentication.*
