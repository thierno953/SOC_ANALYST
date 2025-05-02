# SOC ANALYST

```sh
Mount-DiskImage C:\Users\exchangeadmin\Desktop\ExchangeServer2019-x64-CU12.ISO
Get-Volume
D:\Setup.EXE /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareSchema
D:\Setup.EXE /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /PrepareAD /OrganizationName:"barry ca"
Dismount-DiskImage C:\Users\exchangeadmin\Desktop\ExchangeServer2019-x64-CU12.ISO
Install-WindowsFeature Web-Server -IncludeManagementTools

Mount-DiskImage C:\Users\exchangeadmin\Desktop\ExchangeServer2019-x64-CU12.ISO
Get-Volume
D:\Setup.EXE /Mode:Install /IAcceptExchangeServerLicenseTerms_DiagnosticDataON /Roles:Mailbox /InstallWindowsComponents
Restart-Computer -Force
LaunchEMS
```