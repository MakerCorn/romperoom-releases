# Romperoom documentation

Start with the [project README](../README.md). Every document in this folder and every package
README is listed here; a test fails if one is not.

## For players

| Document                              | What it covers                                                        |
| ------------------------------------- | --------------------------------------------------------------------- |
| [User guide](user-guide.md)           | Every screen, the keyboard and gamepad controls, and common questions |
| [Troubleshooting](troubleshooting.md) | Unreachable folders, missing games, unreadable files, starting over   |
| [Supported systems](systems.md)       | Every system and the folder names that map to it (generated)          |
| [Roadmap](roadmap.md)                 | What is built, what is planned, and the known gaps                    |

## For engineers

| Document                                   | What it covers                                                                             |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ |
| [Architecture](architecture.md)            | Processes and IPC, the catalog schema, the scan and its guards, the journal, the hash pool, identification |
| [Development](development.md)              | Commands, the Electron binary, native modules, the renderer, known limitations             |
| [Testing](testing.md)                      | Test strategy, the fixture library, e2e suites, coverage, mutation testing, CI             |
| [Configuration](configuration.md)          | The data folder, environment variables, test seams, saved settings                         |
| [Device profiles and systems](profiles.md) | Every profile field, path safety, adding a system or a device                              |
| [Release](release.md)                      | Versioning, installers, the release workflow, publishing, signing, rollback                |
| [CI runners](ci-runners.md)                | Actions minutes, what runs where, the self-hosted Mac and Windows runners                  |

## For reviewers

| Document                  | What it covers                                                                         |
| ------------------------- | -------------------------------------------------------------------------------------- |
| [Security](security.md)   | Threat model, Electron hardening, CSP, protocols, every IPC channel, network isolation |
| [Decisions](decisions.md) | Short records of the main architecture decisions, with their costs                     |

## Packages

| Package                                                     | What it is                                      |
| ----------------------------------------------------------- | ----------------------------------------------- |
| `packages/engine`                                           | Catalog, scanner, hash pool, file operations and identify |
| `packages/profiles`                                         | The systems catalog and device profiles (data)  |
| `packages/ui`                                               | Themes, tokens and primitives                   |
| `apps/desktop`                                              | The Electron app                                |
| `packages/ui/THIRD_PARTY.md`                                | Bundled fonts and their licences                |

## Public releases repository

Players download from a separate, releases-only repository. Its documents are kept here, in
`releases-repo-template/`, and copied there
([release.md](release.md#publishing-to-the-public-repository)):

| Document                                             | What it covers                                                 |
| ---------------------------------------------------- | -------------------------------------------------------------- |
| [README](../README.md)        | Downloads, checksums, opening an unsigned build, data, privacy |
| [SECURITY.md](../SECURITY.md) | Private vulnerability reporting and supported versions         |

## Design history

The Milestone 1 design spec and
implementation plan record how the foundation was
designed and built, task by task. They are kept as history: where they disagree with the pages
above, the pages above are current. The Milestone 2
design spec does the same for offline DAT
identification.

## Screenshots

[screenshots/](screenshots/) holds the generated screenshots and their
[manifest](screenshots/manifest.json). They are regenerated, never edited by hand (see
[testing.md](testing.md#screenshots)).
