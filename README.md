# ICHA - Isolated Cipher with Hand-delivered Authentication

A portable Windows application for encrypting files so that only one intended recipient can open them. Each encrypted file comes with a one-time key file that is destroyed after the file has been successfully opened.

Est. 2026

## Overview

ICHA is built for people who want to send a file to someone else and keep it private, without relying on an account, a server, or an internet connection. Everything happens on your own computer. The encrypted file and its key are handed over separately, and the key is meant to travel offline, for example on a flash drive or in person.

The application works with any type of file: documents, photos, videos, archives, and so on. It is a single executable with a simple graphical interface. Nothing needs to be installed, and no other software is required.

The interface is in Indonesian.

## Features

- Encrypts any file type, regardless of size or format.
- Each package is addressed to one specific recipient. Nobody else can open it, including the sender.
- Opening a package requires two separate things: the recipient's identity file (protected by a PIN) and the one-time key file that was created with the package. Having only one of them is not enough.
- Every package is digitally signed. If even a single byte is changed after encryption, the package is rejected.
- The original file name is stored inside the encrypted content. The encrypted file and its key file are given random names, so the original name is not visible from the outside.
- File contents are split into equal parts and stored in a shuffled order inside the package.
- After a package has been opened successfully, the key file is scrambled, overwritten several times, and deleted.
- Portable. The application can run from a flash drive and leaves nothing behind on the computer.

## How it works

The process involves two people, a sender and a recipient, and four kinds of files.

1. **The recipient creates an identity.** This produces a private identity file and a public key file. The private file stays with the recipient. The public key file is given to the sender.
2. **The sender encrypts a file.** The sender loads their own identity, chooses the file to protect, and selects the recipient's public key file. The application produces an encrypted package and a matching key file.
3. **The key file is delivered offline.** The package can be sent by any normal means, such as email or a file sharing service. The key file should be delivered through a different route, ideally by hand or on a physical device.
4. **The recipient decrypts.** The recipient loads their identity, selects the package and the key file, and confirms the sender's fingerprint. The original file is restored, and the key file is destroyed.

### File types

| Extension | Purpose | Who holds it |
|-----------|---------|--------------|
| `.iid` | Private identity file, protected by a PIN | Owner only. Never share it. |
| `.ipub` | Public key file | Shared with anyone who wants to send files to the owner |
| `.icha` | The encrypted package | Can be sent by any normal means |
| `.ikey` | One-time key file for a single package | Delivered offline, destroyed after use |

## Requirements

- Windows 10 or Windows 11, 64-bit.
- About 200 MB of free memory when loading an identity.
- Permission to write to the temporary folder, which the single-file version uses when it starts.

## Download and first run

Download `ICHA-ENCRYPTOR-AND-DECRYPTOR.exe` from the Releases section of this repository. No installation is needed. Place it anywhere, such as a folder or a flash drive, and run it.

Before running it for the first time, verify the file. Each release includes a SHA-256 hash. In PowerShell, run:

```
Get-FileHash .\ICHA-ENCRYPTOR-AND-DECRYPTOR.exe -Algorithm SHA256
```

The result must match the hash published with the release. If it does not, do not run the file.

Windows SmartScreen may show a warning the first time because the executable is not signed with a commercial certificate. Choose "More info" and then "Run anyway" only after the hash has been verified. Some antivirus programs may also flag the file as a false positive, which is common for applications packaged from Python.

It is recommended to keep the executable and your `.iid` identity file together on a removable drive that you control.

## Using the application

### Identity tab

To create an identity, enter a username, choose a PIN, and select a folder. Two files are created: one `.iid` and one `.ipub`. Keep the `.iid` file safe and make a backup. Give the `.ipub` file to people who will send you files.

To use an identity, select the `.iid` file, enter its PIN, and press the load button. The username and fingerprint of the active identity are shown at the top of the window. The fingerprint is what other people use to confirm that a key really belongs to you.

The PIN must be at least 6 characters. A long passphrase is strongly recommended over a short number. The application warns when a short numeric PIN is used.

### Encrypt tab

Select the file, the recipient's `.ipub` file, and an output folder. Before encryption starts, the application shows the recipient's name and fingerprint. Compare the fingerprint with the recipient through a separate channel, such as a phone call or in person, and continue only if they match. The result is one `.icha` file and one `.ikey` file with the same random name.

### Decrypt tab

Select the `.icha` file, the `.ikey` file, and an output folder. The application shows the sender's name and fingerprint for confirmation. After a successful decryption, the original file is saved with its original name and the key file is destroyed. If decryption fails for any reason, the key file is left untouched so that another attempt can be made.

### About tab

Shows the application name, the year, and links to the author's GitHub page and website.

## Security notes

ICHA is designed to keep file contents private and to detect tampering. Its limits are listed here so that it is not trusted for more than it can do.

- **Not independently audited.** The application uses well known, standard building blocks, but the way they are combined has not been reviewed by outside experts. Because the source code is not published, outside review is also not currently possible. It is suitable for personal use and non-critical files. For highly sensitive data, use a widely reviewed and open tool.
- **Key file destruction is best effort.** The key file is overwritten and deleted, but on SSDs, flash storage, and some file systems, older copies of data may remain in places the program cannot reach. Storing key files on a removable drive and destroying or formatting the drive afterwards adds another layer of protection.
- **Copies made before use are not affected.** If someone copies the key file before it is used, destroying the original does nothing to the copy. Handle key files with the same care as the encrypted package itself.
- **The PIN is the main weak point.** If an identity file is stolen, an attacker can try guesses against it offline with no limit on attempts. A long passphrase makes this impractical. A short numeric PIN does not.
- **A compromised computer defeats everything.** Malware that records keystrokes or reads memory can capture the PIN and the decrypted file. Only use ICHA on computers you trust.
- **Some information is visible.** The sender's name and public key, the time of sending, the recipient's fingerprint, and the approximate size of the file can be seen without any key. The file name and contents cannot.
- **Nothing can be recovered if a secret is lost.** A forgotten PIN, a lost identity file, or a lost key file means the data cannot be opened. There is no reset option and no backdoor.
- **Verify fingerprints.** The signature proves that a package was signed by a certain key, not who owns that key. Matching fingerprints through a separate channel is the only protection against someone impersonating the sender or the recipient.
- **Only use official releases.** Obtain the executable only from this repository and verify its hash. A modified copy could steal your PIN.

## Cryptography used

ICHA does not invent its own algorithms. It relies on standard ones:

- X25519 for agreeing on a shared secret between sender and recipient
- Ed25519 for digital signatures
- HKDF with SHA-256 for deriving keys
- ChaCha20-Poly1305 for encryption with built-in tamper detection
- scrypt for turning a PIN into a key that protects the identity file

The original part of the project is the package format, the way files are divided and shuffled, and the one-time key file workflow.

## License

This software is not open source. The source code is not published. The executable is provided free of charge for personal use, without warranty of any kind. All rights reserved.

## Author

Ramdan Olii

- GitHub: [github.com/ramdanolii14](https://github.com/ramdanolii14)
- Website: [ramdanolii.my.id](https://ramdanolii.my.id)

Est. 2026
