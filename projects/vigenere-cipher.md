---
title: Vigènere Cipher
layout: project
permalink: /projects/vigenere-cipher
tags: python
---
The [Vigènere cipher](https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher) is a polyalphabetic substitution cipher for encrypting information. A message, the plaintext, is encrypted by taking the first character of the message and the first character of the supplied key and performing a basic mathematic operation to determine the appropriate ciphertext.

### Project code
[GitHub](https://github.com/jongrim/VigenereCipher)

### Languages
- Python

### Notable code snippet
I took an object-oriented to approach when working on this program, and so I created a class called `VigenereMachine`. When created, the class can be supplied with some combination of plaintext, cipher key, or ciphertext. The class then provides methods to perform the encryption and decryption functions, each time updating its stored values to preserve state. Below are snippets showing the class definition, the `encrypt` method that is available to be called, and the helper method `get_encrypted_letter` which performs the actual mathematical computation for producing the ciphertext.
```python
class VigenereMachine:
    num_dict = {num: alpha for num, alpha in enumerate(string.ascii_lowercase)}
    alpha_dict = {alpha: num for num, alpha in enumerate(string.ascii_lowercase)}

    def __init__(self, plaintext=None, cipher_key=None, ciphertext=None):
        if plaintext:
            self._plaintext = plaintext.lower()
        if cipher_key:
            if not cipher_key.isalpha():
                raise ValueError('The key may only contain alphabetic characters')
            else:
                self._cipher_key = cipher_key.lower()
        if ciphertext:
            self._ciphertext = ciphertext.lower()

    def encrypt(self):
        self._ciphertext = []
        key_position = 0
        for letter in self._plaintext:
            if letter.isalpha():
                self._ciphertext.append(self.get_encrypted_letter(key_position, letter))
                key_position += 1
            else:
                self._ciphertext.append(letter)
        self._ciphertext = ''.join(self._ciphertext)
        return self._ciphertext

    def get_encrypted_letter(self, key_position, letter):
        return VigenereMachine.num_dict[
            (self.alpha_dict[letter] + self.alpha_dict[self._cipher_key[key_position % len(self._cipher_key)]]) % 26]
```

### Final notes
This was a fun project for me because I've always been fascinated by encryption methodologies and view encryption as one of the single most important tools we have for security and privacy.
