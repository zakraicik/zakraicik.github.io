---
date: 2022-10-23 00:00:00
title: Public Cryptography with ECDSA Node
subtitle: Signature Basics
image: /uploads/public-key-cryptography.jpeg
---
You can run the react app [here](https://github.com/zakraicik/ecdsa-node).&nbsp;

Public key cryptography, also known as asymmetric cryptography, is a cryptographic system that uses two separate keys for encryption and decryption processes: a public key and a private key. These keys are mathematically related but cannot be feasibly derived from one another. This system allows for secure communication and data exchange between parties without the need for sharing a common secret key.

The public key is openly shared and can be used by anyone to encrypt a message or verify a digital signature. The private key, on the other hand, is kept secret by its owner and is used to decrypt messages or create digital signatures.

Public key cryptography offers several benefits, including:

1. Secure key exchange: Participants can securely exchange keys over an insecure channel without the risk of eavesdropping, as the public key can be freely shared without compromising the private key.
2. Digital signatures: This allows for verifying the authenticity and integrity of a message, as the sender can sign the message using their private key, and the receiver can verify the signature using the sender's public key.
3. Confidentiality: Encrypted messages can only be decrypted by someone possessing the corresponding private key, ensuring that the message remains confidential.

ECDSA, or Elliptic Curve Digital Signature Algorithm, is a widely-used digital signature algorithm based on elliptic curve cryptography (ECC) in public key cryptography. It is an adaptation of the Digital Signature Algorithm (DSA) that uses elliptic curve mathematics instead of traditional modular arithmetic. ECDSA provides a similar level of security as RSA and DSA but with smaller key sizes, resulting in faster computations and lower resource usage.

The main components of ECDSA are:

1. Elliptic curve: A mathematical curve defined by a specific equation. In ECDSA, the curve is chosen from a set of standardized curves that have been extensively studied and are considered secure.
2. Key pair generation: A private key is generated as a random number within a predefined range, and the corresponding public key is calculated by multiplying the private key with a predefined point (called the generator point) on the elliptic curve.
3. Signing: To sign a message, the sender generates a unique signature for the message using their private key and the elliptic curve mathematics. This signature ensures the authenticity and integrity of the message.
4. Verification: To verify the signature, the receiver uses the sender's public key and the elliptic curve mathematics to check whether the signature is valid. If the signature is valid, it confirms that the message was indeed sent by the sender and has not been tampered with.

ECDSA has become popular in various applications, including secure communications, cryptocurrencies (e.g., Bitcoin and Ethereum), and secure authentication. Its main advantage is the smaller key size compared to RSA, which allows for more efficient cryptographic operations while maintaining a similar level of security.

**In this project, we build a simple react application that allows us to use a private key(never do this in practice) to look up user balances, and sign/execute transactions.** The screenshot below shows the final product.&nbsp;

![](/uploads/final-app.png){: width="2232" height="696"}

# Repository Design

---

This repository maintains both a client and a server. Technically, the client side is responsible for presenting information to the user, handling user interactions, and performing tasks on the user's device. On the other hand, the&nbsp; server side of an application typically involves processing and storing data, handling business logic, and providing access to resources such as databases, APIs, and other services. Our repository looks like this.&nbsp;

![](/uploads/repository-design.png){: width="660" height="1122"}

In our repository, the client side is&nbsp;

#