# Chart Packaging and Signing Guide

This guide explains the complete process for packaging and signing Helm charts, focusing on the GPG signing process, theoretical foundations, and practical implementation details.

## Introduction

Packaging and signing Helm charts are crucial steps for distribution that ensure:
- Charts are properly packaged in a standardized format
- The integrity and authenticity of charts can be verified
- Users can trust the source of the charts they install

## Prerequisites

Before starting, ensure you have:
- Helm CLI installed (v3.x recommended)
- GnuPG (GPG) for chart signing
- A properly configured GPG key pair
- Access to a chart repository (optional for distribution)

## Chart Packaging Process

### 1. Prepare Your Chart

Ensure your chart follows the standard Helm chart structure:
```
mychart/
├── Chart.yaml          # Contains chart metadata
├── values.yaml         # Default configuration values
├── templates/          # Directory containing templates
├── charts/             # Directory for dependent charts (optional)
└── README.md           # Documentation
```

### 2. Validate Chart

```bash
helm lint mychart/
```

This checks for structural issues in your chart before packaging.

### 3. Package the Chart

```bash
helm package mychart/
```

**What this does:**
- Bundles all chart files into a versioned `.tgz` archive
- Creates a file like `mychart-0.1.0.tgz` based on name and version in Chart.yaml
- Computes a SHA256 checksum of the package content

**Why packaging is important:**
- Creates a portable, compressed archive
- Ensures consistent deployment across environments
- Facilitates distribution through repositories
- Enables version control for chart releases

## Deep Dive into GPG and Chart Signing

### Understanding GPG (GNU Privacy Guard)

GPG is an implementation of the OpenPGP standard that provides:

1. **Public Key Cryptography**: Uses asymmetric encryption with key pairs
   - **Private Key**: Kept secret, used for signing and decryption
   - **Public Key**: Distributed widely, used for verification and encryption

2. **Digital Signatures**: Mathematical schemes that verify:
   - **Authenticity**: Confirms the identity of the signer
   - **Integrity**: Detects if data has been altered
   - **Non-repudiation**: Prevents the signer from denying they signed it

3. **Web of Trust**: A decentralized trust model where users vouch for others' key authenticity by signing them

4. **Key Components**:
   - **Key Fingerprint**: A unique identifier for a key (40 hexadecimal characters)
   - **User ID**: Contains name, email address, and optional comment
   - **Subkeys**: Additional keys managed under a master key, often used for signing or encryption

### GPG Key Creation in Detail

```bash
gpg --full-generate-key
```

**Parameters explained:**
- `--full-generate-key`: Interactive mode for complete key generation options

During this process, you'll need to specify:

1. **Key Type**: Usually "RSA and RSA" (option 1)
   - First RSA is for signing
   - Second RSA is for encryption

2. **Key Size**: 4096 bits recommended
   - Larger keys provide stronger security
   - 2048 bits is the minimum recommended size
   - 4096 bits offers security margin for future

3. **Key Validity**: How long the key remains valid
   - "0" means never expires (not recommended for production)
   - Recommended to set 1-2 years for regular rotation

4. **User ID**: 
   - Real Name: Identifiable name (individual or organization)
   - Email: Valid email address for contact
   - Comment: Optional context (e.g., "Helm Chart Signing Key")

5. **Passphrase**: 
   - Protects private key with strong password
   - Should be complex but memorable

### Detailed GPG Key Management

#### Viewing Your Keys

```bash
# List public keys
gpg --list-keys

# List private keys
gpg --list-secret-keys
```

**Output explained:**
```
pub   rsa4096 2023-04-03 [SC] [expires: 2025-04-02]
      A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0
uid           [ultimate] John Doe (Helm Chart Signing) <john@example.com>
sub   rsa4096 2023-04-03 [E] [expires: 2025-04-02]
```

