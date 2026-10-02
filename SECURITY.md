# Security policy

## Reporting a vulnerability

Please report security problems **privately**, through GitHub's private vulnerability
reporting: open this repository's **Security** tab and choose **Report a vulnerability**. Only
the maintainers see the report. Please do not open a public issue for a security problem.

Include, if you can:

- the Romperoom version and your operating system and its version;
- what an attacker could do, and what they need first (for example, a crafted file in your ROM
  folder);
- the steps to reproduce it.

You will get an answer in the report's thread. When a fix ships, its release notes say so,
and you are credited unless you ask not to be.

## Supported versions

Only the **latest release** gets security fixes. Romperoom is in beta (`0.x`), and fixes ship
as a new version rather than as patches to older ones: update to the newest release on the
releases page.

## Scope

In scope: the Romperoom app as published here and its installers, including anything a crafted
file or folder in a library can make the app do. Out of scope: problems in the operating system
itself, and harmful files that sit in a library which Romperoom only lists (it catalogues
files; it does not run games or the programs in your library).

## Checking that a download is genuine

Every release lists the SHA-256 checksum of each file in `SHA256SUMS.txt`; see
[Check your download](README.md#check-your-download). The beta builds are not code-signed yet,
so the checksum, downloaded from this repository's releases page, is how you know a file is the
one that was published.
