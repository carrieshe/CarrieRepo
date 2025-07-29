# Downgrade PBIRS

Power BI Report Server (PBIRS) does not support in-place downgrade, but in-place upgrade is supported, refer to [Upgrade and migrate Reporting Services - SQL Server Reporting Services (SSRS) | Microsoft Learn](https://learn.microsoft.com/en-us/sql/reporting-services/install-windows/upgrade-and-migrate-reporting-services?view=sql-server-ver17）). To downgrade PBIRS, we must back up your environment, uninstall the current version, and then install the desired lower version.

---

## Step-by-Step Downgrade Procedure

### 1. Backup Before Downgrade

Take the following backups to prevent data loss or unexpected issues:

- **Encryption Keys:**  
  Back up the encryption keys of your current PBIRS. Refer to: [Back up and restore SQL Server Reporting Services (SSRS) encryption keys - SQL Server Reporting Services (SSRS) | Microsoft Learn](https://learn.microsoft.com/en-us/sql/reporting-services/install-windows/ssrs-encryption-keys-back-up-and-restore-encryption-keys?view=sql-server-ver16)

- **Databases:**  
  Backup the `ReportServer` and `ReportServerTempDB` databases.

- **Configuration Files:**  
  | File Name                                      | Default Location                                                                                   |
  |------------------------------------------------|----------------------------------------------------------------------------------------------------|
  | Rssvrpolicy.config                             | C:\Program Files\Microsoft Power BI Report Server\PBIRS\ReportServer                               |
  | Rsreportserver.config                          | C:\Program Files\Microsoft Power BI Report Server\PBIRS\ReportServer                               |
  | Web.config for the Report Server ASP.NET app   | C:\Program Files\Microsoft Power BI Report Server\PBIRS\ReportServer                               |
  | ReportingServicesService.exe.config            | C:\Program Files\Microsoft Power BI Report Server\PBIRS\ReportServer\bin                           |
  | Machine.config for ASP.NET                     | %windir%\Microsoft.NET\Framework64\[version]\config\machine.config                                 |


### 2. Uninstall and Reinstall PBIRS

- Uninstall the current version of PBIRS.
- Install the desired lower version of PBIRS.


### 3. Restore Databases

- Restore the `ReportServer` and `ReportServerTempDB` databases **that you backed up before the upgrade**.
  > **Important:** Restoring the correct backup is critical; otherwise, the database schema version may be higher than the service schema version, causing compatibility issues.


### 4. Reconfigure PBIRS

- Use Report Server Configuration Manager to:
  - Set the database to the restored one.
  - Reapply other configuration settings as previously set.

### 5. Restore Encryption Keys

- Restore the encryption keys **that you backed up before the upgrade**.


### 6. Validate

- Open the PBIRS web portal URL.
- Run a report to confirm successful rendering.

---
