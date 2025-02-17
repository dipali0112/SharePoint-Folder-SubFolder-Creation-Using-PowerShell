# SharePoint Folder Creation using PowerShell 🚀

## Overview
This PowerShell script automates the creation of folders and subfolders within a SharePoint Online document library. Additionally, it ensures that PDF files are uploaded into their respective folders and dynamically updates a 'Folder Count' column.

## Features ✨
- ✅ Creates SharePoint document libraries dynamically.
- ✅ Generates folder and subfolder structures recursively.
- ✅ Uploads PDF files to their corresponding SharePoint folders.
- ✅ Adds custom columns like 'FolderCount' and 'PageCount'.
- ✅ Updates folder count dynamically within SharePoint.
- ✅ Uses PnP PowerShell for seamless SharePoint integration.

## Requirements ⚙️
To run this script, you need:
- 📌 **PnP PowerShell module** (Install using `Install-Module PnP.PowerShell -Force -Scope CurrentUser`)
- 📌 **Microsoft Online SharePoint PowerShell module** (Install using `Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser`)
- 📌 **SharePoint Online access** with necessary permissions to read and update the document library.

## Installation 📝
### Step 1: Install Required PowerShell Modules
```powershell
Install-Module PnP.PowerShell -Force -Scope CurrentUser
Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser
```

### Step 2: Connect to SharePoint Online
```powershell
Connect-PnPOnline -Url "https://yoursharepointsite.sharepoint.com/sites/yourSite" -UseWebLogin
```

### Step 3: Run the Script
Save the script as `CreateFolders.ps1` and execute it using:
```powershell
.\CreateFolders.ps1
```

## Usage 
1. Update the **SharePoint site URL**, **document library name**, and **local folder path** in the script.
2. Run the script in PowerShell **after logging into SharePoint Online**.
3. Folder structures and PDF files will be automatically created and uploaded.
4. The folder count column will be updated dynamically.

## Example Code 📝
```powershell
# Install SharePoint Online PowerShell Module
Install-Module -Name Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser

# Connect to SharePoint Online
Connect-PnPOnline -Url "https://yoursharepointsite.sharepoint.com/sites/yourSite" -UseWebLogin

# Create Document Library
$LibraryName = "testinglib"
$ctx = New-Object Microsoft.SharePoint.Client.ClientContext($SiteURL)
$web = $ctx.Web
$listCreationInfo = New-Object Microsoft.SharePoint.Client.ListCreationInformation
$listCreationInfo.Title = $LibraryName
$listCreationInfo.TemplateType = [Microsoft.SharePoint.Client.ListTemplateType]::DocumentLibrary
$List = $web.Lists.Add($listCreationInfo)
$ctx.ExecuteQuery()
Write-Host "Document Library '$LibraryName' created successfully!" -ForegroundColor Green
```

## Folder & File Processing 🔄
```powershell
# Function to create folder structure in SharePoint Document Library
function Create-SharePointFolders {
    param ([string]$folderPath)
    
    $folders = Get-ChildItem -Path $folderPath -Directory -Recurse
    foreach ($folder in $folders) {
        $relativePath = $folder.FullName.Replace($localPath, "").TrimStart("\")
        $relativePath = $relativePath -replace "\\", "/"
        
        Write-Host "Creating folder in SharePoint: $relativePath"
        $folderLevels = $relativePath -split "/"
        $currentPath = ""
        
        foreach ($level in $folderLevels) {
            $parentFolder = $currentPath
            $currentPath = if ($currentPath -eq "") { $level } else { "$currentPath/$level" }
            
            $existingFolder = Get-PnPFolder -Url "$libraryName/$currentPath" -ErrorAction SilentlyContinue
            if (-not $existingFolder) {
                Write-Host "Creating folder: $currentPath"
                if ($parentFolder -eq "") {
                    Add-PnPFolder -Name $level -Folder $libraryName
                } else {
                    Add-PnPFolder -Name $level -Folder "$libraryName/$parentFolder"
                }
            }
        }
        
        # Upload PDF files to SharePoint
        $pdfFiles = Get-ChildItem -Path $folder.FullName -Filter *.pdf
        foreach ($pdfFile in $pdfFiles) {
            Add-PnPFile -Path $pdfFile.FullName -Folder "$libraryName/$relativePath"
        }
    }
}
```

## Updating Folder Count 📊
```powershell
# Update Folder Count in SharePoint
$libraryName = "MarketingDocuments"
$mainFolders = Get-PnPListItem -List $libraryName -Fields "FileRef", "FileLeafRef", "FSObjType" | Where-Object { $_["FSObjType"] -eq 1 }
foreach ($folder in $mainFolders) {
    $folderName = $folder["FileLeafRef"]
    $folderPath = $folder["FileRef"]
    $subfolders = Get-PnPListItem -List $libraryName -Fields "FileRef", "FSObjType" | Where-Object { $_["FileRef"] -like "$folderPath/*" -and $_["FSObjType"] -eq 1 }
    $folderCount = ($subfolders | Measure-Object).Count
    Set-PnPListItem -List $libraryName -Identity $folder.Id -Values @{"FolderCount" = $folderCount}
    Write-Host "✅ Folder '$folderName' has $folderCount subfolders."
}
```

## Troubleshooting 🔧
- **Module Not Found**: Ensure `PnP.PowerShell` and `Microsoft.Online.SharePoint.PowerShell` modules are installed.
- **Authentication Issues**: Use `-UseWebLogin` when connecting to SharePoint.
- **Folder Creation Issues**: Ensure the correct library name and paths are specified.
- **Folder Count Errors**: Check for proper folder structure in SharePoint.


## GitHub Tags 🔍
- PowerShell
- SharePoint Automation
- Folder Structure in SharePoint
- PnP PowerShell
- Document Library Management
