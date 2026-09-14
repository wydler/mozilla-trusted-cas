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

Place the PEM encoded certificate in:

```text
custom-cas/
