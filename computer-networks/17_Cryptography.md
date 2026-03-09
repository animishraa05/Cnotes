
# 17. Cryptography

## Introduction to Cryptography

Cryptography is the practice and study of techniques for secure communication in the presence of third parties called adversaries. More generally, cryptography is about constructing and analyzing protocols that prevent third parties or the public from reading private messages. Modern cryptography exists at the intersection of the disciplines of mathematics, computer science, electrical engineering, communication science, and physics. Applications of cryptography include electronic commerce, chip-based payment cards, digital currencies, computer passwords, and military communications.

## Types of Cryptography

There are two main types of cryptography: symmetric cryptography and asymmetric cryptography.

### Symmetric Cryptography

Symmetric cryptography, also known as secret-key cryptography, is a type of cryptography where the same key is used for both encryption and decryption. This means that the sender and the receiver must both have the same key. Symmetric cryptography is fast and efficient, but it can be difficult to securely share the key.

Some common symmetric cryptography algorithms include:

- **Data Encryption Standard (DES):** DES is a symmetric-key algorithm for the encryption of electronic data. It was developed in the 1970s and is now considered to be insecure.
- **Advanced Encryption Standard (AES):** AES is a symmetric-key algorithm that was developed in the 1990s. It is now the most widely used symmetric-key algorithm.

### Asymmetric Cryptography

Asymmetric cryptography, also known as public-key cryptography, is a type of cryptography where two different keys are used for encryption and decryption. One key, called the public key, is used for encryption. The other key, called the private key, is used for decryption. The public key can be shared with anyone, but the private key must be kept secret.

Asymmetric cryptography is slower and less efficient than symmetric cryptography, but it is more secure. This is because it is not necessary to share a secret key.

Some common asymmetric cryptography algorithms include:

- **RSA:** RSA is an asymmetric-key algorithm that was developed in the 1970s. It is now the most widely used asymmetric-key algorithm.
- **Elliptic Curve Cryptography (ECC):** ECC is an asymmetric-key algorithm that is based on the mathematics of elliptic curves. It is more efficient than RSA and is becoming increasingly popular.

## Hash Functions

A hash function is a mathematical function that takes an input of any size and produces an output of a fixed size. The output of a hash function is called a hash. A hash function has two important properties:

- **It is one-way:** It is very difficult to reverse a hash function. This means that it is very difficult to find the input that produced a given hash.
- **It is collision-resistant:** It is very difficult to find two different inputs that produce the same hash.

Hash functions are used for a variety of purposes, including:

- **Password storage:** Hash functions are used to store passwords in a secure way. When a user creates a password, the password is hashed and the hash is stored in the database. When the user logs in, the password that they enter is hashed and the hash is compared to the hash in the database. If the two hashes match, then the user is logged in.
- **Data integrity:** Hash functions are used to verify the integrity of data. When a file is created, a hash of the file is created and stored with the file. When the file is opened, the hash of the file is recalculated and compared to the stored hash. If the two hashes match, then the file has not been modified.
