# SharePoint Admin / Owners Cleanup

This repository contains a PowerShell script that automatically removes specific accounts from administrator and owner roles on SharePoint Online sites in a Microsoft 365 tenant.

## Purpose

*   Scan all SharePoint sites in the tenant.
*   Detect listed accounts that are site collection admins or owners.
*   Automatically remove them from:
    *   Site Collection Admins
    *   SharePoint Owners groups
    *   Microsoft 365 Group owners (Teams-connected sites)

## Prerequisites

*   PowerShell 5.1 or PowerShell 7+
*   PnP.PowerShell module installed:
    
    Copy
    
    `Install-Module PnP.PowerShell -Scope CurrentUser`
    

*   An App Registration with certificate in Azure AD, with permissions to:
    *   Administer SharePoint Online
    *   Manage sites and Microsoft 365 groups
*   The certificate installed on the machine running the script (the thumbprint is used).

## Main parameters

Update the following variables in the script:

Copy

`$TenantId = "<YOUR_TENANT_GUID>"$ClientId = "<APP_REGISTRATION_CLIENT_ID>" $Thumbprint = "<CERTIFICATE_THUMBPRINT>"$TenantAdminUrl = "https://<yourTenant>-admin.sharepoint.com" $TargetAccounts = @( "account1@domain.com", "account2@domain.com" \# ... )$ExportPath = "C:\Temp\Actions-SharePoint-Admins-Owners.csv" # Simulation / Real execution $WhatIf =$false # $true = simulation,$false = real removal`

Key settings:

*   <span class='agent\_key'>$TargetAccounts</span>: list of accounts to remove.
*   <span class='agent\_key'>$WhatIf</span>:
    *   $true: simulation mode (no changes, only logs).
    *   $false: real removal of permissions.

## How it works

1.  Connects to the SharePoint Admin Center using Connect-PnPOnline (App Registration + certificate).
2.  Retrieves all sites:
    
    Copy
    
    `$Sites = Get-PnPTenantSite -IncludeOneDriveSites:$false`
    
3.  For each site:
    *   Connects to the site.
    *   Checks Site Collection Admins via Get-PnPSiteCollectionAdmin.
    *   Checks SharePoint Owners groups (groups with "Owners" in the title).
    *   Checks Microsoft 365 Group owners (Teams) if the site is group-connected.
4.  For each match found in <span class='agent\_key'>$TargetAccounts</span>:
    *   Writes a message to the console.
    *   Removes the account if <span class='agent\_key'>$WhatIf</span> -eq $false.
    *   Logs the action into $Results.
5.  Exports all actions to CSV:
    
    Copy
    
    `$Results | Export-Csv$ExportPath -NoTypeInformation -Encoding UTF8`
    

## Manual run

1.  Open PowerShell with appropriate permissions.
2.  Adjust parameters in the script (tenant, app, certificate, target accounts, export path).
3.  Run the script:
    
    Copy
    
    `.\SharePoint-Admins-Owners-Cleanup.ps1`
    

Start with:

Copy

`$WhatIf =$true`

to validate behavior without impacting production.

## Scheduled execution

To run the script weekly:

1.  Copy the script to an admin workstation or server (for example C:\\Scripts\\sp-admins-owners-cleanup.ps1).
2.  Create a Windows Task Scheduler job:
    *   Trigger: Weekly.
    *   Action: powershell.exe
    *   Arguments:
        
        Copy
        
        `-ExecutionPolicy Bypass -File "C:\Scripts\sp-admins-owners-cleanup.ps1"`
        
3.  Make sure the task’s run-as account:
    *   Has access to the certificate used by the App Registration.
    *   Has access to the script file and export path.

## Logging and export

The script generates a CSV file containing all actions performed (or simulated):

*   SiteUrl
*   Role
*   Account
*   Group (if applicable)
*   Action (for example Removed)

This file provides a trace of removed administrator and owner permissions.

## Warnings

*   Always test with <span class='agent\_key'>$WhatIf = $true</span> on a pilot or non-production scope first.
*   Regularly review <span class='agent\_key'>$TargetAccounts</span> to avoid removing legitimate permissions.
*   Misconfigured App Registration or certificate will prevent successful connection to SharePoint Online.

## Customization

You can customize:

*   The Owners group filter:
    
    Copy
    
    `$OwnerGroups = Get-PnPGroup | Where-Object {$_.Title -match "Owners" }`
    
    (for example change to \-like "\*Owners" if needed)
*   The site scope (Get-PnPTenantSite) if you want to exclude specific templates or URLs.
*   The export format or CSV destination.
