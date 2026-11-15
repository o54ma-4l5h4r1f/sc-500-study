+++
slug = "keys-secrets-and-certificates"
title = "Keys, secrets and certificates: cryptography for security engineers"
summary = "Crypto from scratch: symmetric vs asymmetric keys, what a key vs a secret vs a certificate is, software vs HSM protection, envelope encryption (DEK wrapped by KEK), platform-managed vs customer-managed keys, key rotation and the certificate lifecycle — the vocabulary every Key Vault and encryption objective assumes."
order = 10
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/key-vault/general/basic-concepts",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/about-keys-secrets-certificates",
  "https://learn.microsoft.com/en-us/azure/key-vault/keys/about-keys",
  "https://learn.microsoft.com/en-us/azure/key-vault/secrets/about-secrets",
  "https://learn.microsoft.com/en-us/azure/key-vault/certificates/about-certificates",
  "https://learn.microsoft.com/en-us/azure/key-vault/managed-hsm/overview",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/overview",
  "https://learn.microsoft.com/en-us/azure/security/fundamentals/encryption-atrest",
  "https://learn.microsoft.com/en-us/azure/security/fundamentals/encryption-models",
  "https://learn.microsoft.com/en-us/azure/key-vault/keys/how-to-configure-key-rotation",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/soft-delete-overview",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/autorotation",
]
captured = 2026-11-15
related_objectives = ["D1.2.1", "D1.2.2", "D1.2.3", "D1.2.5", "D1.2.6", "D2.1.1", "D2.2.1", "D3.2.1", "D3.3.7"]
+++

## In one sentence

Encryption turns readable data into gibberish that only the holder of the right **key** can reverse; a **secret** is any sensitive value you store (a password, a connection string); a **certificate** is a public key plus a signed statement of who owns it — and Azure Key Vault holds all three.

## Why it exists

Every cloud breach report has the same line: "credentials found in source code" or "storage key leaked". Encryption is only as strong as the place you keep its keys. Put the key next to the data and you have locked the door and taped the key to it.

Think of a bank. Customers' valuables sit in **safe-deposit boxes** (your data, encrypted). Each box has its own key (a **data encryption key**). The bank keeps all those box keys inside a **vault** that only a few staff can open, with a sign-out sheet (Key Vault with access control and logging). The vault's own combination never leaves the vault (a **key encryption key**, ideally in an **HSM**).

As an analyst you may have seen "Unusual access to Key Vault" alerts. This page is the vocabulary behind them.

## Mental model

```svg
<svg viewBox="0 0 880 330" role="img" aria-label="Envelope encryption: data is encrypted with a data encryption key (DEK) held by the service; the DEK is wrapped by a key encryption key (KEK) that stays in Key Vault or Managed HSM; platform-managed keys mean Microsoft holds the KEK, customer-managed keys mean you hold it; disabling the KEK makes the data unreadable">
  <defs>
    <marker id="fkeyarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="40" width="200" height="90" rx="8"/>
  <text x="120" y="72" text-anchor="middle">Your data</text>
  <text x="120" y="96" text-anchor="middle" class="muted">disk, blob, database</text>
  <rect class="box-accent" x="300" y="40" width="240" height="90" rx="8"/>
  <text x="420" y="68" text-anchor="middle" class="label-accent">DEK (data key)</text>
  <text x="420" y="92" text-anchor="middle">symmetric AES-256</text>
  <text x="420" y="114" text-anchor="middle" class="muted">kept by the service, wrapped</text>
  <rect class="box-ok" x="620" y="40" width="240" height="90" rx="8"/>
  <text x="740" y="68" text-anchor="middle" class="label-accent">KEK (key-encryption key)</text>
  <text x="740" y="92" text-anchor="middle">RSA key in Key Vault / HSM</text>
  <text x="740" y="114" text-anchor="middle" class="muted">never leaves the vault</text>
  <line class="line-accent" x1="298" y1="85" x2="222" y2="85" marker-end="url(#fkeyarrow)"/>
  <text x="260" y="70" text-anchor="middle" class="muted">encrypts</text>
  <line class="line-accent" x1="618" y1="85" x2="542" y2="85" marker-end="url(#fkeyarrow)"/>
  <text x="580" y="70" text-anchor="middle" class="muted">wraps</text>
  <line class="divider" x1="20" y1="165" x2="860" y2="165"/>
  <rect class="box" x="20" y="190" width="400" height="110" rx="8"/>
  <text x="220" y="218" text-anchor="middle" class="label-accent">Platform-managed key (PMK)</text>
  <text x="220" y="244" text-anchor="middle">Microsoft holds and rotates the KEK</text>
  <text x="220" y="268" text-anchor="middle" class="muted">default, free, zero configuration</text>
  <rect class="box-accent" x="460" y="190" width="400" height="110" rx="8"/>
  <text x="660" y="218" text-anchor="middle" class="label-accent">Customer-managed key (CMK)</text>
  <text x="660" y="244" text-anchor="middle">you hold the KEK in your vault</text>
  <text x="660" y="268" text-anchor="middle" class="muted">disable it = data unreadable</text>
</svg>
```

