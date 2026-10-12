# CA Assignment – Brief Answers

## 1. What are the contents of the CSR sent by server to CA?
File: `Server/pki/reqs/nuces.req` (copied to `CA/pki/reqs/nuces.req`).

It contains:
- **Subject / identity:** `CN = FAST University`
- **Server's public key:** RSA 2048-bit, exponent 65537
- **Signature algorithm + signature:** signed with the server's private key (`Server/pki/private/nuces.key`) to prove the server owns the key
- **Optional attributes/extensions:** none requested

The CSR does **not** contain the private key.

## 2. How do servers send CSRs to a CA in real production?
- Over the network, not by file copy:
  - **Public CA** (Let's Encrypt, DigiCert): upload/paste CSR on the CA's website or API.
  - **Automated:** ACME protocol (`certbot`, `acme.sh`) – the client creates the CSR and submits it over HTTPS automatically.
  - **Enterprise/internal CA:** protocols like SCEP, EST, CMP, or the Microsoft AD CS web enrollment.
- The CA first **verifies** the requester (domain control via DNS/HTTP challenge, or organization validation) before signing.
- The signed certificate is returned over the same channel.

## 3. What key does the CA use to sign certificates? Which file?
The CA's **private key**: `CA/pki/private/ca.key`.

The matching public certificate is `CA/pki/ca.crt`. Clients use it to verify the signature on `CA/pki/issued/nuces.crt`.

## 4. Why does the client browser show a security error?
The server certificate was signed by our **own private CA**, whose root certificate (`ca.crt`) is **not in the browser's/OS trusted root store**. The browser cannot build a chain to a trusted root, so it warns (e.g. `NET::ERR_CERT_AUTHORITY_INVALID`).

Other possible causes:
- The certificate's CN (`FAST University`) does not match the site's hostname. Modern browsers require a matching **Subject Alternative Name (SAN)**, and this CSR has none.

**Fix:** import `ca.crt` into the browser/OS trusted root store, and issue the cert with a SAN matching the hostname.
