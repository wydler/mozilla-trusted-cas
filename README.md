# Mozilla CA Bundle

This repository provides a PEM-encoded CA certificate bundle generated from the Mozilla NSS Root Certificate Store.

The bundle is generated automatically from Mozilla's `certdata.txt` using curl's `mk-ca-bundle.pl` conversion script.

## Files

| File | Description |
| --- | --- |
| [`cacert.pem`](cacert.pem) | Generated PEM-encoded CA certificate bundle |
| [`metadata.json`](metadata.json) | Machine-readable metadata about the source, generator and generated bundle |
| [`LICENSE`](LICENSE) | Mozilla Public License 2.0 |

## Source

The authoritative source for the Mozilla NSS Root Certificate Store is `certdata.txt`.

The file is maintained by Mozilla as part of Network Security Services (NSS):

<https://hg.mozilla.org/releases/mozilla-release/raw-file/default/security/nss/lib/ckfw/builtins/certdata.txt>

Mozilla documents `certdata.txt` as the authoritative source for the NSS root store.

## Generation

The bundle is generated using curl's [`mk-ca-bundle.pl`](https://github.com/curl/curl/blob/master/scripts/mk-ca-bundle.pl) script.

The equivalent generation command is:

```sh
./mk-ca-bundle.pl -f -m -n cacert.pem
```

The options used are:

- `-f` — overwrite an existing output file
- `-m` — include metadata comments from the Mozilla certificate data
- `-n` — use the locally downloaded `certdata.txt` instead of downloading it

The Mozilla source file and the curl conversion script are downloaded separately during the GitHub Actions workflow.

The version of `mk-ca-bundle.pl` used for a build is determined from the script itself and recorded in `metadata.json`.

## Reproducibility

The conversion script is fetched from a specific, immutable commit of the curl repository rather than from a moving branch such as `master`.

This ensures that a future workflow run uses the exact same conversion script until the pinned commit is intentionally updated.

The generated metadata records:

- the Mozilla `certdata.txt` source URL
- the SHA-256 checksum of the source file
- the curl repository
- the exact curl commit used
- the `mk-ca-bundle.pl` version
- the SHA-256 checksum of the generated bundle
- the number of certificates in the bundle
- the generation timestamp

This makes it possible to determine exactly which source data and conversion script were used to produce a particular `cacert.pem`.

## Trust Store

The generated bundle contains certificates selected by `mk-ca-bundle.pl` for server authentication.

The conversion process takes the trust information contained in Mozilla's `certdata.txt` into account. This is important because Mozilla's root store contains both trusted and explicitly distrusted certificates.

The resulting PEM file contains the CA certificates suitable for use by applications that require a traditional PEM-based CA bundle.

The bundle does not reproduce all policies and constraints of the Mozilla root store. In particular, additional constraints used by browsers are not represented in a traditional PEM CA bundle.

## Automated Updates

The bundle is updated automatically using GitHub Actions.

The workflow:

1. Downloads the current Mozilla `certdata.txt`.
2. Calculates its SHA-256 checksum.
3. Downloads the pinned version of `mk-ca-bundle.pl`.
4. Generates `cacert.pem`.
5. Validates the generated PEM bundle.
6. Calculates the bundle's SHA-256 checksum.
7. Updates `metadata.json`.
8. Creates a pull request if the generated files have changed.

The workflow runs automatically on a regular schedule and can also be started manually using GitHub Actions.

Changes are reviewed through a pull request rather than being committed directly to the default branch.

## Validation

The generated bundle is checked during the workflow to ensure that:

- the output contains PEM-encoded certificates
- the bundle contains a reasonable number of certificates
- OpenSSL can successfully parse the generated certificate bundle
- the generated SHA-256 checksum is recorded in `metadata.json`

## Usage

The generated bundle can be used by applications that accept a PEM-encoded CA certificate bundle.

For example, with curl:

```sh
curl --cacert cacert.pem https://example.com
```

Applications may also use the bundle through their respective CA certificate configuration mechanisms.

The file is intentionally provided as a standalone PEM bundle and does not modify or replace the operating system's native trust store.

## Licensing

The generated certificate bundle is derived from Mozilla's CA certificate store and is therefore distributed under the same license as the Mozilla source data: the Mozilla Public License 2.0 (MPL 2.0).

The MPL 2.0 license text is included in [`LICENSE`](LICENSE).

For more information about the Mozilla Public License:

<https://www.mozilla.org/en-US/MPL/2.0/>

The `mk-ca-bundle.pl` script is part of the curl project and is downloaded during the build process. It is not copied into this repository.

## Disclaimer

This repository provides a converted representation of Mozilla's CA certificate store.

The generated bundle is not independently curated. Trust decisions are based on the Mozilla source data and the conversion performed by `mk-ca-bundle.pl`.

Users are responsible for determining whether the resulting CA bundle is appropriate for their application and security requirements.

## Related Projects

- [Mozilla NSS](https://firefox-source-docs.mozilla.org/security/nss/)
- [curl](https://curl.se/)
- [curl CA Extract](https://curl.se/docs/caextract.html)
- [Mozilla Public License 2.0](https://www.mozilla.org/en-US/MPL/2.0/)
