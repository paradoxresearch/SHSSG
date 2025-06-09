# Using GPG for Secure Encryption and Decryption

## Contents

- [Using GPG for Secure Encryption and Decryption](#using-gpg-for-secure-encryption-and-decryption)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding GPG and Public-Key Cryptography](#understanding-gpg-and-public-key-cryptography)
  - [GPG Key Pair Generation](#gpg-key-pair-generation)
  - [Encrypting Data with GPG](#encrypting-data-with-gpg)
    - [Encrypting a File Using a Recipient's Public Key](#encrypting-a-file-using-a-recipients-public-key)
    - [Encrypting a File for Multiple Recipients](#encrypting-a-file-for-multiple-recipients)
  - [Decrypting Data with GPG](#decrypting-data-with-gpg)
    - [Decrypting a File Using the Corresponding Private Key](#decrypting-a-file-using-the-corresponding-private-key)
  - [Managing GPG Keys](#managing-gpg-keys)
    - [Exporting and Importing Public Keys for Sharing](#exporting-and-importing-public-keys-for-sharing)
      - [Exporting Your Public Key](#exporting-your-public-key)
      - [Importing Public Keys](#importing-public-keys)
    - [Revoking and Exipring Keys](#revoking-and-exipring-keys)
      - [Key Expiration](#key-expiration)
      - [Key Revocation](#key-revocation)
    - [Backing Up your Private Key](#backing-up-your-private-key)
    - [Restoring a Backed Up Private Key](#restoring-a-backed-up-private-key)
    - [Trusting Public Keys](#trusting-public-keys)

## Introduction

### Understanding GPG and Public-Key Cryptography

GPG, or **GNU Privacy Guard**, is a free and open-source implementation of the OpenPGP standard. Think of it as a digital padlock and key system for your data. Its primary purpose is to allow you to securely encrypt and decrypt data, ensuring that only the intended recipient can read it. It also enables you to digitally sign data, verifying its authenticity and integrity.

Now, the core concept behind GPG's power is **public-key cryptography**, also known as **asymmetric encryption**. To understand this, let's use a simple analogy: imagine you have a special mailbox with two slots and two unique keys.

- One slot is for **sending messages**, and it's unlocked by your private key. This key is something you keep absolutely secret and never share.
- The other slot is for **receiving messages**, and it's locked with your public key. You can give copies of this public key to anyone.

Here's how it works: If someone wants to send you a secret message, they use your publicly available "receiving" key to lock the message. Once locked, only *your* private key can open it. Even the person who locked the message can't unlock it without your private key.

This is a stark contrast to **symmetric encryption**, where you use the same key for both encrypting and decrypting data. While symmetric encryption is faster, sharing that single key securely can be a challenge. Public-key cryptography elegantly solves this "key exchange problem" by allowing you to share your public key freely without compromising security.

Why is this essential for secure communication and data storage? Because it allows you to:

- **Confidentiality**: Send sensitive data to someone knowing only they can read it.
- **Authentication**: Verify that a message truly came from the person it claims to be from.
- **Integrity**: Ensure that a message hasn't been tampered with since it was signed.

## GPG Key Pair Generation

To use GPG, you first need to create your own unique key pair: a public key and a private key. This pair acts as your digital identity for encryption and decryption.

Here's how you generate a GPG **key pair** in the Linux terminal:

```Bash
gpg --full-generate-key
```

When you run this command, GPG will prompt you for several pieces of information:

1. **Kind of key**: For general use, the default `(1) RSA and RSA` is usually a good choice. This means both the encryption and signing capabilities will use the RSA algorithm.
2. **Key size**: This determines the strength of your key. A larger key size provides more security but takes longer to generate and use. For modern security, a key size of at least `4096` bits is highly recommended.
3. **Key expiration**: You can set an expiration date for your key. This is a good security practice, as it means even if your key were compromised, it would eventually become invalid. Many choose `0` for no expiration, but it's often better to set one (e.g., 1 year) and renew it later.
4. **User ID information**: You'll be asked for your real name, email address, and an optional comment. This information is used to identify your public key to others.
5. **Passphrase**: This is critically important. Your passphrase protects your private key. Think of it as the strong lock on your secret key. If someone gets hold of your private key file, they still can't use it without this passphrase. **Choose a strong, unique passphrase that you won't forget but is hard for others to guess**. The longer and more complex, the better!

As GPG generates your key, it will ask you to perform some random actions (like typing or moving your mouse). This is to gather sufficient entropy (randomness) to create a truly unique and secure key.

Once generated, your public key can be shared with anyone who wants to send you encrypted messages or verify your digital signatures. Your private key, however, must be kept absolutely secure and never shared.

## Encrypting Data with GPG

### Encrypting a File Using a Recipient's Public Key

This is where the power of public-key cryptography really shines. To encrypt a file for someone else, you need their public key. You don't need their private key, and they don't need yours. This means you can send sensitive information securely without ever sharing a secret key directly.

Imagine you have a file named `secret_report.txt` that you need to send to a colleague, let's call her Alice, and you want to ensure only Alice can read it. You would use her public key to encrypt the file.

The general command structure for encrypting a file with GPG for a specific recipient is:

```Bash
gpg --encrypt --recipient "Alice's Name or Email" secret_report.txt
```

Let's break down this command:

- `gpg --encrypt`: This tells GPG that you want to encrypt a file.
- `--recipient "Alice's Name or Email"`: This is crucial. You specify the identity of the person for whom you are encrypting the file. GPG will then look for Alice's public key in your keyring (a collection of GPG keys you have). The identity can be her full name or email address, as entered when her key was generated.
- `secret_report.txt`: This is the input file you want to encrypt.

After running this command, GPG will typically create an encrypted output file with a `.gpg` or `.asc` extension (if you add `--armor` for ASCII armored output, which makes it suitable for email). For example, `secret_report.txt` would become `secret_report.txt.gpg`. This new file is now unintelligible to anyone without Alice's corresponding private key and passphrase.

**Important Note**: Before you can encrypt for Alice, you first need to have her public key on your system. We'll cover how to import keys in a later step, but for now, assume you have it.

### Encrypting a File for Multiple Recipients

Sometimes, you might need to send a secure message or file to more than one person. GPG handles this gracefully! You don't have to encrypt the file separately for each recipient. Instead, you can specify multiple recipients in a single command.

The process is very similar to encrypting for a single recipient, you just list all the individuals for whom you want to encrypt the file.

Let's say you have a file `project_update.txt` that needs to be securely shared with both Alice and Bob. Assuming you have both their public keys, the command would look like this:

```Bash
gpg --encrypt --recipient "Alice's Name or Email" --recipient "Bob's Name or Email" project_update.txt
```

As you can see, you simply add an additional `--recipient` flag for each person you want to be able to decrypt the file. When you run this command, GPG encrypts the file in such a way that both Alice's private key and Bob's private key can decrypt it. Each recipient will only need their own private key to access the content.

This is incredibly efficient for team collaboration or distributing sensitive information to a select group.

## Decrypting Data with GPG

### Decrypting a File Using the Corresponding Private Key

You've received an encrypted file – perhaps `secret_message.txt.gpg` – and now you need to read its contents. This is where your private key and your trusty passphrase come into play. Only the private key corresponding to one of the public keys used for encryption can decrypt the file.

The process is remarkably straightforward:

```Bash
gpg --decrypt secret_message.txt.gpg
```

Let's break this down:

- `gpg --decrypt`: This tells GPG that you want to decrypt a file.
- `secret_message.txt.gpg`: This is the encrypted file you want to decrypt.

When you run this command, GPG will:

1. Identify which of your private keys can decrypt the file.
2. Prompt you for the passphrase associated with that private key. This is why a strong passphrase is so important – it's the final lock protecting your data even if someone gets your private key file.

Once you enter the correct passphrase, GPG will decrypt the file. By default, it will output the decrypted content directly to your terminal. If you want to save the decrypted content to a new file, you can redirect the output:

```Bash
gpg --decrypt secret_message.txt.gpg > decrypted_message.txt
```

This command will take the decrypted output and save it into a new file called `decrypted_message.txt`.

**Key Point**: Your private key is like the unique key to your digital safe. Without it, and its protective passphrase, encrypted data meant for you remains inaccessible.

## Managing GPG Keys

As a system administrator, you'll likely accumulate several GPG keys over time – your own, keys of colleagues, keys for specific servers or services. Knowing how to view and manage these keys is crucial.

GPG keeps all the keys it knows about in what's called a keyring. You can think of it as a digital address book for all your GPG contacts and your own identity.

To list the public keys you have in your keyring, you use the following command:

```Bash
gpg --list-keys
```

This command will display information about all the public keys GPG knows about, including:

- **pub**: The public key itself, along with its ID and creation date.
- **uid**: The User ID associated with the key (e.g., "John Doe <john.doe@example.com>").
- **sub**: Subkeys, which are often used for specific purposes like encryption or signing, separate from the primary key.

To list your private keys (the secret ones you generated), you can use:

```Bash
gpg --list-secret-keys
```

This command will show similar information, but specifically for the private keys you hold. It's important to remember that these are the keys you should guard closely!

Knowing how to list your keys is the first step in managing them effectively. It allows you to verify that keys have been imported correctly, check their expiration dates, and confirm the identities associated with them.

### Exporting and Importing Public Keys for Sharing

The whole point of public-key cryptography is to share your public key so others can encrypt data for you, and you can import their public keys to encrypt data for them.

#### Exporting Your Public Key

When you generate a GPG key pair, your **public key** is what you give to others. It's perfectly safe to share, as it can only encrypt data for you, not decrypt data from you.

To export your public key, you typically use your User ID (name or email) that you associated with the key:

```Bash
gpg --output my_public_key.asc --armor --export "Your Name or Email"
```

Let's break that down:

- `--output my_public_key.asc`: This specifies the name of the file where your public key will be saved. The `.asc` extension is common for "ASCII armored" files, which means the key data is converted into readable text.
- `--armor`: This flag tells GPG to output the key in ASCII armored format. This is important because it makes the key suitable for pasting into emails, web pages, or other text-based communication. Without `--armor`, the output would be binary, which isn't easily shareable.
- `--export "Your Name or Email"`: This tells GPG to export the public key associated with that specific User ID.

After running this, you'll have a file like my_public_key.asc that you can send to anyone who needs to encrypt data for you.

#### Importing Public Keys

Conversely, when someone wants to send you an encrypted file, they'll send you their public key. You need to **import** this key into your GPG keyring so that GPG knows about it and can use it for encryption.

Let's say Alice sent you her public key file, `alice_public_key.asc`. You would import it like this:

```Bash
gpg --import alice_public_key.asc
```

GPG will then add Alice's public key to your keyring. Once imported, you can now use her User ID (e.g., "Alice <alice@example.com>") as a recipient when encrypting files, just as we discussed in Step 3.

This ability to easily exchange public keys is what makes GPG so powerful for secure communication. It's like exchanging digital padlocks without ever needing to share the secret key that opens them!

### Revoking and Exipring Keys

#### Key Expiration

As we discussed, setting an expiration date for your GPG key is a good security practice. If your key has an expiration date, it will automatically become invalid after that date, reducing risk if it's ever compromised.

To **change the expiration date** of your key (or set one if it didn't have one initially), you can edit your key:

```Bash
gpg --edit-key "Your Name or Email"
```

Once in the GPG edit prompt:

1. Type expire
2. Enter the desired expiration time (e.g., `1y` for 1 year, `6m` for 6 months, `0` for no expiration).
3. Type `save` to apply the changes.

#### Key Revocation

Key revocation is the crucial step you take if your private key is ever compromised. This immediately signals to the world that your key should no longer be trusted.

**The most important step is to generate a revocation certificate in advance**, right after you create your key pair, and keep it in a safe, offline location (like a USB drive in a secure vault). If your key is compromised, you can no longer generate this certificate safely on the compromised system.

1. **Generating a Revocation Certificate (PROACTIVE STEP!)**:

    ```Bash
    ```gpg --output revoke.asc --gen-revoke "Your Name or Email"
    ```

    - `--output revoke.asc`: This specifies the file name for your revocation certificate.
    - `--gen-revoke`: This command tells GPG to generate a revocation certificate for the specified key.

    GPG will then guide you through a few prompts, asking for a reason for revocation and a comment. It will then ask for your passphrase to sign the revocation certificate. Once generated, store this `revoke.asc` file very securely, separate from your main system.

2. **Importing/Publishing a Revocation Certificate (REACTIVE STEP!)**:

    If your key is compromised and you need to revoke it, you would use this pre-generated `revoke.asc` file.

    ```Bash
    gpg --import revoke.asc
    ```

    Importing the revocation certificate marks your key as revoked in your local keyring. To effectively revoke the key for others, you would then need to publish this `revoke.asc` file to public key servers (similar to how public keys are distributed). This ensures that anyone who looks up your public key will see that it has been revoked.

    **Why is this important?**

    - **Immediate Action**: Allows you to swiftly invalidate a compromised key.
    - **Global Notification**: Publishing the certificate informs others not to trust your key, preventing misuse of a compromised key for encryption or signing.

Revocation is a powerful safety net. While we hope you never need to use it, knowing how to generate and apply a revocation certificate is a cornerstone of responsible GPG key management.

### Backing Up your Private Key

Your private key is stored securely on your system, usually in your `~/.gnupg` directory. However, if your system fails or is lost, you'd lose access to your key and any data encrypted with it. Therefore, backing up your private key is arguably one of the most important security practices for GPG users.

To export your **private (secret) key** for backup, you use a command similar to exporting a public key, but with the `--export-secret-keys` flag:

```Bash
gpg --output my_private_key_backup.asc --armor --export-secret-keys "Your Name or Email"
```

Let's dissect this command:

- `--output my_private_key_backup.asc`: This specifies the name of the file where your private key will be saved. Again, `.asc` is used for ASCII armored output.
- `--armor`: Ensures the output is in readable text format, suitable for copying and storing.
- `--export-secret-keys`: This is the crucial flag that tells GPG you want to export your private key.
- `"Your Name or Email"`: The User ID associated with the private key you wish to back up.

When you run this command, GPG will ask for your **passphrase**. This is because your private key is encrypted with this passphrase, and GPG needs it to correctly export the key.

**Where to store the backup?**

This backup file (`my_private_key_backup.asc`) contains your secret key! It is paramount that you store this file in an extremely secure, offline location. Think of:

- An encrypted USB drive.
- A secure cloud storage service with strong encryption.
- A physical safe if stored on media like a CD/DVD.

**Never store this file on a system connected to the internet unless it's on an encrypted partition, and always remember the passphrase that protects it.**

### Restoring a Backed Up Private Key

Just as crucial as backing up is knowing how to get your key back if you ever need to. This process is essentially the reverse of exporting. If you've had a system crash, moved to a new machine, or simply need to access your GPG functions on a different Linux box, you'll use this method.

To restore your private key from your secure backup file (e.g., `my_private_key_backup.asc`), you will use the `gpg --import` command, just like importing a public key:

```Bash
gpg --import my_private_key_backup.asc
```

When you run this command:

1. GPG will read the contents of `my_private_key_backup.asc`.
2. Since it contains a private key, GPG will prompt you for the passphrase that protects the key within the backup file. You must provide the correct passphrase to successfully import the key.
3. Once imported, your private key will be added to your GPG keyring on the new system, and you'll be able to use it for decryption and signing, just as you would have on your old system.

It's a good practice to then list your secret keys (`gpg --list-secret-keys`) to verify that the key has been successfully imported.

This process ensures that even if your primary system suffers a catastrophic failure, your ability to decrypt sensitive data and sign communications is not lost, as long as you have your secure backup and remember your passphrase.

### Trusting Public Keys

When you import someone else's public key, GPG adds it to your keyring. However, simply having a public key doesn't automatically mean you **trust** that it truly belongs to the person it claims to be. This is where the concept of the "web of trust" comes in.

In GPG, trust is decentralized. Instead of relying on a central authority (like a certificate authority on the web), GPG allows individuals to vouch for the authenticity of public keys. When you "sign" someone's public key, you are essentially saying, "I have verified that this public key truly belongs to this person, and I trust it."

There are different levels of trust you can assign to a key. For a system administrator, it's particularly important to correctly verify and trust the keys of colleagues or services you interact with securely.

To edit the trust level of a public key in your keyring, you first enter the key editing mode for that key:

```Bash
gpg --edit-key "Recipient's Name or Email"
```

Once in the GPG edit prompt (where you see a gpg> prompt):

1. Type `trust`
2. GPG will then present you with several options for the level of trust you wish to assign:
   - `1 = I don't know or won't say`
   - `2 = I do NOT trust`
   - `3 = I trust ultimately`
   - `4 = I trust fully`
   - `5 = I trust marginally`
   - `s = Show me the key's validity`
   - `m = Show me the ownertrust validity`
3. For keys you have personally verified and are certain of their authenticity (e.g., you met the person face-to-face and exchanged key fingerprints), you might choose `4` (I trust fully). If it's your own key, you'd mark it as `3` (I trust ultimately).
4. Type `save` to apply the changes and exit.

**Why is this important?**

- **Security**: GPG uses these trust levels to decide whether to implicitly trust other keys signed by a key you trust. This helps prevent "man-in-the-middle" attacks where an attacker might try to substitute their public key for someone else's.
- **Decentralized Verification**: It puts the power of verification in the hands of the users, creating a network of trust.

This process helps build a chain of trust. If you trust Alice's key, and Alice has signed Bob's key, then GPG might consider Bob's key more trustworthy through its connection to Alice's.
