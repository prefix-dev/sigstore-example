# prefix.dev Sigstore integration example

This example uses `rattler-build` to build a Conda package, creates a CEP-27 attestation using `github/attest` and publishes it to prefix.dev (using trusted publishing).

Let's walk through the steps:

1. The action uses the `prefix-dev/rattler-build-action@v0.2.34` action to install rattler-build and build the recipe found in `conda.recipe`.
2. We then run
   ```yaml
      # We are using Github's official attest action to create a sigstore certificate with
      # the Conda predicate.
      - uses: actions/attest@v1
        id: attest
        with:
          subject-path: "**/*.conda"
          predicate-type: "https://schemas.conda.org/attestations-publish-1.schema.json"
          predicate: "{\"targetChannel\": \"https://prefix.dev/sigstore-example\"}"
   ```
   to create the attestation on the public-good sigstore instance. Note: if you have private repositories, you'll have to enable sigstore in your settings.
3. On prefix.dev we have configured trusted publishing to allow uploads from: prefix-dev / sigstore-example / action.yaml.
4. The package and attestation are uploaded (with trusted publishing, no API key is needed!) to prefix.dev:
   ```yaml
      # Note: no API keys needed because we have configured a trusted publisher!
      - name: Upload the package
        run: |
          rattler-build upload prefix -c sigstore-example ./output/**/*.conda --attestation ${{ steps.attest.outputs.bundle-path }}
   ```
5. The package can be found on prefix.dev: https://prefix.dev/channels/sigstore-example/packages/signed-package
6. The signature can be found on prefix.dev, on [Github](https://github.com/prefix-dev/sigstore-example/attestations/10209596) and on the [sigstore public good instance](https://search.sigstore.dev/?logIndex=456061810).
7. The attestation can be verified using 
   ```sh
   # In case you don't have `gh` and `curl` installed yet, you can globally install it easily with pixi:
   $ pixi global install gh curl
   # Download the package with curl
   $ curl -L https://prefix.dev/sigstore-example/linux-64/signed-package-2.1.0-hb0f4dca_0.conda -o package.conda
   # Run `attestation verify`
   $ gh attestation verify --owner prefix-dev --predicate-type "https://schemas.conda.org/attestations-publish-1.schema.json" package.conda

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
8. There are also a number of other ways to verify the attestion, for example using `cosign` or `sigstore-python`.