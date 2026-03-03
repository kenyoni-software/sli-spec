# Semantic Localization Identifier (SLI)

![Version](https://img.shields.io/badge/version-1.0--draft-blue)
![License](https://img.shields.io/badge/license-CC%20BY%204.0-green)
![Status](https://img.shields.io/badge/status-draft-orange)

A naming convention for the keys of translatable strings. SLIs are structured, readable keys that tell developers, translators and tooling exactly what a string is and where it belongs — just from the identifier itself.

## Format

```text
[prefix][scope.]key
```

```text
@ui/dashboard/greetings.welcome_{username}
```

| Part   | Value                    | Optional |
| ------ | ------------------------ | -------- |
| Prefix | `@`                      | Yes      |
| Scope  | `ui/dashboard/greetings` | Yes      |
| Key    | `welcome_{username}`     | No       |

## Key Features

- predictable structure that tooling can parse, validate, and extract reliably
- scopes group related strings under stable functional boundaries
- a prefix makes SLIs unambiguously extractable from source code and templates by tools and scripts
- placeholders embedded in the key make the identifier self-documenting and functional as fallback text without any translation present
- placeholder syntax is framework-agnostic and adaptable to existing localization pipelines
- the entire structure validates with a single regex

*Keys may contain placeholders for dynamic values, but don't have to.*

## Examples

| SLI                                             | Translation                                                             |
| ----------------------------------------------- | ----------------------------------------------------------------------- |
| `ui/common/buttons.cancel`                      | Cancel                                                                  |
| `ui/dashboard/greetings.welcome_{username}`     | Welcome, {username}!                                                    |
| `auth/errors.invalid_login_{attempts}`          | Sign in failed. {attempts} attempt(s) remaining.                        |
| `#error/messages.server_connection_failed_short` | Disconnected.                                                           |
| `@error/messages.server_connection_failed_long` | Could not connect to the server. Please check your internet connection. |

## Specification

The full specification is in [spec.md (1.0-draft)](./spec.md).