## What actually matters

**1. Symmetric vs asymmetric.**

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | **One** shared key encrypts and decrypts | A **key pair**: public key (share freely) + private key (never share) |
| Speed | Fast — used for bulk data | Slow — used for small things |
| Typical algorithm | **AES-256** | **RSA** (2048/3072/4096-bit), **EC** (elliptic curve, P-256/P-384/P-521) |
| Used for | Encrypting disks, blobs, databases | **Wrapping** other keys, **signing**, TLS handshakes, certificates |

Two asymmetric tricks matter: encrypt with someone's **public** key and only their private key can decrypt; **sign** with your **private** key and anyone can verify with your public key. That is how a token or a certificate proves it wasn't forged.

**2. Key vs secret vs certificate in Key Vault.**

| Object | What it is | Can you read it back? | Example |
|---|---|---|---|
| **Key** | A cryptographic key used **inside** Key Vault for encrypt/decrypt, wrap/unwrap, sign/verify | ==!No — the private part never leaves== (HSM keys are non-exportable) | CMK for Storage, SQL TDE protector, disk encryption |
| **Secret** | Any value up to **25 KB**, stored encrypted | Yes — that is its purpose | Password, connection string, API key |
| **Certificate** | An X.509 certificate with lifecycle management. Creating one also creates an addressable **key** and **secret** of the same name | Public part yes; private key only if the policy marks it **exportable** (PFX or PEM) | TLS certificate for an App Service |

Each object has a URL: `https://{vault-name}.vault.azure.net/{keys or secrets or certificates}/{name}/{version}`. Every change creates a new **version**. A URL without a version always points to the latest — services use this "versionless" URI to pick up rotated keys automatically.

**3. Software vs HSM protection.** An **HSM** (hardware security module) is a tamper-resistant device that generates and uses keys without ever exposing them. Protection levels:

| Container | Protection | Compliance |
|---|---|---|
| Key Vault **Standard** | Software-protected keys | FIPS 140-2 Level 1 |
| Key Vault **Premium** | Adds **HSM-protected** keys (multitenant HSMs) | FIPS 140-3 Level 3 for new keys (older key versions on HSM Platform 1: FIPS 140-2 Level 2) |
| **Managed HSM** | HSM keys only, **single-tenant**, customer-controlled **security domain** | FIPS 140-3 Level 3 |

==@Bring your own key (BYOK)== = generate a key in your on-premises HSM and import it into Premium or Managed HSM without it ever leaving an HSM boundary.

**4. Envelope encryption.** Azure services don't call Key Vault for every block of data. They encrypt data with a fast symmetric **DEK** (data encryption key, AES-256) and store that DEK **wrapped** (encrypted) by a **KEK** (key encryption key) that lives in Key Vault. To read data, the service asks Key Vault to unwrap the DEK. ==+Disable or delete the KEK and every DEK it wrapped becomes useless== — that is "cryptographic erase", and why customers want to hold the KEK themselves.

**5. Platform-managed vs customer-managed keys.** Azure encrypts data at rest **by default** with **platform-managed keys (PMK)**: Microsoft generates, stores and rotates them; you configure nothing. **Customer-managed keys (CMK)** put the KEK in **your** Key Vault or Managed HSM. You control access, rotation and revocation — and you own the risk. ==!If you lose or purge a CMK, Microsoft can't recover your data.== That is why most services that use CMK (Azure Storage, for example) require **soft delete** and **purge protection** on the vault. Soft delete keeps deleted objects for **7–90 days (default 90)**; purge protection blocks permanent deletion until that period ends and can't be turned off. A third model, **client-side encryption**, encrypts data before it ever reaches Azure.

**6. Key rotation.** Rotation creates a **new version** of a key with new key material. Key Vault can rotate automatically with a **rotation policy**: rotate at a time after creation (default) or before expiry; minimum interval **7 days**; optional Event Grid near-expiry notification. Microsoft recommends rotating encryption keys **at least every two years**. Rotation **re-wraps DEKs**; it does not re-encrypt the data, so old versions must stay enabled until re-wrapping finishes. Managing rotation needs the **Key Vault Crypto Officer** role.

