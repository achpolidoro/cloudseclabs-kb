# Inventariar Sites e Bibliotecas de Documentos no SharePoint Online

> **Objetivo:** Coletar todas as bibliotecas de documentos (*Document Libraries*), quantidade de itens, IDs e volume ocupado (GB) em múltiplos sites do SharePoint Online, exportando para CSV.

## 1. Script com PnP.PowerShell (PowerShell 7+)

```powershell
$ErrorActionPreference = "Stop"
$startTime = Get-Date
$timestamp = $startTime.ToString("yyyyMMdd_HHmmss")

$outputFolder = "C:\Temp\SharePointReports"
$outputFile = "$outputFolder\Bibliotecas_SharePoint_$timestamp.csv"
$logFile = "$outputFolder\Bibliotecas_SharePoint_$timestamp.log"

if (-not (Test-Path -Path $outputFolder)) {
    New-Item -ItemType Directory -Path $outputFolder | Out-Null
}

Start-Transcript -Path $logFile -Append

# 1. Conectar ao SharePoint Admin Center
$adminUrl = "https://seu-tenant-admin.sharepoint.com"
Connect-PnPOnline -Url $adminUrl -Interactive

# 2. Obter lista dinâmica de sites (ou especificar URLs manualmente)
$allSites = (Get-PnPTenantSite | Where-Object { $_.Template -notlike "SRCHCEN*" -and $_.Template -notlike "SPSMSITEHOST*" }).Url

"SiteURL,SiteTitle,LibraryTitle,LibraryID,ItemCount,LastModified,LibrarySize(GB)" |
    Out-File -FilePath $outputFile -Encoding UTF8

$totalSites = $allSites.Count
$processedSites = 0

foreach ($siteUrl in $allSites) {
    $processedSites++
    try {
        $connection = Connect-PnPOnline -Url $siteUrl -Interactive -ReturnConnection -ErrorAction Stop
        $web = Get-PnPWeb -Connection $connection -Includes Title
        $siteTitle = $web.Title

        $libraries = Get-PnPList -Connection $connection | Where-Object {
            $_.BaseTemplate -eq 101 -and $_.Hidden -eq $false -and $_.Title -ne "Formulários"
        }

        foreach ($library in $libraries) {
            $fullLibrary = Get-PnPList -Identity $library.Id -Connection $connection -Includes ItemCount, LastItemModifiedDate, RootFolder
            $folder = Get-PnPFolder -Url $fullLibrary.RootFolder.ServerRelativeUrl -Connection $connection -Includes StorageMetrics
            $sizeGB = [math]::Round($folder.StorageMetrics.TotalSize / 1GB, 2)

            "$siteUrl,$siteTitle,$($fullLibrary.Title),$($fullLibrary.Id),$($fullLibrary.ItemCount),$($fullLibrary.LastItemModifiedDate),$sizeGB" |
                Out-File -FilePath $outputFile -Encoding UTF8 -Append
        }
    }
    catch {
        "$siteUrl,ERRO AO ACESSAR SITE,,,,," | Out-File -FilePath $outputFile -Encoding UTF8 -Append
    }
    finally {
        if ($connection) { Disconnect-PnPOnline -Connection $connection }
    }
}

Stop-Transcript
```

## 2. Consulta Rápida de Uso de Armazenamento (`Microsoft.Online.SharePoint.PowerShell`)

```powershell
Connect-SPOService -Url "https://seu-tenant-admin.sharepoint.com"
Get-SPOSite -Limit All |
    Select-Object Url, Title, StorageUsageCurrent, StorageQuota, LastContentModifiedDate |
    Format-Table -AutoSize
```
