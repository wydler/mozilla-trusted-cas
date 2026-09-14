# Mozilla Trusted CAs

This repository provides a CA certificate bundle based on the Mozilla Root Certificate Store, with support for additional locally trusted CA certificates.

The resulting `cacert.pem` can be used by applications that require a PEM-formatted CA certificate bundle.

## Overview

The CA bundle consists of:

1. CA certificates from the Mozilla NSS Root Certificate Store
2. Additional custom CA certificates from `custom-cas/`

The Mozilla CA bundle is generated using curl's `mk-ca-bundle.pl` script.

Custom CA certificates are validated, checked for duplicates and appended to the generated Mozilla bundle.

## Files

### `cacert.pem`

The final CA certificate bundle.

It contains the Mozilla CA certificates together with any additional certificates from `custom-cas/`.

### `mozilla-cacert.pem`

The CA bundle generated directly from the Mozilla NSS certificate store.

This file contains only the Mozilla certificates and does not contain custom CAs.

### `metadata.json`

Contains provenance and generation information for the bundle, including:

- Mozilla NSS source URL
- SHA-256 checksum of the Mozilla `certdata.txt`
- curl repository and commit
- `mk-ca-bundle.pl` version
- SHA-256 checksum of the exact `mk-ca-bundle.pl` script used
- custom CA statistics
- SHA-256 checksum of the final `cacert.pem`
- certificate counts
- generation timestamp

## Custom CA certificates

Additional CA certificates can be placed in the `custom-cas/` directory.

Only files with the `.pem` extension are processed.

Each custom certificate is validated as an X.509 certificate using OpenSSL before it is added to the bundle.

Custom certificates are:

- validated using OpenSSL
- checked against the Mozilla bundle using their SHA-256 fingerprint
- skipped if the certificate is already included in the Mozilla bundle
- processed in deterministic filename order
- added together with certificate metadata
- appended to the final `cacert.pem`

The generated metadata contains information about the number of custom certificates found, added and skipped as duplicates.

### Adding a custom CA

Place the PEM-encoded certificate in:

    custom-cas/

For example:

    custom-cas/
    ├── Example-Root-CA.pem
    └── Example-Intermediate-CA.pem

The next workflow run will automatically include valid certificates in `cacert.pem`.

### Removing a custom CA

Removing a `.pem` file from `custom-cas/` causes the certificate to be removed from the generated `cacert.pem` during the next bundle update.

## Generation

The Mozilla portion of the bundle is generated using curl's `mk-ca-bundle.pl`:

    ./mk-ca-bundle.pl -m -n mozilla-cacert.pem

The generated Mozilla bundle is then used as the base for the final `cacert.pem`.

Custom certificates from `custom-cas/` are subsequently validated, de-duplicated and appended.

## Validation

The generated CA bundle is validated using OpenSSL.

The workflow checks that:

- the bundle is not empty
- certificates contain matching `BEGIN` and `END` markers
- the bundle contains at least one certificate
- OpenSSL can parse the complete certificate bundle
- no trailing whitespace is present
- the bundle ends with exactly one newline

Individual custom CA certificates are also validated before they are added.

## Duplicate detection

Custom certificates are compared with the Mozilla bundle using their SHA-256 certificate fingerprint.

If a custom certificate is already present in the Mozilla CA bundle, it is not added again.

The same fingerprint tracking also prevents duplicate custom certificates from being added during a single workflow run.

## Change detection

The workflow does not compare the generated `cacert.pem` files byte-for-byte.

Instead, it extracts the SHA-256 fingerprint of every certificate and compares the resulting, sorted fingerprint lists.

This means that changes to:

- generation timestamps
- certificate ordering
- metadata comments

do not trigger an update by themselves.

Changes to the actual certificate set do trigger an update.

This includes:

- certificates added by Mozilla
- certificates removed by Mozilla
- custom CA certificates added to `custom-cas/`
- custom CA certificates removed from `custom-cas/`

## Automated updates

The workflow runs automatically on a scheduled basis and can also be started manually.

When certificate changes are detected, the workflow creates a pull request containing:

    cacert.pem
    metadata.json

The pull request contains information about:

- the Mozilla certificate source
- the curl commit used
- the `mk-ca-bundle.pl` version
- custom CA statistics
- the resulting certificate count
- the SHA-256 checksum of the final bundle

The generated pull request is intended to make CA bundle updates auditable and reviewable.

## Reproducibility and provenance

The exact versions and source data used to generate the bundle are recorded in `metadata.json`.

The metadata includes the SHA-256 checksum of:

- the Mozilla `certdata.txt`
- the exact `mk-ca-bundle.pl` script
- the final `cacert.pem`

This makes it possible to verify which source data and generator were used for a particular bundle.

The curl commit containing `mk-ca-bundle.pl` is pinned in the workflow and updated automatically by Renovate.

## Workflow

The update workflow is located at:

    .github/workflows/update.yml

The workflow:

1. Checks out the repository
2. Installs the required dependencies
3. Downloads the pinned `mk-ca-bundle.pl`
4. Validates the generator
5. Records the generator version and SHA-256 checksum
6. Downloads the Mozilla NSS `certdata.txt`
7. Calculates its SHA-256 checksum
8. Generates the Mozilla CA bundle
9. Validates the Mozilla bundle
10. Processes certificates from `custom-cas/`
11. Generates the final `cacert.pem`
12. Validates the final bundle
13. Compares certificate fingerprints with the committed bundle
14. Generates `metadata.json` when certificates changed
15. Creates a pull request containing the updated files

## Dependencies

The workflow uses:

- `curl`
- `jq`
- `openssl`
- `perl`

The GitHub Actions used by the workflow are pinned to commit SHAs.

## License and certificate sources

The Mozilla certificates are sourced from the Mozilla NSS Root Certificate Store.

The generated bundle is intended to provide the Mozilla trusted CA set together with locally supplied custom certificates.

Users of this repository are responsible for ensuring that custom certificates added under `custom-cas/` are appropriate for their intended environment.

## Disclaimer

This repository is not affiliated with or endorsed by Mozilla or curl.

Always review CA bundle changes before deploying them to security-sensitive systems.
