# Semantic Localization Identifier (SLI)

[![Version](https://img.shields.io/badge/Version-1.0-orange)](spec.md)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-blue.svg)](LICENSE.md)

A naming convention for the keys of translatable strings. SLIs are structured, readable keys that tell developers, translators and tooling exactly what a string is and where it belongs — just from the identifier itself.

## Format

```text
[prefix][scope.]key
```

```text
@ui/dashboard/greeting.welcome_{username}
```

| Part   | Value                   | Optional |
| ------ | ----------------------- | -------- |
| Prefix | `@`                     | Yes      |
| Scope  | `ui/dashboard/greeting` | Yes      |
| Key    | `welcome_{username}`    | No       |

## Key Features

- predictable structure that tooling can parse, validate, and extract reliably
- scopes group related strings under stable functional boundaries
- a prefix makes SLIs unambiguously extractable from source code and templates by tools and scripts
- placeholders embedded in the key make the identifier self-documenting and functional as fallback text without any translation present
- placeholder syntax is framework-agnostic and adaptable to existing localization pipelines
- the entire structure validates with a single regex

*Keys may contain placeholders for dynamic values, but don't have to.*

## Examples

*Each row is an independent illustration and does not represent a single project.*

| SLI                                             | Translation                                                             |
| ----------------------------------------------- | ----------------------------------------------------------------------- |
| `ui/common/button.cancel`                       | Cancel                                                                  |
| `ui/dashboard/greeting.welcome_{username}`      | Welcome, {username}!                                                    |
| `auth/error.invalid_login_{attempts}`           | Sign in failed. {attempts} attempt(s) remaining.                        |
| `#error/message.server_connection_failed_short` | Disconnected.                                                           |
| `@error/message.server_connection_failed_long`  | Could not connect to the server. Please check your internet connection. |

## Specification

The full specification is in [spec.md](spec.md).

Minor editorial changes, such as typo fixes or rewording that does not change the intended meaning, may be made without notice. Breaking changes will always be documented.

## Best Practices

Document:

- the prefix character in use (if any)
- placeholder syntax (if embedded placeholders are used)
- common scope naming conventions and patterns (e.g. `ui/`, `auth/`, `error/`)

## License

This specification is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE.md).

This license requires that reusers give credit to the creator. It allows reusers to distribute, remix, adapt, and build upon the material in any medium or format, even for commercial purposes.
