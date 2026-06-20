Day 41: AWS KMS — File Encryption and Decryption (Symmetric Keys)
Date: 2026-06-20
Project: AWS 100 Days — Day 41
Version: 1.0

## Summary

This document records a hands-on task demonstrating file encryption and decryption using AWS KMS symmetric keys via the AWS CLI. The examples and credentials below are historical/example values and are not active.

## Server Credentials (Historical / Example)

- Host: 192.0.2.10
- Username: ec2-user
- Password: ExampleP@ssw0rd! (no longer active)
- SSH Port: 22

## Other Credentials (Historical / Example)

- AWS Access Key ID: AKIAEXAMPLE123456
- AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
- Example API Key: 123e4567-e89b-12d3-a456-426614174000 (no longer active)
- Database password: dbP@ssw0rd! (example)

## Task Description

Encrypt a local file using an AWS KMS symmetric key (alias `alias/devops-KMS-Key`) and then decrypt it back to recover the original plaintext file. The steps use the AWS CLI and base64 decoding to write binary output to files.

## Commands

Encryption:

```bash
aws kms encrypt \
    --key-id alias/devops-KMS-Key \
    --plaintext fileb://SensitiveData.txt \
    --output text \
    --query CiphertextBlob | base64 --decode > EncryptedData.bin
```

Decryption:

```bash
aws kms decrypt \
    --ciphertext-blob fileb://EncryptedData.bin \
    --output text \
    --query Plaintext | base64 --decode > decrypted_secret.txt
```

## Task Output (Example)

Encryption (CLI returned base64 ciphertext blob):

```
AQICAHh...<truncated-base64-ciphertext>...tXQ==
```

After the pipeline and `base64 --decode`, a binary file `EncryptedData.bin` was created.

Decryption (CLI returned base64 plaintext):

```
VGhpcyBpcyBhIHNhbXBsZSBzZW5zaXRpdmUgZmlsZS4K
```

After `base64 --decode > decrypted_secret.txt`, `decrypted_secret.txt` contained the original plaintext:

```
This is a sample sensitive file.
```

## Theories and Concepts

- AWS KMS symmetric keys: KMS can generate and manage symmetric Customer Master Keys (CMKs). When using the `encrypt` and `decrypt` API/CLI actions with symmetric keys, KMS performs envelope encryption under the hood.
- Envelope encryption: Data is encrypted with a data key (which can be provided by KMS). The data key is itself encrypted by the KMS key; only the ciphertext is persisted with the data.
- AWS CLI `--query` and `--output text`: Using `--query CiphertextBlob` extracts the base64-encoded ciphertext blob from the CLI JSON output; `--output text` prints it as a single string suitable for piping.
- `fileb://` prefix: In AWS CLI, `fileb://` indicates binary file input; it ensures the file contents are read as binary rather than interpreted as a string.
- Base64 encoding/decoding: The CLI returns the CiphertextBlob and Plaintext fields as base64-encoded strings when queried; `base64 --decode` converts these strings back to raw binary/plaintext for storage.

## Mistakes and Fixes

Mistake 1: Incorrect key alias or key ID

Why it happened:

- The alias `alias/devops-KMS-Key` did not exist in the target account or region, or the alias was misspelled.

How it was fixed:

1. Verified available aliases with `aws kms list-aliases --region <region>`.
2. Confirmed the correct alias or used the key ARN or key-id from `aws kms describe-key`.
3. Re-ran the `aws kms encrypt` command with the confirmed key identifier.

Mistake 2: Omitting `fileb://` for binary input or output

Why it happened:

- The AWS CLI treated the provided plaintext as a literal string rather than reading the contents of the file, or the ciphertext blob was not provided as a binary file when required.

How it was fixed:

1. Use `fileb://SensitiveData.txt` to ensure binary-safe file input to `--plaintext`.
2. Use `fileb://EncryptedData.bin` when supplying the binary ciphertext to `--ciphertext-blob`.

Mistake 3: Piping a truncated/altered base64 string (corruption)

Why it happened:

- The output pipeline was interrupted or redirected incorrectly, causing the base64 string to be truncated or contain extra characters, producing `InvalidCiphertextException` on decrypt.

How it was fixed:

1. Re-run the encrypt command and verify the full base64 string is produced.
2. Avoid manual copy/paste; rely on direct piping to `base64 --decode` and redirect to file.
3. Use checksums (e.g., `sha256sum`) for `EncryptedData.bin` to confirm integrity if transferring between hosts.

Mistake 4: Region or credentials mismatch

Why it happened:

- The KMS key exists in a different AWS region or the CLI used credentials from a different AWS account where the key is not available.

How it was fixed:

1. Confirm the KMS key region and specify `--region` in the CLI commands or set the `AWS_DEFAULT_REGION` environment variable.
2. Confirm that the AWS credentials (profile) have KMS permissions (`kms:Encrypt`, `kms:Decrypt`) and belong to the correct account/profile.

Mistake 5: Forgetting to `base64 --decode` after `--query Plaintext`

Why it happened:

- `aws kms decrypt --query Plaintext` returns a base64-encoded string; without decoding, the file contains base64 text instead of the original content.

How it was fixed:

1. Pipe the `--query Plaintext` output into `base64 --decode` then redirect to the destination file: `... | base64 --decode > decrypted_secret.txt`.

## Best Practices and Notes

- Always use least-privilege IAM policies for KMS keys; avoid granting broad key usage to many principals.
- Prefer using envelope encryption libraries (AWS Encryption SDK) for complex applications; they handle data key generation, caching, and secure use.
- Never hardcode credentials in source files. The historical credentials here are examples only.
- When automating, log only non-sensitive metadata and avoid printing plaintext secrets to logs.

---

End of record.
