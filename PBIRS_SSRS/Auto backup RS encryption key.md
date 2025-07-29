# Automatically Backup PBIRS Encryption Key

This guide explains how to automate the backup of the Power BI Report Server (PBIRS) encryption key using the `rskeymgmt` tool and Windows scheduled tasks.

---

## Step 1: Create the Backup Script

Write a batch file (e.g., `backup_encryptionkey.bat`) to back up the encryption key.

Reference: [Back Up and Restore Reporting Services Encryption Keys - SQL Server Reporting Services (SSRS) | Microsoft Learn](https://learn.microsoft.com/en-us/sql/reporting-services/install-windows/ssrs-encryption-keys-back-up-and-restore-encryption-keys?view=sql-server-ver16)

```bat
@echo off
echo Y | "C:\Program Files\Microsoft Power BI Report Server\Shared Tools\rskeymgmt.exe" -e -f C:\Backup\SSRSEncKey.snk -p YourStrongPassword
```

- `"C:\Program Files\Microsoft Power BI Report Server\Shared Tools"` is the default location for `rskeymgmt.exe`.
- `-e` extracts the encryption key.
- `-f` specifies the backup file path.
- `-p` sets the password for the backup file.
- Replace `C:\Backup\SSRSEncKey.snk` and `YourStrongPassword` with your desired file path and a strong password.


## Step 2: Schedule the Backup Task

Create another batch file (e.g., `scheduletask.bat`) to schedule the backup script to run automatically (e.g., every minute).

```bat
@echo off
schtasks /create /sc minute /mo 1 /tn "RunBatchFile" /tr "C:\backup_encryptionkey.bat"
```

- Replace `C:\backup_encryptionkey.bat` with the actual path to your backup script.

---

By following these steps, you can ensure regular, automated backups of your PBIRS encryption key for disaster recovery and compliance.