- `pub`: Public key details
- `rsa4096`: Algorithm and bit strength
- `[SC]`: Key capabilities (S=Signing, C=Certifying)
- `A1B2...`: Key fingerprint
- `uid`: User Identity
- `[ultimate]`: Trust level
- `sub`: Subkey (typically for encryption)
- `[E]`: Encryption capability

#### Exporting Keys

```bash
# Export public key for distribution
gpg --export --armor "John Doe" > john-doe-public.asc

# Export private key for backup (KEEP SECURE!)
gpg --export-secret-key --armor "John Doe" > john-doe-private.asc
```

**Parameters explained:**
- `--export`: Extracts the public key
- `--export-secret-key`: Extracts the private key (sensitive!)
- `--armor`: Outputs in ASCII-armored format (text-based, not binary)
- `"John Doe"`: Key identifier (can use name, email, or key ID)

#### Preparing GPG for Helm

Helm looks for keys in specific locations. For modern GPG (2.1+):

```bash
# Export secret keys to the location Helm expects
gpg --export-secret-keys > ~/.gnupg/secring.gpg
```

**Why needed?**: Modern GPG versions store keys differently, but Helm still uses the legacy format.

### Chart Signing Process in Detail

```bash
helm package --sign --key "John Doe" --keyring ~/.gnupg/secring.gpg mychart/
```

**Parameters explained:**
- `--sign`: Activates the signing process
- `--key`: Specifies which key to use (name, email, or key ID)
- `--keyring`: Path to the keyring file containing secret keys
- `mychart/`: Path to the chart directory

**What happens during signing:**

1. Helm packages the chart as normal
2. Calculates a SHA256 digest of the package
3. Creates a JSON file with metadata and the digest
4. Uses GPG to create a detached signature of this JSON
5. Combines the metadata and signature into a provenance file (.prov)

#### Anatomy of a .prov File

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

name: mychart
version: 0.1.0
description: Example Helm chart
keywords:
  - example
home: https://example.com
sources:
  - https://github.com/example/mychart
maintainers:
  - name: John Doe
    email: john@example.com
...
files:
  mychart-0.1.0.tgz: sha256:a1b2c3d4e5f6...
-----BEGIN PGP SIGNATURE-----

iQEzBAEBCgAdFiEE1234567890abcdef1234567890abcdef=
...signature data...
=abCD
-----END PGP SIGNATURE-----
```

**Key components:**
- Chart metadata (from Chart.yaml)
- File hash (SHA256 of the .tgz file)
- GPG signature block

### The Cryptographic Theory Behind Chart Signing

1. **Hash Function (SHA256)**:
   - One-way function that creates a fixed-length "fingerprint" of data
   - Any change to the original data produces a completely different hash
   - Computationally infeasible to find two inputs with same hash
   - SHA256 produces a 256-bit (64 character hex) output regardless of input size

2. **Signing Process**:
   - The hash of the chart package is computed
   - This hash is encrypted using the signer's private key
   - The encrypted hash becomes the signature

3. **Verification Process**:
   - Verifier computes hash of the received chart
   - Decrypts signature using signer's public key to get original hash
   - Compares computed hash with decrypted hash
   - If identical, chart is unmodified and signed by key owner

### Verification Process

```bash
# Import the publisher's public key
gpg --import publisher-public-key.asc

