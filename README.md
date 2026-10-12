# IS Assignment 2 – Private CA & HTTPS Server

Builds a small PKI with EasyRSA: a Root CA signs a certificate for a server, and the server uses it for HTTPS.

## Layout

| Path | Purpose |
|---|---|
| `CA/` | EasyRSA instance for the Root CA (`CyberShield Global Root Authority`) |
| `CA/pki/private/ca.key` | CA private key, used to sign certificates (passphrase-protected) |
| `CA/pki/ca.crt` | CA root certificate (public) – clients must trust this |
| `CA/pki/reqs/nuces.req` | CSR received from the server |
| `CA/pki/issued/nuces.crt` | Signed server certificate |
| `Server/` | EasyRSA instance for the server (`CN = FAST University`) |
| `Server/pki/private/nuces.key` | Server private key (never leaves the server) |
| `Server/pki/reqs/nuces.req` | Server CSR |
| `Answers.md`, `Answers.docx` | Written answers with screenshots |

## Steps

1. **Server – create key and CSR**
   ```bash
   cd Server
   ./easyrsa init-pki
   ./easyrsa gen-req nuces nopass      # CN = FAST University
   ```
2. **Copy CSR to CA** (manual copy here; ACME/SCEP/EST in production)
   ```bash
   cp Server/pki/reqs/nuces.req CA/pki/reqs/nuces.req
   ```
3. **CA – sign the request**
   ```bash
   cd CA
   ./easyrsa init-pki
   ./easyrsa build-ca                  # creates ca.key and ca.crt
   ./easyrsa import-req pki/reqs/nuces.req nuces   # skip if already in pki/reqs
   ./easyrsa sign-req server nuces     # creates pki/issued/nuces.crt
   ```
4. **Run the HTTPS server** with `nuces.crt` and `nuces.key` (e.g. Python `ssl` + `http.server`, `https://localhost:8443`).
5. **Verify**
   ```bash
   openssl verify -CAfile CA/pki/ca.crt CA/pki/issued/nuces.crt
   curl --cacert CA/pki/ca.crt https://localhost:8443/
   ```

## Known issues

- Browser shows `NET::ERR_CERT_AUTHORITY_INVALID` until `ca.crt` is imported into the browser/OS trusted root store.
- The certificate has no Subject Alternative Name and its CN (`FAST University`) does not match `localhost`, so hostname checks fail. Fix by issuing with `--subject-alt-name=DNS:localhost` (`./easyrsa --subject-alt-name="DNS:localhost" sign-req server nuces`).

## Security

`CA/pki/private/ca.key` and `Server/pki/private/nuces.key` are secrets. Do not commit or share them.
