# Sigstore Integration with prefix.dev

A complete example demonstrating how to build a Conda package with cryptographic signatures using Sigstore attestations and trusted publishing to prefix.dev.

## Overview

This repository showcases the integration of:

- **`rattler-build`** - Building Conda packages
- **Sigstore** - Creating cryptographic attestations (CEP-27 compliance)
- **prefix.dev** - Trusted publishing without API keys
- **GitHub Actions** - Automated CI/CD pipeline

## How It Works

### 1. Package Building

The workflow uses the `prefix-dev/rattler-build-action@v0.2.34` to:

- Install `rattler-build`
- Build the Conda package from the recipe in `conda.recipe/`

### 2. Attestation Creation

We create a cryptographic attestation using GitHub's official attest action:

```yaml
- uses: actions/attest@v1
  id: attest
  with:
    subject-path: "**/*.conda"
    predicate-type: "https://schemas.conda.org/attestations-publish-1.schema.json"
    predicate: "{\"targetChannel\": \"https://prefix.dev/sigstore-example\"}"
```

This creates an attestation on the public Sigstore instance with CEP-27 compliance.

> **Note:** For private repositories, you'll need to enable Sigstore in your repository settings.

### 3. Trusted Publishing Setup

On prefix.dev, we've configured trusted publishing to allow uploads from:

- **Repository:** `prefix-dev/sigstore-example`
- **Workflow:** `.github/workflows/action.yaml`

![Trusted publishing configuration on prefix.dev](https://github.com/user-attachments/assets/8119f636-377a-48c9-8354-40226d2ea6b5)

### 4. Secure Upload

The package and attestation are uploaded securely without API keys:

```yaml
- name: Upload the package
  run: |
    rattler-build upload prefix -c sigstore-example ./output/**/*.conda --attestation ${{ steps.attest.outputs.bundle-path }}
```

## 🔍 Verification & Results

### Package Location

- **Package:** [prefix.dev/channels/sigstore-example/packages/signed-package](https://prefix.dev/channels/sigstore-example/packages/signed-package)

### Attestation Locations

The signature can be verified on multiple platforms:

- **prefix.dev** - Package metadata
- **[GitHub Attestations](https://github.com/prefix-dev/sigstore-example/attestations/10209596)** - Repository attestations
- **[Sigstore Public Instance](https://search.sigstore.dev/?logIndex=456061810)** - Public transparency log

## Verifying Attestations

### Prerequisites

Install required tools (if not already available):

```sh
pixi global install gh curl
```

### Download and Verify

```sh
# Download the package
curl -L https://prefix.dev/sigstore-example/linux-64/signed-package-2.1.0-hb0f4dca_0.conda -o package.conda

# Verify the attestation
gh attestation verify \
  --owner prefix-dev \
  --predicate-type "https://schemas.conda.org/attestations-publish-1.schema.json" \
  package.conda
```

### Expected Output

```text
Loaded digest sha256:3862a3677d33a45134a2ce3452b23f8f7459fe581cefbc3818272648cd987cfb for file://package.conda
Loaded 1 attestation from GitHub API

The following policy criteria will be enforced:
- Predicate type must match:................ https://schemas.conda.org/attestations-publish-1.schema.json
- Source Repository Owner URI must match:... https://github.com/prefix-dev
- Subject Alternative Name must match regex: (?i)^https://github.com/prefix-dev/
- OIDC Issuer must match:................... https://token.actions.githubusercontent.com

✓ Verification succeeded!

The following 1 attestation matched the policy criteria

- Attestation #1
- Build repo:..... prefix-dev/sigstore-example
- Build workflow:. .github/workflows/action.yaml@refs/heads/main
- Signer repo:.... prefix-dev/sigstore-example
- Signer workflow: .github/workflows/action.yaml@refs/heads/main
```

## Alternative Verification Methods

You can also verify attestations using:

- **`cosign`** - Sigstore's native CLI tool
- **`sigstore-python`** - Python SDK for Sigstore verification