**7. Certificate lifecycle and CAs.** A **certificate authority (CA)** is a trusted issuer that signs certificates. Key Vault can request certificates from **partner CAs** (DigiCert, GlobalSign) and **auto-renew** them; it can create **self-signed** certificates; and it can import certificates from any CA (non-partner CAs don't auto-renew). The lifecycle: create (or import) → use → near-expiry notification → renew → retire. Expired certificates cause outages, so set auto-renew or alerts.

## Where it lives

- **Azure portal > Key vaults > {vault} > Objects > Keys | Secrets | Certificates** — create, import, view versions.
- **Keys > {key} > Rotation policy** — autorotation; **Rotate now** for on-demand.
- **Azure portal > {service} > Settings > Encryption** (Storage, SQL, disks…) — switch from Microsoft-managed keys to customer-managed keys and pick the vault key.
- CLI: `az keyvault key create --vault-name kv-lab -n cmk1 --kty RSA --size 3072`, `az keyvault secret set --vault-name kv-lab -n dbpass --value '...'`, `az keyvault key rotate --vault-name kv-lab -n cmk1`.

![Diagram that shows Azure encryption at rest components and key hierarchy](https://learn.microsoft.com/en-us/azure/security/fundamentals/media/encryption-atrest/azure-security-encryption-atrest-fig1.png){Encryption at rest: the DEK encrypts data, the KEK in Key Vault wraps the DEK, Entra ID controls access to the KEK}

![Overview of how Azure Key Vault works](https://learn.microsoft.com/en-us/azure/key-vault/media/key-vault-whatis/azurekeyvault_overview.png){Key Vault in one picture: apps and services fetch keys, secrets and certificates from one controlled place}

## Exam traps

- ==!A key is not a secret.== You can't "read" a Key Vault key's private part; you ask Key Vault to use it. Passwords and connection strings are **secrets**.
- ==!Software keys don't need Premium; HSM keys do.== "Keys must be protected by an HSM" → Premium or Managed HSM. "Single-tenant HSM / customer controls the security domain" → **Managed HSM**.
- ==+CMK setups generally require soft delete and purge protection== on the vault.
- Rotating a key doesn't re-encrypt the data; disabling a KEK makes the data inaccessible.
- PMK is already on by default. "Encrypt data at rest" alone doesn't require any action — "control the encryption key" does (CMK).

## Skip

- **Skip** the key-type and algorithm detail tables (RSA-OAEP variants, AES-GCM IV handling, key operation names). Know RSA, EC, AES and wrap/unwrap.
- **Skip** quantum-safe algorithm notes, Azure Payment HSM, Cloud HSM and Integrated HSM.
- **Skip** certificate policy JSON fields (`x509_props`, `lifetime_actions` syntax) and the certificate composition internals; know addressable key + secret + exportable flag.
- **Skip** the per-service encryption support matrix in the encryption models article — each service's CMK option is taught on its own objective page.
- **Skip** Managed HSM security-domain ceremony steps (quorum key files) beyond "customer-controlled security domain".

## Screenshots

![Diagram that shows the role of certificates in a key vault](https://learn.microsoft.com/en-us/azure/key-vault/media/azure-key-vault.png){A Key Vault certificate links a key, a secret and certificate metadata}

![Diagram showing an Azure service performing server-side encryption with managed encryption keys](https://learn.microsoft.com/en-us/azure/security/fundamentals/media/encryption-models/azure-security-encryption-atrest-fig3.png){Server-side encryption with customer-managed keys in your Key Vault}

![Diagram showing an application encrypting data before sending encrypted data to an Azure service](https://learn.microsoft.com/en-us/azure/security/fundamentals/media/encryption-models/azure-security-encryption-atrest-fig2.png){Client-side encryption: data is encrypted before it reaches Azure}

![Diagram showing Microsoft-managed key storage for server-side encryption by using platform-managed keys](https://learn.microsoft.com/en-us/azure/security/fundamentals/media/encryption-models/azure-security-encryption-atrest-fig4.png){Server-side encryption with platform-managed keys — the default}

![Rotation policy configuration](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-1.png){Key rotation policy: rotation time, expiry, notification}

![Configure rotation during key creation](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-2.png){Setting rotation when creating a key}

![Configure rotation on existing key](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-3.png){Adding rotation to an existing key}

![Rotation on-demand](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-4.png){Rotate now — on-demand rotation}

![Configure Notification](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-5.png){Near-expiry notification through Event Grid}

![Screenshot of key rotation policy compliance](https://learn.microsoft.com/en-us/azure/key-vault/media/keys/key-rotation/key-rotation-policy.png){Azure Policy reporting which keys lack a rotation policy}