# Verify chart signature
helm verify mychart-0.1.0.tgz
```

**What happens during verification:**
1. Helm extracts signature and expected hash from .prov file
2. Uses GPG to verify signature authenticity
3. Computes actual hash of .tgz file
4. Compares expected and actual hashes
5. Returns success only if both verification steps pass

## Security Considerations

### Key Management
- **Securely store your private key** with proper backup procedures
- **Protect passphrase** using a password manager
- Consider using **hardware security modules** for key storage
- Implement **key rotation policies**

### Repository Security
- Use **HTTPS** for chart repository access
- Implement proper **access controls** to chart repositories
- Consider using **notary** or similar tools for additional security

## Troubleshooting

### Common Signing Issues

1. **Missing Secret Keyring**
   ```
   Error: open ~/.gnupg/secring.gpg: no such file or directory
   ```
   
   Solution for GPG 2.1+:
   ```bash
   gpg --export-secret-keys > ~/.gnupg/secring.gpg
   ```

2. **Key Not Found**
   ```
   Error: secret key not available
   ```
   
   Verify your key is available:
   ```bash
   gpg --list-secret-keys
   ```

3. **Verification Failures**
   
   Common causes:
   - Chart was modified after signing
   - Wrong public key used for verification
   - Corrupted provenance file
   
   Check with:
   ```bash
   helm verify --debug mychart-0.1.0.tgz
   ```

4. **Passphrase Issues**
   
   If GPG isn't asking for passphrase or remembering it:
   ```bash
   # Configure GPG agent to cache passphrase
   echo "default-cache-ttl 3600" >> ~/.gnupg/gpg-agent.conf
   echo "max-cache-ttl 86400" >> ~/.gnupg/gpg-agent.conf
   gpg-connect-agent reloadagent /bye
   ```

## Tricky Questions with Code Snippets

### Question 1: Key Trust Levels

**Q: What does this output mean, and why might it cause verification issues?**

```
$ gpg --list-keys
pub   rsa4096 2023-04-03 [SC] [expires: 2025-04-02]
      A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0
uid           [unknown] John Doe (Helm Chart Signing) <john@example.com>
sub   rsa4096 2023-04-03 [E] [expires: 2025-04-02]
```

**Answer:** The `[unknown]` trust level indicates GPG doesn't know if this key is trustworthy. This could cause issues with automated verification systems that require minimum trust levels. To fix this:

```bash
# Set trust level for a key
gpg --edit-key john@example.com
> trust
> 5  # Ultimate trust
> save
```

This sets the key to ultimate trust level, which means you're certifying the key belongs to the specified user.

### Question 2: Key Expiration Management

**Q: Your chart signature is failing with "key expired" errors. How do you extend your key's expiration without creating a new key?**

```bash
# Current status:
$ gpg --list-keys
pub   rsa4096 2023-01-01 [SC] [expired: 2024-01-01]
      A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0
uid           [ultimate] John Doe <john@example.com>
```

**Answer:** You need to edit the key and update its expiration date:

```bash
# Edit the key
gpg --edit-key A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0

# In the GPG prompt:
> key 0  # Select the main key
> expire
> 2y     # Set new expiration to 2 years
> save

# Export updated public key for distribution
gpg --export --armor A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0 > updated-key.asc
```

After extending the expiration, users need to re-import your updated public key.

### Question 3: Subkey Signing

**Q: Can this command work? If not, why not and how would you fix it?**

```bash
helm package --sign --key "john@example.com!1" --keyring ~/.gnupg/secring.gpg mychart/
```

**Answer:** This command is attempting to sign using a specific subkey (designated by `!1`). Helm's signing implementation doesn't directly support subkey selection syntax. To fix:

1. Either use the main key identifier without subkey notation:
   ```bash
   helm package --sign --key "john@example.com" --keyring ~/.gnupg/secring.gpg mychart/
   ```

2. Or set the specific subkey as default for signing in GPG configuration:
   ```bash
   # Edit key
   gpg --edit-key john@example.com
   
   # In GPG prompt
   > key 1  # Select subkey (index may vary)
   > primary
   > save
   ```

### Question 4: Air-Gapped Signing

**Q: How would you implement chart signing on an air-gapped system where the signing machine has no internet access?**

**Answer:** You would use a two-stage process:

```bash
# On internet-connected machine:
# 1. Package the chart without signing
helm package mychart/

# 2. Transfer the .tgz file to air-gapped machine via secure media

# On air-gapped machine with GPG keys:
# 3. Sign the pre-packaged chart
helm sign mychart-0.1.0.tgz --key "John Doe" --keyring ~/.gnupg/secring.gpg

