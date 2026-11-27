# Mozilla Thunderbird Instructions

This directory contains the codebase for Mozilla Thunderbird, which is contained
in an additional repository that is checked out inside the Mozilla Firefox
codebase.

## Code Structure

- The Thunderbird code is contained in `comm/`. That is the primary
  code that these rules apply to.
- The Firefox code is contained in all of the other subdirectories of the
  parent folder.
- The AGENTS.md file in the parent folder contains guidance for working with
  both the Firefox and Thunderbird code bases. However, there are additional
  rules and exceptions for Thunderbird that are described in this file.
  When Firefox and Thunderbird guidance are at odds, prefer
  Thunderbird guidance.
- Completely ignore the 'suite' folder, unless work is specifically directed at
  it.

## Version Control

Both projects are separate Git repositories.

The Firefox repository was checked out to the parent folder and the Thunderbird
repository was checked out to comm/ folder inside of it. Almost all
modifications needed when working on Thunderbird will be in the comm/ directory.

To move files that are version controlled, use the `git mv` command to ensure
the metadata for the move is recorded in Git.

## Searching

- When searching for usage examples, favor newer Thunderbird files and recently
  updated patterns over older examples. This codebase contains both current and
  out-of-date approaches.
- To use `searchfox-cli` for the comm/ repository, use the `--repo comm-central`
  command line argument.
- Prefer ripgrep (`rg`) for text searching. If not installed, prompt the user to
  install it.

## Build System

Thunderbird uses the same `mach` build command as Firefox. To build the app,
when necessary:

- You must be in the parent folder
- Run either './mach build faster', or if that fails, './mach build'
- Run this in a subagent as a single task if possible

### Linting Code

To format source code appropriately for Thunderbird, use the `../mach commlint`
command as follows from the `comm` directory:

```
$ ../mach commlint [file paths...]
```

Do NOT include the 'comm' folder in the path.

Do NOT use `../mach lint`, as that will fail.

## Writing and Reviewing Front-end Code

Utilize the skills available (e.g., `thunderbird-frontend-guidelines`, `writing-js-comments` , etc.)

## Automated Tests

There are multiple kinds of tests:

* xpcshell tests, often referred to as unit tests, are implemented in
  JavaScript and do not have an application window. They are used to test
  components at lower levels.
* mochitests, often referred to as browser tests, are also implemented in
  JavaScript and allow scripting of UI actions such as opening windows,
  clicking on UI elements.

The testing framework is not any of the commonly used testing frameworks, so
it's important not to assume any particular testing or mocking framework is
available.

For writing, running, or reviewing frontend tests, use the
`thunderbird-frontend-testing-guidelines` skill.

## Reviewer instructions for this fork

This repository is a fork of Thunderbird. The branch `pqc` carries a small patch series that lets Thunderbird's OpenPGP integration (`mail/extensions/openpgp/`) use RFC 9980 post-quantum keys: accept, display and use v6 and post-quantum keys, and generate them in the user interface. The OpenPGP engine underneath is `sequoia-octopus-librnp`, loaded in place of RNP. The work is done by a scheduled agent; every change comes as a pull request into `pqc`, and the agent merges only when your latest review of the latest commit has no open P0 or P1 findings. The patches are meant to be offered upstream, so review like a Thunderbird module peer.

## Review guidelines

Only changes under `mail/extensions/openpgp/`, its tests, its Fluent strings and the fork files are expected. Review the changed code in the context of the modules it calls (`RNP.sys.mjs`, `RNPLib.sys.mjs`, the key manager and key details dialogs).

Treat as **P0**:
- A signature, decryption or key-acceptance state shown to the user that is better than what the library reported, or a security check (key validity, expiry, revocation, acceptance, algorithm or feature checks) removed or bypassed instead of extended.
- Sending or saving a message less protected than the user chose (unencrypted, unsigned, or encrypted to fewer or other keys) because of a post-quantum code path.
- Secret keys left unprotected on disk, or passphrases, secret key material or plaintext written to the console or logs.
- A change in behaviour for existing keys and messages that the PR does not declare and justify.
- A `ctypes` declaration whose signature does not match the C function it binds.
- A test that does not test what it claims, or a test or lint that was disabled to get a green run.

Treat as **P1**:
- Algorithm names, key versions or fingerprint lengths hard-coded in a way that will break for the other RFC 9980 algorithms or for future ones.
- User-visible strings not in Fluent, strings added to locales other than en-US, or unclear wording for security-relevant choices.
- Missing tests for new behaviour; eslint findings; missing or wrong licence headers.
- Changes outside the expected paths, or unrelated refactoring that makes the patch harder to upstream.
- Fork-only files mixed into a code commit; they belong in commits whose message starts with `fork:`.

Do not report style that matches the surrounding Thunderbird code.

For each finding, quote the exact line, say what is wrong, and say what would fix it. If the PR has no P0 or P1 findings, say so explicitly in one sentence ("No P0/P1 findings."), so the agent can merge.
