+++
title = '信息安全密码学基础 - 实验代码'
date = 2024-06-21T18:54:10+08:00
+++

> 造福下一届，人人有责。

所属实验：西安邮电大学 - 网络空间安全学院 - 密码学基础 - 课程设计实验** 

**实验时间：2024年上半年**

文中所用第三方库使用 pip 自行安装。

另外还有👉[**信息安全密码学基础 - 实验原理**](./crypto-experimental-principles)，这里有各个实验的实验原理。

*免责声明：实验代码仅供参考，请自行设计实现。*

## 实验一 对称加密

### 古典密码Hill密码

随机生成一个2×2和一个3×3的可逆的密钥矩阵，矩阵元素取自Z26。

1. 求这两个矩阵的逆矩阵
2. 分别用这两个密钥矩阵加密消息（学号+姓名），得密文；解密密文，检验是否正确解密了？

{{% fold-block "代码 1-1_Symmetric_Hill" %}}

```python
import numpy as np
from sympy import Matrix

# 定义可见 ASCII 字符的范围
MIN_ASCII = 32
MAX_ASCII = 126
Z = MAX_ASCII - MIN_ASCII + 1  # 字符集大小

# 生成一个随机的 n x n 可逆矩阵，元素取自 Z
def generate_invertible_matrix(n):
    while True:
        matrix = np.random.randint(0, Z, size=(n, n))
        # 检查矩阵行列式与 Z 是否互质，只有互质时矩阵才是可逆的
        if np.gcd(int(round(np.linalg.det(matrix))), Z) == 1:
            return matrix

# 计算矩阵在模 Z 下的逆矩阵
def modular_inverse(matrix, mod):
    mat = Matrix(matrix)
    # 使用 sympy 库计算矩阵的模逆
    inv_mat = mat.inv_mod(mod)
    return np.array(inv_mat).astype(int)

# 生成 2x2 和 3x3 可逆矩阵
matrix_2x2 = generate_invertible_matrix(2)
matrix_3x3 = generate_invertible_matrix(3)

# 计算矩阵的逆矩阵
inverse_2x2 = modular_inverse(matrix_2x2, Z)
inverse_3x3 = modular_inverse(matrix_3x3, Z)

# 将文本转换为数字列表
def text_to_numbers(text):
    # 将字符的 ASCII 码减去 MIN_ASCII
    return [ord(char) - MIN_ASCII for char in text]

# 将数字列表转换为文本
def numbers_to_text(numbers):
    # 将数字加上 MIN_ASCII 转换回字符
    return ''.join(chr(num + MIN_ASCII) for num in numbers)

# Hill 加密函数
def hill_encrypt(message, key_matrix):
    n = key_matrix.shape[0]
    message_numbers = text_to_numbers(message)
    while len(message_numbers) % n != 0:
        message_numbers.append(0)  # 使用最小的可见 ASCII 字符进行填充

    encrypted_numbers = []
    for i in range(0, len(message_numbers), n):
        block = np.array(message_numbers[i:i + n])
        # 矩阵乘法并取模 Z
        encrypted_block = np.dot(key_matrix, block) % Z
        encrypted_numbers.extend(encrypted_block)

    return numbers_to_text(encrypted_numbers)

# Hill 解密函数
def hill_decrypt(encrypted_message, inverse_key_matrix):
    n = inverse_key_matrix.shape[0]
    encrypted_numbers = text_to_numbers(encrypted_message)

    decrypted_numbers = []
    for i in range(0, len(encrypted_numbers), n):
        block = np.array(encrypted_numbers[i:i + n])
        # 使用逆矩阵进行矩阵乘法并取模 Z
        decrypted_block = np.dot(inverse_key_matrix, block) % Z
        decrypted_numbers.extend(decrypted_block)

    return numbers_to_text(decrypted_numbers).rstrip(chr(0))  # 移除填充

# 实验 ①: 打印生成的矩阵及其逆矩阵
print("实验 ①: 生成的矩阵及其逆矩阵")
print("2x2 矩阵:")
print(matrix_2x2)
print("2x2 逆矩阵:")
print(inverse_2x2)
print("\n3x3 矩阵:")
print(matrix_3x3)
print("3x3 逆矩阵:")
print(inverse_3x3)

# 实验 ②: 加密和解密消息
message = "26221013_Daaihang_Wong"

# 仅保留消息中的可见字符
filtered_message = ''.join(filter(lambda x: MIN_ASCII <= ord(x) <= MAX_ASCII, message))

encrypted_message_2x2 = hill_encrypt(filtered_message, matrix_2x2)
decrypted_message_2x2 = hill_decrypt(encrypted_message_2x2, inverse_2x2)

encrypted_message_3x3 = hill_encrypt(filtered_message, matrix_3x3)
decrypted_message_3x3 = hill_decrypt(encrypted_message_3x3, inverse_3x3)

# 输出结果
print("\n实验 ②: 加密和解密结果")
print("原始消息:", filtered_message)
print("\n使用 2x2 矩阵")
print("加密消息:", encrypted_message_2x2)
print("解密消息:", decrypted_message_2x2)

print("\n使用 3x3 矩阵")
print("加密消息:", encrypted_message_3x3)
print("解密消息:", decrypted_message_3x3)

```

{{% /fold-block %}}

### 置换密码

随机生成一个置换表（置换表长度可设置）

1. 求逆置换
2. 用此置换表加密消息（学号+姓名），得密文；解密密文，检验是否正确解密了？

{{% fold-block "代码 1-2_Symmetric_Permutation" %}}

```python
import random

# 定义可见 ASCII 字符的范围
MIN_ASCII = 32
MAX_ASCII = 126
CHARS = [chr(i) for i in range(MIN_ASCII, MAX_ASCII + 1)]
LENGTH = len(CHARS)

# 生成随机置换表
def generate_permutation_table(length):
    table = CHARS[:length]
    random.shuffle(table)  # 随机打乱字符列表，形成置换表
    return table

# 生成逆置换表
def generate_inverse_permutation_table(table):
    inverse_table = [''] * len(table)
    for i, char in enumerate(table):
        inverse_table[CHARS.index(char)] = CHARS[i]  # 找到字符在原字符集中的位置，并在逆置换表中记录相应的原字符
    return inverse_table

# 置换加密函数
def permutation_encrypt(message, table):
    # 用置换表替换消息中的每个字符
    return ''.join(table[CHARS.index(char)] if char in CHARS else char for char in message)

# 置换解密函数
def permutation_decrypt(encrypted_message, inverse_table):
    # 用逆置换表还原消息中的每个字符
    return ''.join(inverse_table[CHARS.index(char)] if char in CHARS else char for char in encrypted_message)

# 实验 ①: 打印生成的置换表及其逆置换表
table = generate_permutation_table(LENGTH)
inverse_table = generate_inverse_permutation_table(table)

print("实验 ①: 生成的置换表及其逆置换表")
print("置换表:", ''.join(table))
print("逆置换表:", ''.join(inverse_table))

# 实验 ②: 加密和解密消息
message = "26221013_Daaihang_Wong"
encrypted_message = permutation_encrypt(message, table)
decrypted_message = permutation_decrypt(encrypted_message, inverse_table)

# 输出结果
print("\n实验 ②: 加密和解密结果")
print("原始消息:", message)
print("加密消息:", encrypted_message)
print("解密消息:", decrypted_message)

```

{{% /fold-block %}}

### 对称加密算法AES和工作模式

1. 编写AES加密和解密算法
2. 分别结合CBC和OFB工作模式，加密消息（学号+姓名），得密文；解密密文，检验是否正确解密了？

