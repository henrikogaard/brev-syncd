# Contributing

`brev-syncd` is in its protocol-design stage. Contributions should stay within
the current phase and begin with an issue or focused design discussion when they
change protocol, cryptography, device lifecycle, storage, deployment, privacy,
or compatibility.

Read [`AGENTS.md`](AGENTS.md) and the documents under [`docs/`](docs/) before
working on the repository.

## Before opening a pull request

- Keep the change focused on one reviewed outcome.
- Use synthetic fixtures only; never commit real email data, credentials,
  account identifiers, device keys, or production diagnostics.
- Add tests first for code changes.
- Update the relevant architecture/protocol/threat/deployment documents.
- Append a factual entry to `WORKLOG.md`.
- State all checks run and all checks skipped.

Contributions are provided under the repository's AGPL-3.0 license. Third-party
dependencies require explicit compatibility and maintenance review.
