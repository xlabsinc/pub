# API Key Encrypt/Decrypt

A single-file browser utility for securely sharing API keys and secrets via text messages or any untrusted channel.

**Live URL:** https://xlabsinc.github.io/pub/apikey/apikey.html

---

## Purpose

When you need to share an API key or sensitive secret with someone over an insecure channel (SMS, Slack, email), you encrypt it first and send the encrypted hex blob. The recipient pastes it into the tool with the same password and recovers the original secret. No server is involved — everything runs in the browser.

---

## Shell equivalents

The tool is fully interoperable with standard OpenSSL commands. Every encrypt/decrypt operation in the UI produces and consumes the exact same bytes as:

**Encrypt (shell → UI-readable):**
```bash
echo "my-api-key" | openssl aes-256-ecb -a -pass "pass:<password>" -e | xxd -u
```

**Decrypt (UI output → shell):**
```bash
cat encrypted.txt | xxd -r | openssl aes-256-ecb -a -pass "pass:<password>" -d
```

You can encrypt in the UI and decrypt in the shell, or encrypt in the shell and decrypt in the UI — both directions work identically.

---

## Encryption format

The tool follows the standard OpenSSL salted binary format:

```
+----------+----------+-----------------+
| Salted__ | 8B salt  |   ciphertext    |
| (8 bytes)| (random) | (AES-256-ECB)   |
+----------+----------+-----------------+
```

1. **Random 8-byte salt** is generated for every encryption — the same plaintext+password produces different ciphertext each time.
2. The above structure is Base64-encoded (the OpenSSL `-a` flag).
3. The Base64 string is then hex-encoded in `xxd` format with offset addresses, producing output like:
   ```
   00000000: 5532 4673 6447 566B 5831 2B47 ...
   00000010: ...
   ```

---

## Cryptographic details

| Property          | Value                                             |
|-------------------|---------------------------------------------------|
| Cipher            | AES-256-ECB                                       |
| Key size          | 256 bits (32 bytes)                               |
| Padding           | PKCS#7                                            |
| Key derivation    | EVP_BytesToKey, 1 iteration, **SHA-256**          |
| Salt              | 8 bytes, random per encryption                    |
| Output encoding   | Base64 → xxd hex (offset-addressed, 2-byte groups)|
| Library           | CryptoJS 4.2.0 (CDN, no server)                  |

> **Note on hash:** Modern OpenSSL (1.1.0+) uses SHA-256 as the default hash for `EVP_BytesToKey`. Older material may reference MD5 — this tool uses SHA-256 to match the current OpenSSL default.

---

## How the code works

### `deriveKey(password, salt)`
Runs OpenSSL's `EVP_BytesToKey` key derivation via `CryptoJS.EvpKDF`:
- Input: password string + 8-byte salt
- Hash: SHA-256, 1 iteration
- Output: 32-byte (256-bit) AES key

### `encryptSecret(plainText, password)`
1. Generates a random 8-byte salt.
2. Derives the 32-byte key via `deriveKey`.
3. Encrypts with AES-256-ECB + PKCS#7 padding using CryptoJS.
4. Prepends `Salted__` + salt to the raw ciphertext bytes.
5. Base64-encodes the result + appends a newline (matching OpenSSL `-a` output).
6. Converts the Base64 string character-by-character to uppercase hex.
7. Formats the hex into `xxd`-style offset lines via `formatHexOutput`.

### `decryptSecret(hexString, password)`
1. Strips xxd offset addresses (e.g. `00000000:`) and all non-hex characters.
2. Converts the cleaned hex back to the Base64 string (reversing the xxd hex encoding).
3. First attempts CryptoJS default AES-CBC decrypt (handles UI-encrypted blobs directly).
4. On failure, falls back to manual ECB path:
   - Base64-decodes to raw bytes.
   - Validates the `Salted__` prefix (bytes 0–7).
   - Extracts the 8-byte salt (bytes 8–15) and ciphertext (bytes 16+).
   - Derives the key from password + salt via `deriveKey`.
   - Decrypts using `CryptoJS.algo.AES.createDecryptor` with ECB mode and PKCS#7 padding.
5. Returns the decrypted UTF-8 string.

### `formatHexOutput(hexString)`
Formats a raw uppercase hex string into `xxd`-compatible output:
- Each line starts with an 8-digit hex offset followed by `:`.
- Hex bytes are grouped in pairs of 2 bytes (4 hex chars), space-separated.
- Lines are capped at 50 characters to match `xxd -u | cut -c1-50`.

---

## Usage

1. Open `apikey.html` in a browser (no server required — it is a static file).
2. **Encrypt:**
   - Select **Encrypt** mode.
   - Paste your API key or secret into the input box.
   - Enter a shared password.
   - Click **Submit** — copy the xxd hex output and send it.
3. **Decrypt:**
   - Select **Decrypt** mode.
   - Paste the xxd hex blob received from the sender.
   - Enter the shared password.
   - Click **Submit** — the original secret is displayed.
4. Use **Save Output** or **Copy to Clipboard** for convenience.
5. You may also upload a file instead of typing text.

---

## Security notes

- The password is never sent anywhere — all computation is local, in-browser.
- AES-256-ECB does not authenticate ciphertext; a wrong password produces garbage output or an error, not a verified failure.
- A unique random salt is used per encryption, so identical plaintexts produce different ciphertext blobs.
- For highly sensitive keys, prefer an end-to-end encrypted channel in addition to this encryption layer.
