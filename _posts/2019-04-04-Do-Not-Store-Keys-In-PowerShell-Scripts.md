Storing the clear text of an account or resource access key (for anything) in scripts makes these critical pieces of information vulnerable to any process within the logged in user context. In enterprise scenarios, the same critical information becomes exposed through administrative accounts and processes. Instead, use PowerShell to encrypt the key into a secured key file. The encryption will remain unique to the computer used to perform the encryption, further guarding the key from accidental exposure.

The concept of encrypting an example account key was adapted from this post,[https://stackoverflow.com/questions/23538200/encrypt-azure-storage-account-name-and-key-in-powershell-script/23541332#23541332](https://stackoverflow.com/questions/23538200/encrypt-azure-storage-account-name-and-key-in-powershell-script/23541332#23541332)

*Note:* If the script is executing inside of the Azure environment, use [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/overview) to store the information. Azure Key Vault provides an encrypted, Roles Based Access Control enabled and serverless service to store critical data.

## Create the encrypted key file
```
$KeyFile = 'Path\to\secured.key'
$EncryptedStorageKey = Read-Host -Prompt "Enter Storage Key" -AsSecureString
ConvertFrom-SecureString $EncryptedStorageKey | Set-Content $KeyFile
```

## Use the encrypted key file to authenticate to the Azure Storage Account
```
$KeyFile = 'Path\to\secured.key'
$StorageAccountName = 'MyStorageAccount1'
$StorageContainerName = "MyContainer1"
$EncryptedStorageKey = Get-Content $KeyFile | ConvertTo-SecureString
$DecryptedKey = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($EncryptedStorageKey))
$StorageContext = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $DecryptedKey
```

## Copy everything under $SourcePath to $StorageContext $StorageContainer
```
$SourcePath = "Path\to\data\to\copy\to\blob"
Get-ChildItem -File -Recurse $SourcePath | Set-AzStorageBlobContent -Context $StorageContext -Container $StorageContainerName
```

## Or, copy everything under $SourcePath to $StorageContext $StorageContainer if modified in last 5 days
```
Get-ChildItem -File -Recurse $SourcePath | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-5)} | Set-AzStorageBlobContent -Context $StorageContext -Container $StorageContainerName
```
