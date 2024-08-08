### Simple Encryption and Decryption Process Using Terminal

This guide explains how to securely transfer a directory named `data-112` from a PC in Germany (PC_GERMANY) to a PC in Finland (PC_FINLAND) by encrypting it on PC_GERMANY and decrypting it on PC_FINLAND.

#### Key Generation on PC_FINLAND

1. **Generate Private Key:**
   ```bash
   openssl genpkey -algorithm RSA -out private_key.pem
   ```

2. **Generate Public Key:**
   ```bash
   openssl rsa -pubout -in private_key.pem -out public_key.pem
   ```

   Share the generated `public_key.pem` with PC_GERMANY.

#### Encryption Process on PC_GERMANY

Before starting, ensure the following tasks are executed within a Screen Session due to the large size of files to avoid interruptions caused by connection issues.

1. **Create Screen Session:**
   ```bash
   screen -S session_name
   ```

2. **Initiate Interactive Job in the Screen Session:**
   ```bash
   qsub -I -A ngsukdkohi -l select=1:ncpus=1:mem=20GB -l walltime=10:00:00
   ```

3. **Archive the Directory:**
   ```bash
   tar -cvzf data-112.tar.gz data-112
   ```

4. **Generate a Random Symmetric Key and Encrypt the Tarball:**
   ```bash
   openssl rand 32 > symmetric_key.bin
   openssl enc -aes-256-cbc -pbkdf2 -in data-112.tar.gz -out encrypted_data-112.tar.gz -pass file:symmetric_key.bin
   ```

5. **Encrypt the Symmetric Key with the Recipient's Public Key:**
   ```bash
   openssl pkeyutl -encrypt -pubin -inkey public_key.pem -in symmetric_key.bin -out encrypted_symmetric_key.bin
   ```

Now, `encrypted_data-112.tar.gz` and `encrypted_symmetric_key.bin` are ready to be transferred to PC_FINLAND.

#### Decryption Process on PC_FINLAND

1. **Decrypt the Symmetric Key with the Private Key:**
   ```bash
   openssl pkeyutl -decrypt -inkey private_key.pem -in encrypted_symmetric_key.bin -out decrypted_symmetric_key.bin
   ```

2. **Decrypt the Tarball with the Decrypted Symmetric Key:**
   ```bash
   openssl enc -d -aes-256-cbc -pbkdf2 -in encrypted_data-112.tar.gz -out decrypted_data-112.tar.gz -pass file:decrypted_symmetric_key.bin
   ```

3. **Extract the Contents of the Decrypted Tarball:**
   ```bash
   tar -xvzf decrypted_data-112.tar.gz
   ```

This is how the encrypted files are decrypted successfully.

### Important Notes

- **Handle keys securely:** Ensure private keys are kept confidential and public keys are shared securely.
- **Adapt the process:** Modify the steps as needed to suit your specific security requirements and environment.
- **Ensure secure transfer channels:** Use secure methods (like SCP or SFTP) to transfer files between PCs.

By following these steps, you can securely transfer and handle sensitive data across different locations.
