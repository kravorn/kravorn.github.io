---
title: 【探索】通过AES加密HF space中的敏感文件
layout: post
index_img: /img/4-1.jpg
date: 2024-11-24 23:55:12
tags:
  - HuggingFace
  - AES
categories:
  - 探索
---

经常需要通过demo来展示一些小模型，HuggingFace的Space是用的最顺手的，但是有时候一些特殊的文件不方便完全开源在HF中。和Limour交流之后，他提出了一种借助Space中隐私环境变量的方法：使用AES加密文件，使用时再解密。

![](/img/4-1.jpg)

## AES加密

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import os
import base64

key = base64.b64decode("your_key")

def encrypt_file(input_file, output_file, key):
    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    with open(input_file, 'rb') as f:
        plaintext = f.read()
    ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
    with open(output_file, 'wb') as f:
        f.write(iv + ciphertext)

def encrypt_folder(folder_path, output_folder, key):
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    for root, _, files in os.walk(folder_path):
        for file in files:
            input_file = os.path.join(root, file)
            rel_path = os.path.relpath(input_file, folder_path)
            output_file = os.path.join(output_folder, rel_path + ".enc")
            os.makedirs(os.path.dirname(output_file), exist_ok=True)
            encrypt_file(input_file, output_file, key)
            print(f"Encrypted {input_file} to {output_file}")

#示例
if __name__ == "__main__":
    #文件
    encrypt_file("model", "model.enc", key)

    #文件夹
    encrypt_folder("models", "encrypted_models", key)

```

## 解密
使用一个基于Streamlit的web app作为演示。

- 首先设置你的**Secrets**：

![](/img/4-2.jpg)

- 然后在加载模式前先解密文件\文件夹：

```python
import os
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import streamlit as st
import pandas as pd
import pickle

your_key = os.getenv('env_your_key') # 你的环境变量

key = base64.b64decode(your_key)  # 解码密钥


# 解密单个文件并返回解密后的数据
def decrypt_file(input_file, key):
    with open(input_file, 'rb') as f:
        file_data = f.read()
    iv = file_data[:16]
    ciphertext = file_data[16:]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)
    return plaintext


# 解密加密文件并保存到指定路径
def decrypt_file_and_save(encrypted_file, decrypted_file, key):
    decrypted_data = decrypt_file(encrypted_file, key)

    with open(decrypted_file, 'wb') as f:
        f.write(decrypted_data)
    print(f"Decrypted {encrypted_file} to {decrypted_file}")


# 解密整个文件夹并解密其中所有文件
def decrypt_folder_and_save(encrypted_folder, decrypted_folder, key):
    if not os.path.exists(decrypted_folder):
        os.makedirs(decrypted_folder)

    for root, _, files in os.walk(encrypted_folder):
        for file in files:
            if file.endswith('.enc'):
                encrypted_file = os.path.join(root, file)

                relative_path = os.path.relpath(encrypted_file, encrypted_folder)
                decrypted_file = os.path.join(decrypted_folder, relative_path[:-4])

                os.makedirs(os.path.dirname(decrypted_file), exist_ok=True)

                try:
                    decrypt_file_and_save(encrypted_file, decrypted_file, key)
                    print(f"Successfully decrypted: {encrypted_file}")
                except Exception as e:
                    print(f"Error decrypting {encrypted_file}: {str(e)}")


# 解密文件
encrypted_model_file = 'model.enc'
decrypted_model_file = 'model'

if os.path.exists(encrypted_model_file):
    decrypt_file_and_save(encrypted_model_file, decrypted_model_file, key)

# 解密整个文件夹
encrypted_folder = 'encrypted_models'
decrypted_folder = 'models'

if os.path.exists(encrypted_folder):
    decrypt_folder_and_save(encrypted_folder, decrypted_folder, key)

# 加载解密后的模型文件
with open(decrypted_model_file, 'rb') as f:
    model = pickle.load(f)
predictor = model

# st.title('Following is your model')
```

## 总结
这个方法可以让未加密的文件不暴露在HF的spcae中，但是在本地运行时，解密后的文件/文件夹还是会显示在目录中。可惜本人能力有限，不清楚是否有其他手段可以获取到解密后的文件。