**我给出了4种方式实现。优先使用第4个。其他的或多或少使用第三方的非基础密码库，可能不符合实验要求。**



{{% fold-block "代码 - 实现一 1-3_Symmetric_AES_1" %}}

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend
import os


# 生成随机密钥和初始化向量（IV）
def generate_key_iv(key_size):
    key = os.urandom(key_size // 8)
    iv = os.urandom(16)  # IV 大小应为块大小（16 字节）
    return key, iv


# AES 加密函数
def aes_encrypt(message, key, iv, mode):
    # PKCS7填充
    padder = padding.PKCS7(algorithms.AES.block_size).padder()
    padded_data = padder.update(message.encode()) + padder.finalize()

    # 创建AES加密器
    cipher = Cipher(algorithms.AES(key), mode(iv), backend=default_backend())
    encryptor = cipher.encryptor()

    # 加密数据
    ciphertext = encryptor.update(padded_data) + encryptor.finalize()
    return ciphertext


# AES 解密函数
def aes_decrypt(ciphertext, key, iv, mode):
    # 创建AES解密器
    cipher = Cipher(algorithms.AES(key), mode(iv), backend=default_backend())
    decryptor = cipher.decryptor()

    # 解密数据
    padded_data = decryptor.update(ciphertext) + decryptor.finalize()

    # 去除PKCS7填充
    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
    decrypted_data = unpadder.update(padded_data) + unpadder.finalize()
    return decrypted_data.decode()


# 主函数
def main():
    # 设置密钥长度（128, 192, 或 256 位）
    key_size = 256

    # 生成密钥和 IV
    key, iv = generate_key_iv(key_size)

    # 打印密钥和 IV
    print("实验 ①: 密钥和 IV")
    print("密钥:", key.hex())
    print("IV:", iv.hex())

    # 定义消息
    message = "26221013_Daaihang_Wong"

    # 实验 ②: CBC 模式加密和解密
    print("\n实验 ②: CBC 模式")
    ciphertext_cbc = aes_encrypt(message, key, iv, modes.CBC)
    print("CBC 加密消息:", ciphertext_cbc.hex())

    decrypted_message_cbc = aes_decrypt(ciphertext_cbc, key, iv, modes.CBC)
    print("CBC 解密消息:", decrypted_message_cbc)

    # 实验 ②: OFB 模式加密和解密
    print("\n实验 ②: OFB 模式")
    ciphertext_ofb = aes_encrypt(message, key, iv, modes.OFB)
    print("OFB 加密消息:", ciphertext_ofb.hex())

    decrypted_message_ofb = aes_decrypt(ciphertext_ofb, key, iv, modes.OFB)
    print("OFB 解密消息:", decrypted_message_ofb)


# 运行主函数
if __name__ == "__main__":
    main()

```

{{% /fold-block %}}



{{% fold-block "代码 - 实现二 1-3_Symmetric_AES_2" %}}

```python
import os
from pyaes import pyaes


def pad(data):
    padding_needed = 16 - len(data) % 16
    return data + chr(padding_needed) * padding_needed


def split_blocks(data, block_size=16):
    return [data[i:i + block_size] for i in range(0, len(data), block_size)]


def encrypt_cbc(key, iv, data):
    aes = pyaes.AESModeOfOperationCBC(key, iv=iv)
    padded_data = pad(data)
    blocks = split_blocks(padded_data)
    ciphertext = b''.join(aes.encrypt(block) for block in blocks)
    return ciphertext


def unpad(data):
    padding = ord(data[-1])
    return data[:-padding]


def decrypt_cbc(key, iv, ciphertext):
    aes = pyaes.AESModeOfOperationCBC(key, iv=iv)
    blocks = split_blocks(ciphertext)
    plaintext = ''.join(aes.decrypt(block).decode('utf-8') for block in blocks)
    return unpad(plaintext)


def encrypt_ofb(key, iv, data):
    aes = pyaes.AESModeOfOperationOFB(key, iv=iv)
    return aes.encrypt(data)


def decrypt_ofb(key, iv, ciphertext):
    aes = pyaes.AESModeOfOperationOFB(key, iv=iv)
    return aes.decrypt(ciphertext).decode()


def main():
    key = os.urandom(32)
    iv = os.urandom(16)
    print(repr(iv))

    # Using CBC mode
    plaintext_cbc = "26221013_Daaihang_Wong"
    ciphertext_cbc = encrypt_cbc(key, iv, plaintext_cbc)
    print(repr(ciphertext_cbc))

    decrypted_cbc = decrypt_cbc(key, iv, ciphertext_cbc)
    print(decrypted_cbc)
    print(decrypted_cbc == plaintext_cbc)

    # Using OFB mode
    plaintext_ofb = "26221013_Daaihang_Wong"
    ciphertext_ofb = encrypt_ofb(key, iv, plaintext_ofb)
    print(repr(ciphertext_ofb))

    decrypted_ofb = decrypt_ofb(key, iv, ciphertext_ofb)
    print(decrypted_ofb)
    print(decrypted_ofb == plaintext_ofb)


if __name__ == "__main__":
    main()

```

{{% /fold-block %}}



{{% fold-block "代码 - 实现三 1-3_Symmetric_AES_3" %}}

```python
# 一个简单的 AES-128-ECB，其中 128 指采用 128 位密钥块，
# ECB 模式，为长度不足 128 位的数据块填充 0x00。

s_box = [
    [0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5, 0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76],
    [0xca, 0x82, 0xc9, 0x7d, 0xfa, 0x59, 0x47, 0xf0, 0xad, 0xd4, 0xa2, 0xaf, 0x9c, 0xa4, 0x72, 0xc0],
    [0xb7, 0xfd, 0x93, 0x26, 0x36, 0x3f, 0xf7, 0xcc, 0x34, 0xa5, 0xe5, 0xf1, 0x71, 0xd8, 0x31, 0x15],
    [0x04, 0xc7, 0x23, 0xc3, 0x18, 0x96, 0x05, 0x9a, 0x07, 0x12, 0x80, 0xe2, 0xeb, 0x27, 0xb2, 0x75],
    [0x09, 0x83, 0x2c, 0x1a, 0x1b, 0x6e, 0x5a, 0xa0, 0x52, 0x3b, 0xd6, 0xb3, 0x29, 0xe3, 0x2f, 0x84],
    [0x53, 0xd1, 0x00, 0xed, 0x20, 0xfc, 0xb1, 0x5b, 0x6a, 0xcb, 0xbe, 0x39, 0x4a, 0x4c, 0x58, 0xcf],
    [0xd0, 0xef, 0xaa, 0xfb, 0x43, 0x4d, 0x33, 0x85, 0x45, 0xf9, 0x02, 0x7f, 0x50, 0x3c, 0x9f, 0xa8],
    [0x51, 0xa3, 0x40, 0x8f, 0x92, 0x9d, 0x38, 0xf5, 0xbc, 0xb6, 0xda, 0x21, 0x10, 0xff, 0xf3, 0xd2],
    [0xcd, 0x0c, 0x13, 0xec, 0x5f, 0x97, 0x44, 0x17, 0xc4, 0xa7, 0x7e, 0x3d, 0x64, 0x5d, 0x19, 0x73],
    [0x60, 0x81, 0x4f, 0xdc, 0x22, 0x2a, 0x90, 0x88, 0x46, 0xee, 0xb8, 0x14, 0xde, 0x5e, 0x0b, 0xdb],
    [0xe0, 0x32, 0x3a, 0x0a, 0x49, 0x06, 0x24, 0x5c, 0xc2, 0xd3, 0xac, 0x62, 0x91, 0x95, 0xe4, 0x79],
    [0xe7, 0xc8, 0x37, 0x6d, 0x8d, 0xd5, 0x4e, 0xa9, 0x6c, 0x56, 0xf4, 0xea, 0x65, 0x7a, 0xae, 0x08],
    [0xba, 0x78, 0x25, 0x2e, 0x1c, 0xa6, 0xb4, 0xc6, 0xe8, 0xdd, 0x74, 0x1f, 0x4b, 0xbd, 0x8b, 0x8a],
    [0x70, 0x3e, 0xb5, 0x66, 0x48, 0x03, 0xf6, 0x0e, 0x61, 0x35, 0x57, 0xb9, 0x86, 0xc1, 0x1d, 0x9e],
    [0xe1, 0xf8, 0x98, 0x11, 0x69, 0xd9, 0x8e, 0x94, 0x9b, 0x1e, 0x87, 0xe9, 0xce, 0x55, 0x28, 0xdf],
    [0x8c, 0xa1, 0x89, 0x0d, 0xbf, 0xe6, 0x42, 0x68, 0x41, 0x99, 0x2d, 0x0f, 0xb0, 0x54, 0xbb, 0x16]
]

s_box_inv = [
    [0x52, 0x09, 0x6a, 0xd5, 0x30, 0x36, 0xa5, 0x38, 0xbf, 0x40, 0xa3, 0x9e, 0x81, 0xf3, 0xd7, 0xfb],
    [0x7c, 0xe3, 0x39, 0x82, 0x9b, 0x2f, 0xff, 0x87, 0x34, 0x8e, 0x43, 0x44, 0xc4, 0xde, 0xe9, 0xcb],
    [0x54, 0x7b, 0x94, 0x32, 0xa6, 0xc2, 0x23, 0x3d, 0xee, 0x4c, 0x95, 0x0b, 0x42, 0xfa, 0xc3, 0x4e],
    [0x08, 0x2e, 0xa1, 0x66, 0x28, 0xd9, 0x24, 0xb2, 0x76, 0x5b, 0xa2, 0x49, 0x6d, 0x8b, 0xd1, 0x25],
    [0x72, 0xf8, 0xf6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xd4, 0xa4, 0x5c, 0xcc, 0x5d, 0x65, 0xb6, 0x92],
    [0x6c, 0x70, 0x48, 0x50, 0xfd, 0xed, 0xb9, 0xda, 0x5e, 0x15, 0x46, 0x57, 0xa7, 0x8d, 0x9d, 0x84],
    [0x90, 0xd8, 0xab, 0x00, 0x8c, 0xbc, 0xd3, 0x0a, 0xf7, 0xe4, 0x58, 0x05, 0xb8, 0xb3, 0x45, 0x06],
    [0xd0, 0x2c, 0x1e, 0x8f, 0xca, 0x3f, 0x0f, 0x02, 0xc1, 0xaf, 0xbd, 0x03, 0x01, 0x13, 0x8a, 0x6b],
    [0x3a, 0x91, 0x11, 0x41, 0x4f, 0x67, 0xdc, 0xea, 0x97, 0xf2, 0xcf, 0xce, 0xf0, 0xb4, 0xe6, 0x73],
    [0x96, 0xac, 0x74, 0x22, 0xe7, 0xad, 0x35, 0x85, 0xe2, 0xf9, 0x37, 0xe8, 0x1c, 0x75, 0xdf, 0x6e],
    [0x47, 0xf1, 0x1a, 0x71, 0x1d, 0x29, 0xc5, 0x89, 0x6f, 0xb7, 0x62, 0x0e, 0xaa, 0x18, 0xbe, 0x1b],
    [0xfc, 0x56, 0x3e, 0x4b, 0xc6, 0xd2, 0x79, 0x20, 0x9a, 0xdb, 0xc0, 0xfe, 0x78, 0xcd, 0x5a, 0xf4],
    [0x1f, 0xdd, 0xa8, 0x33, 0x88, 0x07, 0xc7, 0x31, 0xb1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xec, 0x5f],
    [0x60, 0x51, 0x7f, 0xa9, 0x19, 0xb5, 0x4a, 0x0d, 0x2d, 0xe5, 0x7a, 0x9f, 0x93, 0xc9, 0x9c, 0xef],
    [0xa0, 0xe0, 0x3b, 0x4d, 0xae, 0x2a, 0xf5, 0xb0, 0xc8, 0xeb, 0xbb, 0x3c, 0x83, 0x53, 0x99, 0x61],
    [0x17, 0x2b, 0x04, 0x7e, 0xba, 0x77, 0xd6, 0x26, 0xe1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0c, 0x7d]
]

rc = [0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1b, 0x36, 0x6c, 0xd8, 0xab, 0x4d]


def sub_bytes(grid, inv=False):
    for i, v in enumerate(grid):
        if inv:  # for decryption
            grid[i] = s_box_inv[v >> 4][v & 0xf]
        else:
            grid[i] = s_box[v >> 4][v & 0xf]


def shift_rows(grid, inv=False):
    for i in range(4):
        if inv:  # for decryption
            grid[i::4] = grid[i::4][-i:] + grid[i::4][:-i]
        else:
            grid[i::4] = grid[i::4][i:] + grid[i::4][:i]


def mix_columns(grid):
    def mul_by_2(n):
        s = (n << 1) & 0xff
        if n & 128:
            s ^= 0x1b
        return s

    def mul_by_3(n):
        return n ^ mul_by_2(n)

    def mix_column(c):
        return [
            mul_by_2(c[0]) ^ mul_by_3(c[1]) ^ c[2] ^ c[3],  # [2 3 1 1]
            c[0] ^ mul_by_2(c[1]) ^ mul_by_3(c[2]) ^ c[3],  # [1 2 3 1]
            c[0] ^ c[1] ^ mul_by_2(c[2]) ^ mul_by_3(c[3]),  # [1 1 2 3]
            mul_by_3(c[0]) ^ c[1] ^ c[2] ^ mul_by_2(c[3]),  # [3 1 1 2]
        ]

    for i in range(0, 16, 4):
        grid[i:i + 4] = mix_column(grid[i:i + 4])


def key_expansion(grid):
    for i in range(10 * 4):
        r = grid[-4:]
        if i % 4 == 0:  # 对上一轮最后4字节自循环、S-box置换、轮常数异或，从而计算出当前新一轮最前4字节
            for j, v in enumerate(r[1:] + r[:1]):
                r[j] = s_box[v >> 4][v & 0xf] ^ (rc[i // 4] if j == 0 else 0)

        for j in range(4):
            grid.append(grid[-16] ^ r[j])

    return grid


def add_round_key(grid, round_key):
    for i in range(16):
        grid[i] ^= round_key[i]


def encrypt(b, expanded_key):
    # First round
    add_round_key(b, expanded_key)

    for i in range(1, 10):
        sub_bytes(b)
        shift_rows(b)
        mix_columns(b)
        add_round_key(b, expanded_key[i * 16:])

    # Final round
    sub_bytes(b)
    shift_rows(b)
    add_round_key(b, expanded_key[-16:])
    return b


def decrypt(b, expanded_key):
    # First round
    add_round_key(b, expanded_key[-16:])

    for i in range(9, 0, -1):
        shift_rows(b, True)
        sub_bytes(b, True)
        add_round_key(b, expanded_key[i * 16:])
        for _ in range(3):
            mix_columns(b)

    # Final round
    shift_rows(b, True)
    sub_bytes(b, True)
    add_round_key(b, expanded_key)
    return b


def aes(typ, key, msg):
    expanded = key_expansion(bytearray(key))

    # Pad the message to a multiple of 16 bytes
    b = bytearray(msg)
    if typ == 0:  # only for encryption
        b = bytearray(msg + b'\x00' * (16 - len(msg) % 16))

    # Encrypt/decrypt the message
    for i in range(0, len(b), 16):
        if typ == 0:
            b[i:i + 16] = encrypt(b[i:i + 16], expanded)
        else:
            b[i:i + 16] = decrypt(b[i:i + 16], expanded)
    return bytes(b)


if __name__ == '__main__':
    key = b"Qw3rTyQw3rTyQw3r"
    enc = aes(0, key, b'Daaihang_Wong_26221013')
    dec = aes(1, key, enc)

    print('加密：', enc)
    print('解密：', dec)

```

{{% /fold-block %}}



{{% fold-block "代码 - 实现四 1-3_Symmetric_AES_4（推荐参考）" %}}

```python
import os

# AES块大小
BLOCK_SIZE = 16


# 生成随机字节
def generate_random_bytes(length):
    return os.urandom(length)


# 填充函数：PKCS#7
def pad(data):
    pad_length = BLOCK_SIZE - len(data) % BLOCK_SIZE
    return data + bytes([pad_length] * pad_length)


# 去填充函数：PKCS#7
def unpad(data):
    pad_length = data[-1]
    return data[:-pad_length]


# 定义S盒和逆S盒
s_box = [
    0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5, 0x30, 0x01, 0x67, 0x2b, 0xfe, 0xd7, 0xab, 0x76,
    0xca, 0x82, 0xc9, 0x7d, 0xfa, 0x59, 0x47, 0xf0, 0xad, 0xd4, 0xa2, 0xaf, 0x9c, 0xa4, 0x72, 0xc0,
    0xb7, 0xfd, 0x93, 0x26, 0x36, 0x3f, 0xf7, 0xcc, 0x34, 0xa5, 0xe5, 0xf1, 0x71, 0xd8, 0x31, 0x15,
    0x04, 0xc7, 0x23, 0xc3, 0x18, 0x96, 0x05, 0x9a, 0x07, 0x12, 0x80, 0xe2, 0xeb, 0x27, 0xb2, 0x75,
    0x09, 0x83, 0x2c, 0x1a, 0x1b, 0x6e, 0x5a, 0xa0, 0x52, 0x3b, 0xd6, 0xb3, 0x29, 0xe3, 0x2f, 0x84,
    0x53, 0xd1, 0x00, 0xed, 0x20, 0xfc, 0xb1, 0x5b, 0x6a, 0xcb, 0xbe, 0x39, 0x4a, 0x4c, 0x58, 0xcf,
    0xd0, 0xef, 0xaa, 0xfb, 0x43, 0x4d, 0x33, 0x85, 0x45, 0xf9, 0x02, 0x7f, 0x50, 0x3c, 0x9f, 0xa8,
    0x51, 0xa3, 0x40, 0x8f, 0x92, 0x9d, 0x38, 0xf5, 0xbc, 0xb6, 0xda, 0x21, 0x10, 0xff, 0xf3, 0xd2,
    0xcd, 0x0c, 0x13, 0xec, 0x5f, 0x97, 0x44, 0x17, 0xc4, 0xa7, 0x7e, 0x3d, 0x64, 0x5d, 0x19, 0x73,
    0x60, 0x81, 0x4f, 0xdc, 0x22, 0x2a, 0x90, 0x88, 0x46, 0xee, 0xb8, 0x14, 0xde, 0x5e, 0x0b, 0xdb,
    0xe0, 0x32, 0x3a, 0x0a, 0x49, 0x06, 0x24, 0x5c, 0xc2, 0xd3, 0xac, 0x62, 0x91, 0x95, 0xe4, 0x79,
    0xe7, 0xc8, 0x37, 0x6d, 0x8d, 0xd5, 0x4e, 0xa9, 0x6c, 0x56, 0xf4, 0xea, 0x65, 0x7a, 0xae, 0x08,
    0xba, 0x78, 0x25, 0x2e, 0x1c, 0xa6, 0xb4, 0xc6, 0xe8, 0xdd, 0x74, 0x1f, 0x4b, 0xbd, 0x8b, 0x8a,
    0x70, 0x3e, 0xb5, 0x66, 0x48, 0x03, 0xf6, 0x0e, 0x61, 0x35, 0x57, 0xb9, 0x86, 0xc1, 0x1d, 0x9e,
    0xe1, 0xf8, 0x98, 0x11, 0x69, 0xd9, 0x8e, 0x94, 0x9b, 0x1e, 0x87, 0xe9, 0xce, 0x55, 0x28, 0xdf,
    0x8c, 0xa1, 0x89, 0x0d, 0xbf, 0xe6, 0x42, 0x68, 0x41, 0x99, 0x2d, 0x0f, 0xb0, 0x54, 0xbb, 0x16,
]

inv_s_box = [
    0x52, 0x09, 0x6a, 0xd5, 0x30, 0x36, 0xa5, 0x38, 0xbf, 0x40, 0xa3, 0x9e, 0x81, 0xf3, 0xd7, 0xfb,
    0x7c, 0xe3, 0x39, 0x82, 0x9b, 0x2f, 0xff, 0x87, 0x34, 0x8e, 0x43, 0x44, 0xc4, 0xde, 0xe9, 0xcb,
    0x54, 0x7b, 0x94, 0x32, 0xa6, 0xc2, 0x23, 0x3d, 0xee, 0x4c, 0x95, 0x0b, 0x42, 0xfa, 0xc3, 0x4e,
    0x08, 0x2e, 0xa1, 0x66, 0x28, 0xd9, 0x24, 0xb2, 0x76, 0x5b, 0xa2, 0x49, 0x6d, 0x8b, 0xd1, 0x25,
    0x72, 0xf8, 0xf6, 0x64, 0x86, 0x68, 0x98, 0x16, 0xd4, 0xa4, 0x5c, 0xcc, 0x5d, 0x65, 0xb6, 0x92,
    0x6c, 0x70, 0x48, 0x50, 0xfd, 0xed, 0xb9, 0xda, 0x5e, 0x15, 0x46, 0x57, 0xa7, 0x8d, 0x9d, 0x84,
    0x90, 0xd8, 0xab, 0x00, 0x8c, 0xbc, 0xd3, 0x0a, 0xf7, 0xe4, 0x58, 0x05, 0xb8, 0xb3, 0x45, 0x06,
    0xd0, 0x2c, 0x1e, 0x8f, 0xca, 0x3f, 0x0f, 0x02, 0xc1, 0xaf, 0xbd, 0x03, 0x01, 0x13, 0x8a, 0x6b,
    0x3a, 0x91, 0x11, 0x41, 0x4f, 0x67, 0xdc, 0xea, 0x97, 0xf2, 0xcf, 0xce, 0xf0, 0xb4, 0xe6, 0x73,
    0x96, 0xac, 0x74, 0x22, 0xe7, 0xad, 0x35, 0x85, 0xe2, 0xf9, 0x37, 0xe8, 0x1c, 0x75, 0xdf, 0x6e,
    0x47, 0xf1, 0x1a, 0x71, 0x1d, 0x29, 0xc5, 0x89, 0x6f, 0xb7, 0x62, 0x0e, 0xaa, 0x18, 0xbe, 0x1b,
    0xfc, 0x56, 0x3e, 0x4b, 0xc6, 0xd2, 0x79, 0x20, 0x9a, 0xdb, 0xc0, 0xfe, 0x78, 0xcd, 0x5a, 0xf4,
    0x1f, 0xdd, 0xa8, 0x33, 0x88, 0x07, 0xc7, 0x31, 0xb1, 0x12, 0x10, 0x59, 0x27, 0x80, 0xec, 0x5f,
    0x60, 0x51, 0x7f, 0xa9, 0x19, 0xb5, 0x4a, 0x0d, 0x2d, 0xe5, 0x7a, 0x9f, 0x93, 0xc9, 0x9c, 0xef,
    0xa0, 0xe0, 0x3b, 0x4d, 0xae, 0x2a, 0xf5, 0xb0, 0xc8, 0xeb, 0xbb, 0x3c, 0x83, 0x53, 0x99, 0x61,
    0x17, 0x2b, 0x04, 0x7e, 0xba, 0x77, 0xd6, 0x26, 0xe1, 0x69, 0x14, 0x63, 0x55, 0x21, 0x0c, 0x7d,
]


# 字节替换
def sub_bytes(state):
    return bytes(s_box[b] for b in state)


# 字节替换的逆操作
def inv_sub_bytes(state):
    return bytes(inv_s_box[b] for b in state)


# 行移位
def shift_rows(state):
    return bytes([
        state[0], state[5], state[10], state[15],
        state[4], state[9], state[14], state[3],
        state[8], state[13], state[2], state[7],
        state[12], state[1], state[6], state[11],
    ])


# 行移位的逆操作
def inv_shift_rows(state):
    return bytes([
        state[0], state[13], state[10], state[7],
        state[4], state[1], state[14], state[11],
        state[8], state[5], state[2], state[15],
        state[12], state[9], state[6], state[3],
    ])


# GF(2^8)中的乘法
def gmul(a, b):
    p = 0
    for _ in range(8):
        if b & 1:
            p ^= a
        hi_bit_set = a & 0x80
        a <<= 1
        if hi_bit_set:
            a ^= 0x1b
        b >>= 1
    return p & 0xff


# 列混合
def mix_columns(state):
    mixed = bytearray(16)
    for i in range(4):
        mixed[i * 4 + 0] = gmul(state[i * 4 + 0], 2) ^ gmul(state[i * 4 + 1], 3) ^ state[i * 4 + 2] ^ state[i * 4 + 3]
        mixed[i * 4 + 1] = state[i * 4 + 0] ^ gmul(state[i * 4 + 1], 2) ^ gmul(state[i * 4 + 2], 3) ^ state[i * 4 + 3]
        mixed[i * 4 + 2] = state[i * 4 + 0] ^ state[i * 4 + 1] ^ gmul(state[i * 4 + 2], 2) ^ gmul(state[i * 4 + 3], 3)
        mixed[i * 4 + 3] = gmul(state[i * 4 + 0], 3) ^ state[i * 4 + 1] ^ state[i * 4 + 2] ^ gmul(state[i * 4 + 3], 2)
    return bytes(mixed)


# 列混合的逆操作
def inv_mix_columns(state):
    mixed = bytearray(16)
    for i in range(4):
        mixed[i * 4 + 0] = gmul(state[i * 4 + 0], 14) ^ gmul(state[i * 4 + 1], 11) ^ gmul(state[i * 4 + 2], 13) ^ gmul(
            state[i * 4 + 3], 9)
        mixed[i * 4 + 1] = gmul(state[i * 4 + 0], 9) ^ gmul(state[i * 4 + 1], 14) ^ gmul(state[i * 4 + 2], 11) ^ gmul(
            state[i * 4 + 3], 13)
        mixed[i * 4 + 2] = gmul(state[i * 4 + 0], 13) ^ gmul(state[i * 4 + 1], 9) ^ gmul(state[i * 4 + 2], 14) ^ gmul(
            state[i * 4 + 3], 11)
        mixed[i * 4 + 3] = gmul(state[i * 4 + 0], 11) ^ gmul(state[i * 4 + 1], 13) ^ gmul(state[i * 4 + 2], 9) ^ gmul(
            state[i * 4 + 3], 14)
    return bytes(mixed)


# 轮密钥加
def add_round_key(state, round_key):
    return bytes(a ^ b for a, b in zip(state, round_key))


# AES块加密函数
def aes_encrypt_block(block, key):
    state = add_round_key(block, key)
    for _ in range(9):
        state = sub_bytes(state)
        state = shift_rows(state)
        state = mix_columns(state)
        state = add_round_key(state, key)
    state = sub_bytes(state)
    state = shift_rows(state)
    state = add_round_key(state, key)
    return state


# AES块解密函数
def aes_decrypt_block(block, key):
    state = add_round_key(block, key)
    state = inv_shift_rows(state)
    state = inv_sub_bytes(state)
    for _ in range(9):
        state = add_round_key(state, key)
        state = inv_mix_columns(state)
        state = inv_shift_rows(state)
        state = inv_sub_bytes(state)
    state = add_round_key(state, key)
    return state


# CBC模式加密
def aes_cbc_encrypt(key, plaintext):
    iv = generate_random_bytes(BLOCK_SIZE)  # 生成随机初始化向量（IV）
    plaintext_padded = pad(plaintext.encode('utf-8'))
    ciphertext = iv
    previous_block = iv

    for i in range(0, len(plaintext_padded), BLOCK_SIZE):
        plaintext_block = plaintext_padded[i:i + BLOCK_SIZE]
        xor_block = bytes(a ^ b for a, b in zip(plaintext_block, previous_block))
        encrypted_block = aes_encrypt_block(xor_block, key)
        ciphertext += encrypted_block
        previous_block = encrypted_block

    return ciphertext


# CBC模式解密
def aes_cbc_decrypt(key, ciphertext):
    iv = ciphertext[:BLOCK_SIZE]
    actual_ciphertext = ciphertext[BLOCK_SIZE:]
    previous_block = iv
    plaintext_padded = b''

    for i in range(0, len(actual_ciphertext), BLOCK_SIZE):
        ciphertext_block = actual_ciphertext[i:i + BLOCK_SIZE]
        decrypted_block = aes_decrypt_block(ciphertext_block, key)
        plaintext_block = bytes(a ^ b for a, b in zip(decrypted_block, previous_block))
        plaintext_padded += plaintext_block
        previous_block = ciphertext_block

    plaintext = unpad(plaintext_padded)
    return plaintext.decode('utf-8')


# OFB模式加密
def aes_ofb_encrypt(key, plaintext):
    iv = generate_random_bytes(BLOCK_SIZE)  # 生成随机初始化向量（IV）
    plaintext_bytes = plaintext.encode('utf-8')
    ciphertext = iv
    previous_block = iv

    for i in range(0, len(plaintext_bytes), BLOCK_SIZE):
        block = plaintext_bytes[i:i + BLOCK_SIZE]
        encrypted_block = aes_encrypt_block(previous_block, key)
        cipher_block = bytes(a ^ b for a, b in zip(block, encrypted_block))
        ciphertext += cipher_block
        previous_block = encrypted_block

    return ciphertext


# OFB模式解密
def aes_ofb_decrypt(key, ciphertext):
    iv = ciphertext[:BLOCK_SIZE]
    actual_ciphertext = ciphertext[BLOCK_SIZE:]
    previous_block = iv
    plaintext_bytes = b''

    for i in range(0, len(actual_ciphertext), BLOCK_SIZE):
        ciphertext_block = actual_ciphertext[i:i + BLOCK_SIZE]
        encrypted_block = aes_encrypt_block(previous_block, key)
        plaintext_block = bytes(a ^ b for a, b in zip(ciphertext_block, encrypted_block))
        plaintext_bytes += plaintext_block
        previous_block = encrypted_block

    return plaintext_bytes.decode('utf-8')


# 生成AES密钥
def generate_aes_key():
    return generate_random_bytes(BLOCK_SIZE)


def main():
    message = "26221013_Daaihang_Wong"  # 学号+姓名
    print(f"原始消息: {message}")

    # 生成AES密钥
    aes_key = generate_aes_key()

    # CBC模式
    encrypted_message_cbc = aes_cbc_encrypt(aes_key, message)
    print(f"CBC模式加密后的消息: {encrypted_message_cbc}")
    decrypted_message_cbc = aes_cbc_decrypt(aes_key, encrypted_message_cbc)
    print(f"CBC模式解密后的消息: {decrypted_message_cbc}")

    # OFB模式
    encrypted_message_ofb = aes_ofb_encrypt(aes_key, message)
    print(f"OFB模式加密后的消息: {encrypted_message_ofb}")
    decrypted_message_ofb = aes_ofb_decrypt(aes_key, encrypted_message_ofb)
    print(f"OFB模式解密后的消息: {decrypted_message_ofb}")

    # 检查解密后的消息是否正确
    assert message == decrypted_message_cbc, "CBC模式解密失败！"
    assert message == decrypted_message_ofb, "OFB模式解密失败！"
    print("CBC和OFB模式解密成功！")


if __name__ == "__main__":
    main()

```

{{% /fold-block %}}


## 实验二 非对称加密

### 公钥加密算法RSA

1. 编写RSA密码算法（包含密钥生成、加密和解密）
2. 利用此算法加密消息（学号+姓名），得密文；解密密文，检验能否正确解密

{{% fold-block "代码 2-1_Asymmetric_RSA" %}}

```python
import random
from sympy import isprime, mod_inverse


class RSAEncryption:
    def __init__(self, key_size=2048):
        self.key_size = key_size
        self.public_key, self.private_key = self.generate_keys()

    def generate_prime(self, n_bits):
        """
        生成一个具有指定位数的素数
        """
        while True:
            num = random.getrandbits(n_bits)
            if isprime(num):
                return num

    def generate_keys(self):
        """
        生成RSA公钥和私钥
        """
        # 选择两个不同的大素数 p 和 q
        p = self.generate_prime(self.key_size // 2)
        q = self.generate_prime(self.key_size // 2)

        # 计算 n = p * q
        n = p * q

        # 计算 φ(n) = (p-1)(q-1)，其中φ(n)为n的欧拉函数
        phi = (p - 1) * (q - 1)

        # 选择一个整数 e，使得 1 < e < φ(n) 并且 e 与 φ(n) 互质
        e = 65537  # 常用的 e 值

        # 计算 d 使得 d * e ≡ 1 (mod φ(n))，即 d 是 e 模 φ(n) 的乘法逆元
        d = mod_inverse(e, phi)

        # 公钥: (e, n)
        # 私钥: (d, n)
        public_key = (e, n)
        private_key = (d, n)

        return public_key, private_key

    def encrypt(self, message):
        """
        使用公钥加密消息。将消息转换为整数 m，计算密文 c = m^e mod n。
        """
        e, n = self.public_key
        # 将消息转换为字节，再转换为整数
        message_bytes = message.encode('utf-8')
        message_int = int.from_bytes(message_bytes, byteorder='big')
        # 使用公钥加密消息
        encrypted_message_int = pow(message_int, e, n)
        return encrypted_message_int

    def decrypt(self, encrypted_message_int):
        """
        使用私钥解密消息。计算原文 m = c^d mod n，将整数 m 转换为字节，再转换为字符串。
        """
        d, n = self.private_key
        # 使用私钥解密消息
        decrypted_message_int = pow(encrypted_message_int, d, n)
        # 将整数转换为字节，再转换为字符串
        decrypted_message_bytes = decrypted_message_int.to_bytes((decrypted_message_int.bit_length() + 7) // 8,
                                                                 byteorder='big')
        return decrypted_message_bytes.decode('utf-8')


# 实例化RSAEncryption类
rsa_encryption = RSAEncryption()

# 定义要加密的消息
student_info = "26221013_Daaihang_Wong"

# 加密消息
encrypted_message = rsa_encryption.encrypt(student_info)
print(f"加密消息：{encrypted_message}")

# 解密消息
decrypted_message = rsa_encryption.decrypt(encrypted_message)
print(f"解密消息：{decrypted_message}")

# 验证解密是否正确
if student_info == decrypted_message:
    print("解密成功，消息正确")
else:
    print("解密失败，消息不正确")

```

{{% /fold-block %}}

### 公钥加密算法ElGamal

1. 编写ElGamal加密算法（包含密钥生成、加密和解密过程）
2. 利用此算法加密消息（学号+姓名），得密文；解密密文，检验能否正确解密

{{% fold-block "代码 2-2_Asymmetric_ElGamal" %}}

```python
from sympy import mod_inverse, isprime, randprime
from random import randint

class ElGamal:
    def __init__(self, key_size=2048):
        self.key_size = key_size
        # 生成一个大素数 p
        self.p = self.generate_large_prime()
        # 生成一个随机整数 g，满足 2 <= g <= p-1
        self.g = randint(2, self.p - 1)
        # 生成私钥 x，满足 1 <= x <= p-2
        self.x = randint(1, self.p - 2)
        # 计算公钥 h = g^x mod p
        self.h = pow(self.g, self.x, self.p)

    def generate_large_prime(self):
        """生成一个大素数"""
        while True:
            # 在指定范围内生成一个随机素数
            prime_candidate = randprime(2 ** (self.key_size - 1), 2 ** self.key_size)
            if isprime(prime_candidate):
                return prime_candidate

    def encrypt(self, message):
        """
        使用公钥加密消息
        步骤：
        1. 生成一个随机整数 y，满足 1 <= y <= p-2
        2. 计算 c1 = g^y mod p
        3. 计算 s = h^y mod p
        4. 将消息转换为整数 m
        5. 计算 c2 = (m * s) mod p
        返回密文对 (c1, c2)
        """
        y = randint(1, self.p - 2)
        c1 = pow(self.g, y, self.p)
        s = pow(self.h, y, self.p)
        m = int.from_bytes(message.encode('utf-8'), 'big')
        c2 = (m * s) % self.p
        return c1, c2

    def decrypt(self, ciphertext):
        """
        使用私钥解密消息
        步骤：
        1. 提取密文对 (c1, c2)
        2. 计算 s = c1^x mod p
        3. 计算 s 的乘法逆元 s_inv = s^-1 mod p
        4. 计算 m = (c2 * s_inv) mod p
        5. 将整数 m 转换为字节，再转换为字符串
        """
        c1, c2 = ciphertext
        s = pow(c1, self.x, self.p)
        s_inv = mod_inverse(s, self.p)
        m = (c2 * s_inv) % self.p
        message = m.to_bytes((m.bit_length() + 7) // 8, 'big').decode('utf-8')
        return message

# 实例化ElGamal类
elgamal = ElGamal()

# 定义要加密的消息
student_info = "26221013_Daaihang_Wong"

# 加密消息
encrypted_message = elgamal.encrypt(student_info)
print(f"加密消息：{encrypted_message}")

# 解密消息
decrypted_message = elgamal.decrypt(encrypted_message)
print(f"解密消息：{decrypted_message}")

# 验证解密是否正确
if student_info == decrypted_message:
    print("解密成功，消息正确")
else:
    print("解密失败，消息不正确")

```

{{% /fold-block %}}

### 混合加密（电子信封）

1. 基于已编写的AES和RSA（或ElGamal）密码算法，实现混合加密
2. 利用混合算法加密消息（学号+姓名），得密文；解密密文，检验能否正确解密

{{% fold-block "代码" %}}

```python
# todo: 还没实现，先等着吧，太久了就在评论区滴滴我。
```

{{% /fold-block %}}

## 实验三 认证技术

> 注：过程中可以利用密码库中的Hash算法（如SHA2，SM3，SHA3等）

### 消息认证码（MAC）

1. 编写基于Hash算法的消息认证码
2. 利用此算法对消息（学号+姓名）进行完整性和真实性认证

{{% fold-block "代码 3-1_Certification_MAC" %}}

```python
import hashlib


def xor_bytes(a, b):
    """对两个字节序列进行按位异或操作"""
    return bytes(x ^ y for x, y in zip(a, b))


def hmac_hash(key, message, hash_func=hashlib.sha256):
    """
    实现 HMAC 算法
    1. 如果密钥长度大于哈希块大小，则对密钥进行哈希处理
    2. 如果密钥长度小于哈希块大小，则在密钥末尾补零至块大小
    3. 生成内层和外层填充密钥
    4. 使用填充后的密钥和消息进行两次哈希计算
    """
    block_size = hash_func().block_size

    # 如果密钥长度大于块大小，则对密钥进行哈希处理
    if len(key) > block_size:
        key = hash_func(key).digest()

    # 如果密钥长度小于块大小，则在密钥末尾补零至块大小
    if len(key) < block_size:
        key = key.ljust(block_size, b'\0')

    # 生成内层和外层填充密钥
    o_key_pad = xor_bytes(key, b'\x5c' * block_size)
    i_key_pad = xor_bytes(key, b'\x36' * block_size)

    # 内层哈希
    inner_hash = hash_func(i_key_pad + message).digest()
    # 外层哈希
    final_hash = hash_func(o_key_pad + inner_hash).hexdigest()

    return final_hash


def generate_mac(key, message, hash_func=hashlib.sha256):
    """
    生成消息认证码（MAC）
    使用给定的密钥和消息生成 HMAC
    """
    message_bytes = message.encode('utf-8')
    return hmac_hash(key, message_bytes, hash_func)


def verify_mac(key, message, mac_to_verify, hash_func=hashlib.sha256):
    """
    验证消息认证码（MAC）
    比较生成的 MAC 和提供的 MAC 是否相同
    """
    generated_mac = generate_mac(key, message, hash_func)
    return compare_digest(generated_mac, mac_to_verify)


def compare_digest(a, b):
    """比较两个哈希值，防止时间攻击"""
    if len(a) != len(b):
        return False
    result = 0
    for x, y in zip(a, b):
        result |= ord(x) ^ ord(y)
    return result == 0


# 消息
message = "26221013_Daaihang_Wong"

# 使用的密钥
key = b'Qw3rTyQw3rTy'

# 生成消息认证码
mac = generate_mac(key, message)
print(f"消息: {message}")
print(f"生成的MAC: {mac}")

# 验证消息认证码
is_valid = verify_mac(key, message, mac)
print(f"MAC验证结果: {'有效' if is_valid else '无效'}")

# 模拟篡改消息后的MAC验证
tampered_message = "26221013_D@a1han9_W0ng"
tampered_mac = generate_mac(key, tampered_message) # 篡改后的消息的MAC
is_valid_tampered = verify_mac(key, tampered_message, mac)
print(f"篡改后的消息 与 原消息的MAC 验证的结果: {'有效' if is_valid_tampered else '无效'}")

```

{{% /fold-block %}}

### RSA数字签名算法

1. 编写RSA签名算法（包含密钥生成、签名和验证过程）
2. 利用此算法对消息（学号+姓名）进行签名；并在接收方进行验证

{{% fold-block "代码 3-2_Certification_RSA" %}}

```python
# RSA数字签名算法（利用密码库中的Hash算法）

from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization


def generate_rsa_key_pair():
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=2048
    )
    public_key = private_key.public_key()
    return private_key, public_key


def sign_message(private_key, message, hash_algorithm):
    signature = private_key.sign(
        message.encode(),
        padding.PSS(
            mgf=padding.MGF1(hash_algorithm),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hash_algorithm
    )
    return signature


def verify_signature(public_key, message, signature, hash_algorithm):
    try:
        public_key.verify(
            signature,
            message.encode(),
            padding.PSS(
                mgf=padding.MGF1(hash_algorithm),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hash_algorithm
        )
        return True
    except Exception as e:
        return False


# 生成密钥对
private_key, public_key = generate_rsa_key_pair()

# 消息
message = "26221013_Daaihang_Wong"

# 使用的哈希算法
hash_algorithm = hashes.SHA256()

# 生成签名
signature = sign_message(private_key, message, hash_algorithm)
print(f"消息: {message}")
print(f"签名: {signature.hex()}")

# 验证签名
is_valid = verify_signature(public_key, message, signature, hash_algorithm)
print(f"签名验证结果: {'有效' if is_valid else '无效'}")

# 模拟篡改消息后的签名验证
tampered_message = "26221013_D@a1han9_W0ng"
is_valid_tampered = verify_signature(public_key, tampered_message, signature, hash_algorithm)
print(f"篡改后消息的签名验证结果: {'有效' if is_valid_tampered else '无效'}")
```

{{% /fold-block %}}

### ElGamal数字签名算法

1. 编写ElGamal签名算法（包含密钥生成、签名和验证过程）
2. 利用此算法对消息（学号+姓名）进行签名；并在接收方进行验证

{{% fold-block "代码 3-3_Certification_ElGamal" %}}

```python
import hashlib
import random
from math import gcd
from sympy import mod_inverse, randprime, isprime


# 确保生成元是模p的一个原根
def find_primitive_root(p):
    """
    找到模 p 的一个原根。

    参数：
    p -- 模数，应为素数

    返回：
    g -- 模 p 的一个原根
    """
    if not isprime(p):
        raise ValueError("p must be a prime number")
    phi = p - 1
    prime_factors = set()
    n = phi
    i = 2
    while i * i <= n:
        if n % i == 0:
            prime_factors.add(i)
            while n % i == 0:
                n //= i
        i += 1
    if n > 1:
        prime_factors.add(n)

    for g in range(2, p):
        if all(pow(g, phi // factor, p) != 1 for factor in prime_factors):
            return g
    return None


# ElGamal签名算法类
class ElGamalSignature:
    def __init__(self, p=None, g=None, x=None, bits=256):
        """
        初始化 ElGamal 签名算法对象。

        参数：
        p -- 大素数，模数
        g -- 模 p 的原根
        x -- 私钥
        bits -- 素数 p 的位数，默认为 256

        如果未提供 p、g、x，则随机生成。
        """
        if p is None or g is None or x is None:
            self.p = randprime(2 ** (bits - 1), 2 ** bits)
            self.g = find_primitive_root(self.p)
            self.x = random.randint(1, self.p - 2)
        else:
            self.p = p
            self.g = g
            self.x = x
        self.y = pow(self.g, self.x, self.p)

    # 签名函数
    def sign(self, message):
        """
        对消息进行签名。

        参数：
        message -- 要签名的消息

        返回：
        (r, s) -- 签名结果
        """
        h = hashlib.sha256(message.encode()).digest()
        h_int = int.from_bytes(h, byteorder='big')
        while True:
            k = random.randint(1, self.p - 2)
            if gcd(k, self.p - 1) == 1:  # 确保 k 和 p-1 互质
                break
        r = pow(self.g, k, self.p)
        k_inv = mod_inverse(k, self.p - 1)
        s = (h_int - self.x * r) * k_inv % (self.p - 1)
        return (r, s)

    # 验证函数
    def verify(self, message, signature):
        """
        验证消息的签名是否有效。

        参数：
        message -- 原始消息
        signature -- 签名元组 (r, s)

        返回：
        valid -- 签名是否有效，True 或 False
        """
        r, s = signature
        if not (0 < r < self.p and 0 < s < self.p - 1):  # 检查 r 和 s 的范围
            return False
        h = hashlib.sha256(message.encode()).digest()
        h_int = int.from_bytes(h, byteorder='big')
        v1 = pow(self.y, r, self.p) * pow(r, s, self.p) % self.p
        v2 = pow(self.g, h_int, self.p)
        return v1 == v2


# 测试代码
if __name__ == '__main__':
    # 大素数和生成元
    p = 97380703184954711490740892696023412334547368592870793188226395165275008037663
    g = 2  # 使用find_primitive_root找出的原根，若已知则直接赋值
    x = 123456789  # 私钥

    # 初始化ElGamal签名对象
    elgamal = ElGamalSignature(p, g, x)

    # 消息
    student_info = "26221013_Daaihang_Wong"

    # 签名
    signature = elgamal.sign(student_info)
    print("签名：", signature)

    # 验证
    if elgamal.verify(student_info, signature):
        print("签名有效")
    else:
        print("签名无效")
```

{{% /fold-block %}}

## 实验四：密钥管理

###  DH密钥交换协议

1. DH密钥交换协议（包含密钥生成、加密和解密）
2. 双方利用此协议生成的密钥，结合已编写的AES算法来加密消息（学号+姓名），得密文；解密密文，检验能否正确解密

{{% fold-block "代码 4-1_Key_Management_DH" %}}

```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import x25519
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

# 函数：执行Diffie-Hellman密钥交换
def diffie_hellman():
    # 生成Alice的私钥和公钥
    alice_private_key = x25519.X25519PrivateKey.generate()
    alice_public_key = alice_private_key.public_key()

    # 生成Bob的私钥和公钥
    bob_private_key = x25519.X25519PrivateKey.generate()
    bob_public_key = bob_private_key.public_key()

    # Alice计算共享密钥
    alice_shared_key = alice_private_key.exchange(bob_public_key)

    # Bob计算共享密钥
    bob_shared_key = bob_private_key.exchange(alice_public_key)

    # 确保双方计算得到的共享密钥相同
    assert alice_shared_key == bob_shared_key

    return alice_shared_key

# 函数：使用AES加密消息
def encrypt_message(message, key):
    backend = default_backend()
    iv = os.urandom(16)  # 生成随机IV
    cipher = Cipher(algorithms.AES(key), modes.CFB(iv), backend=backend)
    encryptor = cipher.encryptor()
    ciphertext = encryptor.update(message) + encryptor.finalize()
    return (iv, ciphertext)

# 函数：使用AES解密消息
def decrypt_message(ciphertext, key, iv):
    backend = default_backend()
    cipher = Cipher(algorithms.AES(key), modes.CFB(iv), backend=backend)
    decryptor = cipher.decryptor()
    plaintext = decryptor.update(ciphertext) + decryptor.finalize()
    return plaintext

# 主函数：演示过程
def main():
    # 执行Diffie-Hellman密钥交换
    shared_key = diffie_hellman()
    print(f"共享密钥 (Hex): {shared_key.hex()}")

    # 使用HKDF从共享密钥派生AES密钥
    derived_key = HKDF(
        algorithm=hashes.SHA256(),
        length=32,  # 256位密钥长度
        salt=None,
        info=b'Qw3rTyQw3rTyQw3rTy',
        backend=default_backend()
    ).derive(shared_key)

    # 加密和解密消息
    message = b"26221013_Daaihang_Wong"
    print(f"原始消息: {message.decode()}")

    # 加密消息
    iv, ciphertext = encrypt_message(message, derived_key)
    print(f"加密后的消息 (Hex): {ciphertext.hex()}")

    # 解密消息
    decrypted_message = decrypt_message(ciphertext, derived_key, iv)
    print(f"解密后的消息: {decrypted_message.decode()}")

if __name__ == "__main__":
    main()
```

{{% /fold-block %}}

### Shamir秘密共享方案

1. 编写一个(n, t) Shamir秘密共享方案（包含秘密的分片和基于Lagrange插值方法恢复秘密）
2. 实例化(n, t) =(5, 3)，而秘密信息为你的“学号+姓名”，演示秘密分片和恢复秘密的过程。

{{% fold-block "代码 4-2_Key_Management_Shamir" %}}

```python
import random
from sympy import nextprime


def generate_shares(secret, n, t, prime):
    """
    使用Shamir的秘密共享方案生成n个分享，阈值为t。

    参数：
    - secret: 要共享的秘密（以数值形式）。
    - n: 要生成的分享总数。
    - t: 恢复秘密所需的分享阈值。
    - prime: 有限域素数，用于计算。

    返回：
    - 一个包含分享的元组列表 (x, y)。
    """
    coefficients = [secret] + [random.randrange(prime) for _ in range(t - 1)]

    def polynomial(x):
        return sum([coeff * pow(x, i, prime) % prime for i, coeff in enumerate(coefficients)]) % prime

    shares = []
    for i in range(1, n + 1):
        x = i
        y = polynomial(x)
        shares.append((x, y))

    return shares


def recover_secret(shares, prime):
    """
    使用Shamir的秘密共享方案恢复秘密，给定足够数量的分享。

    参数：
    - shares: 一个分享的元组列表 (x, y)。
    - prime: 有限域素数，用于计算。

    返回：
    - 恢复的秘密（以数值形式）。
    """

    def lagrange_interpolation(x, shares):
        total = 0
        n = len(shares)
        for i in range(n):
            xi, yi = shares[i]
            term = yi
            for j in range(n):
                if i != j:
                    xj, _ = shares[j]
                    term = term * (x - xj) * pow(xi - xj, -1, prime) % prime
            total = (total + term) % prime
        return total

    return lagrange_interpolation(0, shares)


# 示例：使用学号和姓名作为秘密
secret = "26221013_Daaihang_Wong"
secret_numeric = int.from_bytes(secret.encode(), "big")  # 转换为数值形式

n = 5  # 总分享数
t = 3  # 恢复秘密所需的分享阈值

# 选择一个足够大的素数作为有限域的模数
prime = nextprime(secret_numeric)

# 生成分享
shares = generate_shares(secret_numeric, n, t, prime)

print("分享：")
for share in shares:
    print(share)

# 选择任意 t 个分享来恢复秘密
selected_shares = random.sample(shares, t)

# 恢复秘密
recovered_numeric = recover_secret(selected_shares, prime)

# 将数值形式的秘密转换为字节数据
recovered_secret_bytes = recovered_numeric.to_bytes((recovered_numeric.bit_length() + 7) // 8, "big")

# 比较恢复的秘密与原始秘密
if recovered_secret_bytes == secret.encode():
    print("\n成功恢复秘密：", secret)
else:
    print("\n恢复的秘密与原始秘密不匹配")
```

{{% /fold-block %}}