# 4. Transfer both .tgz and .prov files back via secure media

# On internet-connected machine:
# 5. Verify the signature (optional)
helm verify mychart-0.1.0.tgz

# 6. Upload both files to chart repository
```

This process maintains security by keeping private keys on the isolated system while still enabling signed chart distribution.

### Question 5: Multiple Signatures

**Q: Is this script correct for applying multiple signatures to a chart? If not, why?**

```bash
#!/bin/bash
helm package mychart/
helm sign mychart-0.1.0.tgz --key "Developer" --keyring ~/.gnupg/secring.gpg
helm sign mychart-0.1.0.tgz --key "Security" --keyring ~/.gnupg/secring.gpg
```

**Answer:** No, this script is incorrect. Each `helm sign` command overwrites the previous .prov file instead of adding a second signature. OpenPGP supports multiple signatures, but Helm's implementation doesn't. 

To implement multiple signatures approval process:

1. Create separate .prov files for each signer:
   ```bash
   helm sign mychart-0.1.0.tgz --key "Developer" --keyring ~/.gnupg/secring.gpg
   mv mychart-0.1.0.tgz.prov mychart-0.1.0.tgz.dev.prov
   
   helm sign mychart-0.1.0.tgz --key "Security" --keyring ~/.gnupg/secring.gpg
   mv mychart-0.1.0.tgz.prov mychart-0.1.0.tgz.sec.prov
   ```

2. Use custom verification script that checks both signatures:
   ```bash
   #!/bin/bash
   helm verify mychart-0.1.0.tgz || exit 1
   cp mychart-0.1.0.tgz.dev.prov mychart-0.1.0.tgz.prov
   helm verify mychart-0.1.0.tgz || exit 1
   echo "All signatures verified!"
   ```

### Question 6: Key Management Automation

**Q: What security issues does this script have for automating key management in CI/CD?**

```bash
#!/bin/bash
# CI/CD Chart Signing Script
echo "$GPG_PRIVATE_KEY" | base64 -d > /tmp/private.key
gpg --import /tmp/private.key
echo "$GPG_PASSPHRASE" | gpg --passphrase-fd 0 --pinentry-mode loopback \
  --sign-key "Helm Signer"
helm package --sign --key "Helm Signer" mychart/
rm /tmp/private.key
```

**Answer:** This script has several security issues:

1. **Temporary file exposure**: Writes the private key to an unencrypted file in /tmp which could be read by other processes
2. **Passphrase in environment variable**: Environment variables can be exposed in logs or process listings
3. **No entropy for key generation**: In CI/CD environments, entropy may be limited
4. **No cleanup on failure**: If the script fails, the private key might remain on disk
5. **Unnecessary key signing**: The script signs its own key, which is redundant

Improved version:
```bash
#!/bin/bash
set -e # Exit on any error

# Create secure temporary directory with restricted permissions
TMPDIR=$(mktemp -d)
chmod 700 "$TMPDIR"
trap 'rm -rf "$TMPDIR"' EXIT

# Import key directly from variable without intermediate file
gpg --homedir "$TMPDIR" --import <(echo "$GPG_PRIVATE_KEY" | base64 -d)

# Use key loopback and batch mode for passphrase
gpg --homedir "$TMPDIR" --batch --yes --passphrase-fd 3 \
  --pinentry-mode loopback --export-secret-keys > "$TMPDIR/secring.gpg" 3<<<"$GPG_PASSPHRASE"

# Sign the package
helm package --sign --key "Helm Signer" --keyring "$TMPDIR/secring.gpg" mychart/

# Verify signature before publishing
helm verify mychart-*.tgz
```

This improved script:
- Uses a secure temp directory with proper permissions
- Never writes the unencrypted key to disk
- Properly cleans up via trap even if the script fails
- Verifies the signature before finishing
