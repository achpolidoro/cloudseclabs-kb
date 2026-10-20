# Inventariar Sites e Bibliotecas do SharePoint Online

> **Caminho:** `modern-working/sharepoint-teams/inventariar-bibliotecas-spo.md`  
> **Objetivo:** Exportar para CSV a relação completa de sites do SharePoint Online, suas bibliotecas de documentos, IDs, quantidade de arquivos e volume ocupado em GB.

---

## 1. Pré-requisitos

- **PowerShell 7+** com módulo `PnP.PowerShell` instalado.
- Permissão de **SharePoint Administrator**.

---

## 2. Passo a passo

```powershell
$ErrorActionPreference = "Stop"
$timestamp = (Get-Date).ToString("yyyyMMdd_HHmmss")
$outputFolder = "C:\Temp\SharePointReports"
$outputFile = "$outputFolder\Bibliotecas_SharePoint_$timestamp.csv"

if (-not (Test-Path $outputFolder)) {
    New-Item -ItemType Directory -Path $outputFolder | Out-Null
}

# 1. Conectar ao SharePoint Admin Center
$adminUrl = "https://seu-tenant-admin.sharepoint.com"
Connect-PnPOnline -Url $adminUrl -Interactive

# 2. Obter todos os sites ativos do tenant dinamicamente
$allSites = (Get-PnPTenantSite | Where-Object {
    $_.Template -notlike "SRCHCEN*" -and $_.Template -notlike "SPSMSITEHOST*"
}).Url

"SiteURL,SiteTitle,LibraryTitle,LibraryID,ItemCount,LastModified,LibrarySize(GB)" |
    Out-File -FilePath $outputFile -Encoding UTF8

# 3. Coletar bibliotecas de cada site
foreach ($siteUrl in $allSites) {
    try {
        $conn = Connect-PnPOnline -Url $siteUrl -Interactive -ReturnConnection -ErrorAction Stop
        $web = Get-PnPWeb -Connection $conn -Includes Title

        $libraries = Get-PnPList -Connection $conn | Where-Object {
            $_.BaseTemplate -eq 101 -and -not $_.Hidden -and $_.Title -ne "Formulários"
        }

        foreach ($lib in $libraries) {
            $fullLib = Get-PnPList -Identity $lib.Id -Connection $conn -Includes ItemCount, LastItemModifiedDate, RootFolder
            $folder = Get-PnPFolder -Url $fullLib.RootFolder.ServerRelativeUrl -Connection $conn -Includes StorageMetrics
            $sizeGB = [math]::Round($folder.StorageMetrics.TotalSize / 1GB, 2)

            "$siteUrl,$($web.Title),$($fullLib.Title),$($fullLib.Id),$($fullLib.ItemCount),$($fullLib.LastItemModifiedDate),$sizeGB" |
                Out-File -FilePath $outputFile -Encoding UTF8 -Append
        }
    }
    catch {
        "$siteUrl,ERRO_ACESSO,,,,," | Out-File -FilePath $outputFile -Encoding UTF8 -Append
    }
    finally {
        if ($conn) { Disconnect-PnPOnline -Connection $conn }
    }
}
```

---

## 3. Validação e Pontos de Atenção

1. Confira o arquivo CSV gerado em `C:\Temp\SharePointReports\`.
2. Para uma visão resumida apenas por site (sem iterar por biblioteca), utilize `Get-SPOSite -Limit All | Select-Object Url, Title, StorageUsageCurrent, StorageQuota`.
