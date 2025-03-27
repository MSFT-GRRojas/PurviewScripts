# PurviewScripts
Scripts needed to set things up in Microsoft Purview


<##########################################################################################################
Disclaimer
The sample scripts are not supported under any Microsoft standard support program or service.
The sample scripts are provided AS IS without warranty of any kind. Microsoft further disclaims all
implied warranties including, without limitation, any implied warranties of merchantability or of fitness
for a particular purpose. The entire risk arising out of the use or performance of the sample scripts and
documentation remains with you. In no event shall Microsoft, its authors, or anyone else involved in the
creation, production, or delivery of the scripts be liable for any damages whatsoever (including, without
limitation, damages for loss of business profits, business interruption, loss of business information,
or other pecuniary loss) arising out of the use of or inability to use the sample scripts or documentation,
even if Microsoft has been advised of the  possibility of such damages.
##########################################################################################################>


$ModName = 'ExchangeOnlineManagement'
$version = '3.6.0'

# Check if the module is installed
$module = Get-InstalledModule -Name $ModName -requiredversion $version

if ($module) {
    Write-Host -ForegroundColor Green "Module $ModName version $ReqVer is already installed."
    Import-Module -Name $ModName
} else {
    Write-Host -ForegroundColor Yellow "Module $ModName version $ReqVer is not installed. Installing now..."
    Install-Module -Name $ModName -RequiredVersion $ReqVer -Force
    Write-Host -ForegroundColor Green "Module $ModName version $ReqVer has been installed."
}

#Connect to Azure AD
Connect-AzureAD

#Get AzureADDirectory Setting to validate if False
$null -eq (Get-AzureADDirectorySetting)

#View the values including EnableMIPLabels
Get-AzureADDirectorySetting | foreach values

#Let's get Group settings
$settings = Get-AzureADDirectorySetting | where-object {$_.displayname -eq "Group.Unified"}

#Let's make a variable and set it to True
$setTrue["EnableMIPLabels"] = "True"

#Finally let's set the Azure AD Directory Setting [EnableMIPLabels] to True
Set-AzureADDirectorySetting -Id $setTrue.Id -DirectorySetting $setTrue

#Connect to Exchange Online
Connect-ExchangeOnline -UserPrincipalName admin@rojasgcc.onmicrosoft.com

#Connect to Purview  
Connect-IPPSSession -UserPrincipalName admin@rojasgcc.onmicrosoft.com

#Execute the label synchronization across services (SPO/Teams/M365 Groups)
#Allow some time for the labels to show (usually a few hours)
Execute-AzureAdLabelSync

#Disconnect current session
Disconnect-ExchangeOnline -Confirm:$false
