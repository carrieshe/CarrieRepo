# Fabric Scanner API Usage Guide

## Introduction

Metadata scanning enables governance of your organization's Microsoft Fabric data by cataloging and reporting on all metadata for your Fabric items. This is achieved using a set of Admin REST APIs collectively known as the scanner APIs.  
Refer to: [Metadata scanning overview - Microsoft Fabric | Microsoft Learn](https://learn.microsoft.com/en-us/fabric/governance/metadata-scanning-overview)

---

## How to Run the Scanner API

1. **Retrieve all workspaces in the tenant**  
   Use the [Admin - WorkspaceInfo GetModifiedWorkspaces](https://learn.microsoft.com/en-us/rest/api/power-bi/admin/workspace-info-get-modified-workspaces) API.

2. **Scan workspaces**  
   Pass the workspace list as the request body to the [Admin - WorkspaceInfo PostWorkspaceInfo](https://learn.microsoft.com/en-us/rest/api/power-bi/admin/workspace-info-post-workspace-info) API to initiate a scan.

3. **Check scan status**  
   Use the scan ID returned in the previous step and call [Admin - WorkspaceInfo GetScanStatus](https://learn.microsoft.com/en-us/rest/api/power-bi/admin/workspace-info-get-scan-status) to monitor scan progress.

4. **Retrieve scan results**  
   Once the scan is complete, call [Admin - WorkspaceInfo GetScanResult](https://learn.microsoft.com/en-us/rest/api/power-bi/admin/workspace-info-get-scan-result) to obtain the scan results.

---

## Demo: Sample PowerShell Script

Below is a sample PowerShell script that performs the full scan process and saves the results to a JSON file. The script accepts service principal authentication parameters.

To run the script:
```powershell
.\GetScanResult.ps1 -ClientId "XXXX" -ClientSecret "XXXX" -TenantId "XXXX"
```

**GetScanResult.ps1**
```powershell
param(
    [Parameter(Mandatory=$true)]
    [string]$ClientId,
    [Parameter(Mandatory=$true)]
    [string]$ClientSecret,
    [Parameter(Mandatory=$true)]
    [string]$TenantId
)
# 1. Get Access Token
$body = @{
    grant_type    = "client_credentials"
    client_id     = $ClientId
    client_secret = $ClientSecret
    scope         = "https://analysis.windows.net/powerbi/api/.default"
}
$tokenResponse = Invoke-RestMethod -Method Post -Uri "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token" -Body $body
$accessToken = $tokenResponse.access_token
# 2. Call GetModifiedWorkspaces API
$headers = @{
    Authorization = "Bearer $accessToken"
}
$modifiedWorkspacesUrl = "https://api.powerbi.com/v1.0/myorg/admin/workspaces/modified"
$modifiedWorkspacesResponse = Invoke-RestMethod -Method Get -Uri $modifiedWorkspacesUrl -Headers $headers
# 3. Prepare request body for PostWorkspaceInfo API
$workspaceIds = @()
if ($null -ne $modifiedWorkspacesResponse -and $modifiedWorkspacesResponse.Count -gt 0) {
    foreach ($ws in $modifiedWorkspacesResponse) {
        if ($ws.id) {
            $workspaceIds += $ws.id
        }
    }
} else {
    Write-Host "No modified workspaces found."
}
if ($workspaceIds.Count -eq 0) {
    Write-Host "No workspace IDs to send. Exiting."
    return
}
$bodyPost = @{ workspaces = $workspaceIds } | ConvertTo-Json
# 4. Call PostWorkspaceInfo API to start scan
$postWorkspaceInfoUrl = "https://api.powerbi.com/v1.0/myorg/admin/workspaces/getInfo"
$postResponse = Invoke-RestMethod -Method Post -Uri $postWorkspaceInfoUrl -Headers $headers -Body $bodyPost -ContentType "application/json"
# 5. Get scan ID from response
$scanId = $postResponse.id
if ($scanId) {
    Write-Host "Scan ID is: $scanId"
} else {
    Write-Host "No scan ID returned. Exiting."
    return
}
# 6. Poll GetScanStatus API until scan is complete
$scanStatusUrl = "https://api.powerbi.com/v1.0/myorg/admin/workspaces/scanStatus/$scanId"
$maxAttempts = 30
$attempt = 0
$delaySeconds = 10
$scanStatus = "NotStarted"
do {
    try {
        $scanStatusResponse = Invoke-RestMethod -Method Get -Uri $scanStatusUrl -Headers $headers
        $scanStatus = $scanStatusResponse.status
        Write-Host "Scan status from scanStatus API: $scanStatus"
        if ($scanStatus -eq "Succeeded" -or $scanStatus -eq "Failed") {
            break
        }
    } catch {
        if ($_.Exception.Response.StatusCode.value__ -eq 404) {
            Write-Host "Scan status not ready yet (404). Waiting..."
        } else {
            Write-Host "Error polling scan status:"
            Write-Host $_.Exception.Message
            break
        }
    }
    Start-Sleep -Seconds $delaySeconds
    $attempt++
} while ($attempt -lt $maxAttempts)
if ($scanStatus -eq "Succeeded") {
    # 7. Get scan results only if succeeded
    $scanResultUrl = "https://api.powerbi.com/v1.0/myorg/admin/workspaces/scanResult/$scanId"
    $scanResult = Invoke-RestMethod -Method Get -Uri $scanResultUrl -Headers $headers
    # Use createdDateTime from scanResult or current time if not available
    $createdTime = $scanResult.createdDateTime
    if (-not $createdTime) {
        $createdTime = Get-Date
    } else {
        $createdTime = [datetime]$createdTime
    }
    $timestamp = $createdTime.ToString("yyyyMMdd_HHmmss")
    $jsonFile = ".\ScanResult_$timestamp.json"
    $scanResult | ConvertTo-Json -Depth 10 | Out-File -Encoding utf8 $jsonFile
    Write-Host "raw scan result has been saved in file $jsonFile"
    return
} elseif ($scanStatus -eq "Failed") {
    Write-Host "Scan failed."
    return
} else {
    Write-Host "Scan did not complete in allotted time."
    return
}
```
