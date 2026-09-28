# EMQX package manifests

This repository is a public, append-only record of the EMQX packages EMQ has published.

For every published artifact it records the file name, the version, the source commit it was
built from, and its SHA-256 digest. Anyone can use these records to check that a downloaded
package is the one EMQ published.

## What this record proves, and what it does not

Read this before relying on the records.

**It detects tampering with a published artifact.** If a package is replaced or modified in
object storage, in a package repository, or on a mirror after it was recorded here, its digest
no longer matches this record. Recovering that fact does not depend on the systems that host
the package, because this repository is public and every clone is a copy.

**It does not by itself prove that a build was clean.** If a build system is compromised at
build time, it can produce an unintended artifact and record that artifact's true digest. The
record and the package then agree. Detecting that case requires reviewing what was built: use
the `git_commit` and `git_tag` fields to confirm the artifact came from the source you
expected.

Treat a digest match as evidence that the file has not changed since publication. It is not
evidence, on its own, that the contents are what you intended to ship.

## Layout

One file per artifact, named after the artifact:

```
manifests/<version>/<filename>.json
```

For example:

```
manifests/6.0.4/emqx-6.0.4-ubuntu22.04-amd64.deb.json
```

Records are only ever added. An existing file is never modified and never deleted. Any change
that touches an existing record is a defect or an attack, and is rejected by CI.

## Record format

```json
{
  "filename": "emqx-6.0.4-ubuntu22.04-amd64.deb",
  "version": "6.0.4",
  "git_commit": "d23856be50363d92a961a0d1710e857697af019f",
  "git_tag": "6.0.4",
  "git_tag_signed": true,
  "sha256": "9f2c...",
  "size_bytes": 128394021
}
```

| Field | Meaning |
| --- | --- |
| `filename` | Exact published file name |
| `version` | EMQX release version |
| `git_commit` | Source commit the artifact was built from |
| `git_tag` | Release tag, when the artifact was built from one |
| `git_tag_signed` | Whether that tag carried a valid signature |
| `sha256` | SHA-256 digest of the published file |
| `size_bytes` | Size of the published file |

## Verifying a package

Download the package, then compare it with its record:

```bash
FILE=emqx-6.0.4-ubuntu22.04-amd64.deb
REC=manifests/6.0.4/$FILE.json

jq -r '.sha256 + "  " + .filename' "$REC" | sha256sum -c -
```

On macOS, use `shasum -a 256 -c -`.

To check which source a package came from:

```bash
jq -r '"commit: " + .git_commit, "tag: " + (.git_tag // "none")' "$REC"
```

## Withdrawn packages

A withdrawn package is recorded by adding a new record, not by deleting or editing the original
one. The original record stays, so the history of what was published remains complete.

## Repository protections

- Force pushes and branch deletion are rejected.
- Commits must be signed.
- CI rejects any change that modifies or deletes an existing record.
- An external monitor outside GitHub pins the latest commit and reports any non-fast-forward
  change or any change to the repository's protection rules.

The last point matters. Branch protection is a repository setting, and anyone who administers
the repository can change it. What makes tampering detectable is that this repository is public
and replicated: independent clones and the external monitor will disagree with a rewritten
history. If you rely on these records, keep your own clone and compare.

## Publication timing

Records are added when a release becomes public. This repository is not an advance notice
channel, and the absence of a record for a version does not mean that version does not exist.

## Reporting a mismatch

If a package you downloaded does not match its record here, do not install it. Report it to
EMQ at <security@emqx.io> with the file name, the digest you computed, and where you downloaded
it from.
