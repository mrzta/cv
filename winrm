
# Implementing Secure PowerShell Remoting and Configuring a Windows Machine as a GitLab Runner

## Introduction

PowerShell Remoting offers a powerful tool for administrators to run commands across remote Windows systems seamlessly. This document outlines best practices for securely implementing PowerShell Remoting in a production environment, especially when configuring a Windows machine to serve as a GitLab Runner. We will also cover the use of service accounts for GitLab Runners and secure password management strategies.

## Secure PowerShell Remoting Setup

### 1. **Enable WinRM Securely**

- **HTTPS Protocol:** Always configure WinRM to use HTTPS for encrypted communication. This requires a valid SSL certificate on the remote host, ensuring that all data transmitted over the network is encrypted.
- **Firewall Configuration:** Ensure the firewall allows traffic on the WinRM service ports (5985 for HTTP, 5986 for HTTPS). Preferably, limit access to these ports to known IP addresses or subnets within your network.

### 2. **Configure TrustedHosts Carefully**

- **Selective Trust:** Only add specific hostnames or IP addresses to the TrustedHosts list on clients that need to connect to them. Avoid using wildcards (*) unless absolutely necessary, and understand the risks involved.
- **Regular Audits:** Periodically review the TrustedHosts list on each client for any entries that are no longer required or that pose a security risk.

### 3. **Use Strong Authentication**

- **Kerberos for Domain-joined Machines:** For machines within the same domain, rely on Kerberos for secure, ticket-based authentication.
- **Credential Objects for Non-domain Scenarios:** When dealing with non-domain scenarios or cross-domain authentication, use PowerShell's `Get-Credential` cmdlet to securely handle credentials. Avoid hard-coding credentials in scripts.

## Configuring a Windows Machine as a GitLab Runner

### 1. **Service Account for GitLab Runner**

- **Dedicated Account:** Use a dedicated service account for the GitLab Runner service. This account should have the minimum required permissions for the tasks it needs to perform, following the principle of least privilege.
- **No Local Administrator Rights:** Unless absolutely necessary, the GitLab Runner service account should not have local administrator rights.

### 2. **Secure Password Storage**

- **Windows Credential Manager:** Store the service account password securely using the Windows Credential Manager, accessible to the account running the GitLab Runner service.
- **PowerShell Script Encryption:** If you must store passwords within scripts, use PowerShell's Secure String feature to encrypt them. Only the account that encrypted the password can decrypt it, adding a layer of security.

### 3. **GitLab Runner Configuration**

- **Limit Concurrent Jobs:** Configure the GitLab Runner to limit the number of concurrent jobs based on the capabilities of the host machine. This prevents overloading the system and ensures efficient use of resources.
- **Regular Updates:** Keep the GitLab Runner software up to date to ensure you have the latest security patches and features.

### 4. **Monitoring and Logging**

- **Audit Trails:** Enable logging for both PowerShell Remoting and the GitLab Runner. Regularly monitor these logs for any unusual activity that could indicate a security issue.
- **Performance Monitoring:** Use Windows Performance Monitor (PerfMon) to track the health and performance of the system running as a GitLab Runner. This can help identify issues before they become critical.

## Conclusion

Implementing PowerShell Remoting and configuring a Windows machine as a GitLab Runner requires careful consideration of security and performance. By following the best practices outlined above, organizations can ensure that their remote management infrastructure and CI/CD pipelines are both secure and efficient. Regularly review and update these configurations to adapt to evolving security landscapes and organizational needs.

### Step 1: Obtain a Certificate

#### Option A: Use a Certificate Authority (CA)

The best practice in a production environment is to use a certificate issued by a trusted Certificate Authority (CA), either from an internal CA (if your organization has one) or a public CA. This ensures the certificate is automatically trusted by clients within the organization or globally, depending on the CA.

1. **Request a Certificate:**
   - You can request a certificate from your CA by generating a Certificate Signing Request (CSR) on the server that will run WinRM. The common name (CN) of the certificate should match the fully qualified domain name (FQDN) of the server.
   - Submit the CSR to your CA and obtain the certificate.

#### Option B: Self-Signed Certificate

For testing purposes or in environments where a CA is not available, you can generate a self-signed certificate.

1. **Generate a Self-Signed Certificate:**
   Use PowerShell to create a self-signed certificate with the following command:
   ```powershell
   New-SelfSignedCertificate -DnsName "server.domain.com" -CertStoreLocation Cert:\LocalMachine\My
   ```
   - Replace `"server.domain.com"` with the FQDN of your server.
   - This command creates a certificate and stores it in the local machine's personal certificate store.

### Step 2: Configure WinRM to Use the Certificate

1. **Find the Certificate's Thumbprint:**
   Use PowerShell to list certificates in the local machine store and note the thumbprint of the certificate you created or obtained.
   ```powershell
   Get-ChildItem -Path Cert:\LocalMachine\My
   ```

2. **Assign the Certificate to WinRM:**
   Use the thumbprint to assign the certificate to WinRM for HTTPS.
   ```powershell
   winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{Hostname="server.domain.com"; CertificateThumbprint="THUMBPRINT"}
   ```
   - Replace `"server.domain.com"` with your server's FQDN.
   - Replace `"THUMBPRINT"` with the actual thumbprint of your certificate.

### Step 3: Trust the Certificate (If Self-Signed or From an Internal CA)

If you're using a self-signed certificate or one from an internal CA, clients might not trust it by default. You'll need to import the certificate into the trusted root certificates store on client machines.

1. **Export the Certificate (if necessary):**
   If you need to distribute the certificate, export it from the server:
   ```powershell
   Export-Certificate -Cert (Get-ChildItem -Path Cert:\LocalMachine\My\THUMBPRINT) -FilePath "certificate.cer"
   ```
   - Replace `"THUMBPRINT"` with the thumbprint of your certificate.

2. **Import the Certificate on Client Machines:**
   On each client machine, import the certificate to the Trusted Root Certification Authorities store:
   ```powershell
   Import-Certificate -FilePath "path\to\certificate.cer" -CertStoreLocation Cert:\LocalMachine\Root
   ```

### Step 4: Verify the Configuration

After configuring, restart the WinRM service and verify that you can establish a secure connection using HTTPS. Use the `Test-WSMan` cmdlet from a client machine to test connectivity:
```powershell
Test-WSMan -ComputerName "server.domain.com" -UseSSL
```
- Replace `"server.domain.com"` with the FQDN of your server.

This process ensures that your WinRM service is securely configured to communicate over HTTPS, using certificates for encryption and identity verification.
