"""
Utilities.py: Utilities the QuantumKeyDistribution.py.
"""

import random

def text_to_bits(text, encoding='utf-8', errors='surrogatepass'):
    bits = bin(int.from_bytes(text.encode(encoding, errors), 'big'))[2:]
    return bits.zfill(8 * ((len(bits) + 7) // 8))

def text_from_bits(bits, encoding='utf-8', errors='surrogatepass'):
    n = int(bits, 2)
    return n.to_bytes((n.bit_length() + 7) // 8, 'big').decode(encoding, errors) or '\0'

def random_bits(length):
    i = length
    bits = ''
    while i > 0:
        bit = str(random.getrandbits(1))
        bits += bit
        i -= 1
    return bits

def encrypt(binary, key):
    encrypted = ''
    i = 0
    for b in binary:
        if b == '0' and key[i] == '0' or b == '1' and key[i] == '1':
            encrypted += '0'
        else:
            encrypted += '1'
        i += 1      
    return encrypted

def decrypt(binary, key):
    decrypted = ''
    i = 0
    for b in binary:
        if b == '0' and key[i] == '0' or b == '1' and key[i] == '1':
            decrypted += '0'
        else:
            decrypted += '1'
        i += 1     
    return decrypted