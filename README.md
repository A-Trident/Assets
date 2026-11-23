# Assets
Assets for personal project development

# SelfSignedCertificate

New-SelfSignedCertificate `
    -Type CodeSigningCert `
    -Subject "CN=AuditX Pro, O=AuditX Pro" `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5) `
    -FriendlyName "AuditX Pro"

$thumbprint = "19E2B61E71F930E741D3BB3BB05156AA7839BB67"

# Export the cert
$cert = Get-ChildItem "Cert:\LocalMachine\My\$thumbprint"
Export-Certificate -Cert $cert -FilePath "D:\Certificate\AuditX.cer"

# Import into Trusted Root Certification Authorities
Import-Certificate -FilePath "D:\Certificate\AuditX.cer" `
    -CertStoreLocation "Cert:\LocalMachine\Root"

# Import into Trusted Publishers
Import-Certificate -FilePath "D:\Certificate\AuditX.cer" `
    -CertStoreLocation "Cert:\LocalMachine\TrustedPublisher"

$cert = Get-ChildItem "Cert:\LocalMachine\My\$thumbprint"

Set-AuthenticodeSignature `
    -FilePath "D:\2. Workspace\AuditX\RS Group\Release\3.0.0.1\AuditX.exe" `
    -Certificate $cert `
    -TimestampServer "http://timestamp.digicert.com"

