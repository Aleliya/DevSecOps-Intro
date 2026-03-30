# Lab 8

## Task 1

### How signing protects against tag tampering

In Docker and container registries, tags are just convenient names that point to a specific image. But the tag itself does not guarantee the immutability of the image. Anyone can rewrite the tag by pointing to another image, such as busybox, as in the tamper demo.

#### Signing with **cosign** solves this problem as follows::
1. **Cosign** creates a cryptographic signature using a private key for a specific image.

2. The signature is not linked to the tag, but to the digest of the image. Digest is a SHA256 hash of the image contents, which is unique for each version of the image.

3. During verification, Cosign uses the public key and compares the signature with the digest image.:
    - If the content has changed (for example, the tag now points to a different image), digest will change.
    - The signature will no longer match the new digest → verification will fail.

**Conclusion**: even if someone changed the tag to another image, the signature will not allow such an image to be accepted as "authentic". This protects the supply chain and prevents image substitution.

### What “subject digest” means

**Subject digest** - a hash (SHA256) of a specific image that has been signed. It uniquely identifies the contents of the image, regardless of the tags. In the **cosign** report, the subject field shows the digest that confirms the signature.

---

## Task 2

### How attestations differ from signatures

1. Attestation:
    - **Purpose**: Attaches additional metadata about the image, such as SBOM or provenance.
    - **What is signed**: Hash of the predicate file (an additional JSON document) linked to the image.
    - **Content**: A full-fledged JSON “envelope” with the fields predicateType, predicate, subject and a digital signature.
    - **Why is it necessary**: It proves how and from what this image was created — it enhances the transparency of the supply chain.

2. Signature:
    - **Purpose**: AConfirms the authenticity of the image itself (that the content has not been replaced).
    - **What is signed**: Hash (digest) of the container image.
    - **Content**: Cryptographic signature only.
    - **Why is it necessary**: Proves that the image has not been changed.

### What information the SBOM attestation contains

**The SBOM attestation contains**:

- List of all packages, libraries and dependencies in the image;
- Metadata about their sources, licenses, and hashes;
- Information about the analysis environment.

### What information the SBOM attestation contains

Why is provenance-attestation needed:

- Records who collected the image, when, and how;
- Documents which parameters and which dependencies were involved in the build;
- Allows you to track the source of the artifact (supply-chain transparency);
- Checking whether the image was created in a trusted environment and by someone you trust.

---

## Task 3

### Use cases for signing non-container artifacts (e.g., release binaries, configuration files)

The use of signing non-container artifacts:

- **Release binaries and installation packages** — so that users can verify that the downloaded file has actually been released by the developer.
- **Configuration files and manifests** - used to prevent parameter changes before deployment.
- **Archives with source code or dependencies** — to ensure the integrity of the transfer.
- **Documentation, reports, and security policies** distributed digitally.

### How blob signing differs from container image signing

1. Signing a blob (blob/tarball):
    - **Purpose**: Verify the authenticity and integrity of any file outside the container.
    - **What is signed**: The file itself (for example, tar.gz , binary, etc.).
    - **Where is the signature stored**: In a separate bundle file next to the artifact.
    - **Checking**: It is executed locally using the file and its signature.

2. Signing the container:
    - **Purpose**: Verify the container has not been modified after publication.
    - **What is signed**: Hash of the image (digest) in the registry.
    - **Where is the signature stored**: In the OCI registry (along with the image)
    - **Checking**: Executed at the registry address and digest.