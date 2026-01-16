The Nautilus DevOps team is focusing on improving their data security by using AWS KMS. Your task is to create a KMS key and manage the encryption and decryption of a pre-existing sensitive file using the KMS key.

Specific Requirements:

Create a symmetric KMS key named datacenter-KMS-Key to manage encryption and decryption.
Encrypt the provided SensitiveData.txt file (located in /root/), base64 encode the ciphertext, and save the encrypted version as EncryptedData.bin in the /root/ directory.
Try to decrypt the same and verify that the decrypted data matches the original file.
Make sure that the KMS key is correctly configured. The validation script will test your configuration by decrypting the EncryptedData.bin file using the KMS key you created.

### Solution
1. Create a key as specified on the Customer Managed keys at the AWS KMS

2. Encrypt the file as:
```bash 
aws kms encrypt \
    --key-id alias/datacenter-KMS-Key \
    --plaintext fileb:///root/SensitiveData.txt\                                              
    --output text \
    --query CiphertextBlob | base64 \
    --decode > /root/EncryptedData.bin
```

3. Decrypt the encrypted file as:
```bash
aws kms decrypt \
    --ciphertext-blob fileb:///root/EncryptedData.bin \
    --output text \
    --query Plaintext | base64 \
    --decode > /root/Decrypted.txt
```

4. Verify by:  
` diff /root/Decrypted.txt /root/SensitiveData.txt `  
You will be outputed nothing if everything is correct.