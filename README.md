# Assets

![Repo Status](https://img.shields.io/badge/Assets-Active-2ea44f?style=flat-square)
![CDN](https://img.shields.io/badge/CDN-jsDelivr-ff5627?style=flat-square)
![Projects](https://img.shields.io/badge/Projects-AuditX%20Pro%20%7C%20AVSA%20Tech-0969da?style=flat-square)

Centralized asset repository for personal and enterprise project development (`AuditX Pro`, `AVSA Tech`, and web portals).

---

## 📦 Repository Asset Index

> **Auto-Sync Enabled:** This index updates automatically via GitHub Actions whenever files are added, renamed, or deleted. Use **jsDelivr CDN** links in production web apps and WPF clients, and **Raw URLs** for direct testing.

<!-- ASSETS-START -->
*Push your assets to the repository and trigger the GitHub Action to auto-populate all Image, Icon, Video, and Background links here.*
<!-- ASSETS-END -->

---

## 🔐 Self-Signed Code Signing Certificate (`AuditX Pro`)

Run these scripts in an **Administrator PowerShell** session to generate a local code-signing certificate, establish machine trust, and sign the `AuditX.exe` binary.

### Step 1: Generate & Trust the Certificate

Run once on your build machine to create a 5-year RSA 2048-bit code-signing certificate and register it in the local trust stores:

```powershell
# 1. Create output directory if it does not exist
$certDir = "D:\Certificate"
if (-not (Test-Path $certDir)) { New-Item -ItemType Directory -Path $certDir | Out-Null }

# 2. Generate the Code Signing Certificate and capture the object dynamically
$cert = New-SelfSignedCertificate `
    -Type CodeSigningCert `
    -Subject "CN=AuditX Pro, O=AVSA Tech Consulting" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5) `
    -FriendlyName "AuditX Pro Code Signing"

Write-Host "Generated Certificate Thumbprint: $($cert.Thumbprint)" -ForegroundColor Cyan

# 3. Export the public certificate (.cer)
$cerPath = "$certDir\AuditX.cer"
Export-Certificate -Cert $cert -FilePath$cerPath -Force | Out-Null

# 4. Import into Trusted Root Certification Authorities (Prevents "Untrusted Root" errors)
Import-Certificate -FilePath $cerPath -CertStoreLocation "Cert:\LocalMachine\Root" | Out-Null

# 5. Import into Trusted Publishers (Allows silent execution without SmartScreen publisher warnings)
Import-Certificate -FilePath $cerPath -CertStoreLocation "Cert:\LocalMachine\TrustedPublisher" | Out-Null

Write-Host "Certificate successfully installed in Root and TrustedPublisher stores." -ForegroundColor Green
```

### Step 2: Sign the `AuditX.exe` Executable

Run during your release build process to apply a SHA-256 Authenticode signature with an RFC 3161 timestamp (keeping the signature valid even after the certificate expires):

```powershell
$exePath = "D:\2. Workspace\AuditX\RS Group\Release\3.0.0.1\AuditX.exe"

# Retrieve the latest AuditX Pro certificate automatically by FriendlyName
$signingCert = Get-ChildItem "Cert:\LocalMachine\My" |
    Where-Object { $_.FriendlyName -eq "AuditX Pro Code Signing" } |
    Sort-Object NotAfter -Descending |
    Select-Object -First 1

if ($null -eq$signingCert) {
    throw "AuditX Pro signing certificate not found in Cert:\LocalMachine\My."
}

# Apply Authenticode signature with SHA-256 hash and RFC 3161 timestamp
Set-AuthenticodeSignature `
    -FilePath $exePath `
    -Certificate $signingCert `
    -HashAlgorithm SHA256 `
    -TimestampServer "http://timestamp.digicert.com"

# Verify the applied signature
Get-AuthenticodeSignature -FilePath $exePath | Format-List Status, StatusMessage, SignerCertificate
```

<details>
<summary><b>Step 3: Export Private Key Backup (.pfx) (Optional)</b></summary>
<br>

Use this snippet to back up the certificate and private key before migrating to a new development machine or CI/CD runner:

```powershell
$pfxPassword = ConvertTo-SecureString -String "ChangeThisStrongPassword!" -Force -AsPlainText
Export-PfxCertificate `
    -Cert $signingCert `
    -FilePath "D:\Certificate\AuditX_PrivateBackup.pfx" `
    -Password $pfxPassword
```

</details>
