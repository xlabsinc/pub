# crypt/csv.html

`crypt/csv.html` is a self-contained HTML page that provides a web-based tool for encrypting and decrypting text (or file contents) using AES encryption with password-based key derivation. It offers a modern, responsive interface suitable for desktop and mobile browsers.

---

### **Main Functionality**

- **Encrypt/Decrypt Text or File Contents**
  - Users can input text directly or upload a file (such as CSV or plain text). Each line is processed separately.
  - A password is provided and used to derive a strong AES key with PBKDF2 and salt.
  - Users select between “Encrypt” and “Decrypt” modes via a dropdown menu.
  - Encryption produces AES-encrypted, Base64-encoded output; decryption reverses this process.

---

### **UI Elements**

- **Mode Dropdown**: Select between Encrypt or Decrypt, placed prominently at the top for better UX.
- **File Upload**: Load a file’s contents directly into the input textarea.
- **Input Textarea**: Multi-line input for text or file contents (one line per entry).
- **Password Field**: Enter the encryption/decryption password with an option to show/hide the password.
- **Submit Button**: Processes the input using the selected mode.
- **Reset Button**: Clears all fields and resets selections.
- **Output Textarea**: Displays the encrypted or decrypted result, auto-resizes with content and supports manual resizing with a scrollbar.
- **Save Output Button**: Download the output as a text file.

---

### **How the Algorithm Works**

- **Encryption**
  1. The password is processed with PBKDF2 and a fixed salt to derive an AES key.
  2. For each line, a random initialization vector (IV) is generated.
  3. The line is encrypted with AES-CBC mode using the key and IV.
  4. The IV is prepended to the ciphertext, and the result is Base64-encoded for output.

- **Decryption**
  1. The Base64-encoded input is decoded.
  2. The first 16 bytes are extracted as the IV.
  3. The remaining bytes are the ciphertext.
  4. The ciphertext is decrypted with AES-CBC mode using the derived key and IV.
  5. The original plaintext line is recovered.

---

### **Use Cases**

- Securely encrypt or decrypt text data directly in the browser without server involvement.
- Encrypt or decrypt CSV or plain text files line-by-line using a password.
- A simple, portable AES tool that works on desktops and mobile devices.

---

### **Security Note**

- Uses AES encryption with a standard key derivation function (PBKDF2) and random IVs for each line, making it significantly more secure than simple XOR.
- Suitable for basic client-side encryption needs but not a replacement for comprehensive cryptographic solutions in production environments.

---

**Summary:**  
This file implements a client-side AES encryption/decryption tool with a clean, responsive UI. It enables secure line-by-line processing of text or files with password-based key derivation and output saving functionality.

---


