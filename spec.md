# Semantic Localization Identifier (SLI) Specification

**Version: 1.0**  
**Author:** Iceflower <iceflower@iceflower.eu>  
**Maintainer:** Kenyoni Software <software@kenyoni.eu>  
**Repository:** [github.com/kenyoni-software/sli-spec](https://github.com/kenyoni-software/sli-spec)  
**License:** CC BY 4.0

## 1 Overview & Purpose

A **Semantic Localization Identifier (SLI)** is a structured, readable key for a translatable string.

SLIs provide enough semantic context for developers, reviewers, and translators to understand a string’s purpose without inspecting the source code. This predictable structure supports efficient grouping, filtering, and automated validation throughout localization pipelines.

This document defines the normative syntax, validation rules, and recommended practices for creating and maintaining SLIs. The terminology used in this document follows RFC 2119.

- **MUST** / **MUST NOT** indicate normative requirements.
- **SHOULD** / **SHOULD NOT** indicate recommendations.
- **MAY** indicate optional behavior.

## 2 Semantic Localization Identifier

A Semantic Localization Identifier (SLI) **MUST** follow this structure:

```text
[prefix][scope.]key
```

- `prefix` is optional.
- `scope.` (including the trailing dot) is optional; when present, it separates the scope from the key.
- `key` is required.

Each SLI **MUST** be unique within a project.  
SLIs are stable identifiers and **MUST** remain unchanged in any released, distributed, or otherwise published version of the project, as changing them breaks the link between the code and its translations. While translations **MAY** be updated at any time, SLIs **MAY** only be changed freely during development (prior to release).  
SLIs **MUST NOT** encode presentation rules (for example, capitalization, punctuation, or formatting instructions). Such elements **MUST** be handled in a separate UI/UX style guide to maintain a clean separation between string identity and visual display. SLIs **MAY**, however, reflect presentation context indirectly through scope (for example, using a `button` scope to indicate the string is a button label, from which a style guide can then infer that it requires title case).  
SLIs **SHOULD** be kept as short as possible while remaining semantically clear. Excessively long identifiers reduce readability and can cause issues with tooling or platforms that impose key length limits.  
SLIs **SHOULD** be human-readable and semantically meaningful to developers and translators. Cryptic abbreviations or opaque identifiers **SHOULD** be avoided. Context **SHOULD** be provided via scope or descriptive keys rather than relying on external documentation.

## 3 Prefix, Scope, Key

### 3.1 Prefix

The prefix is an optional element at the start of an SLI. Its purpose is to syntactically mark identifiers as translatable and to provide a consistent token that allows tools and scripts to reliably extract SLIs from source code or templates. When present, the prefix does not change the semantic meaning of the identifier. It is purely structural.

Within a project, either all SLIs **MUST** use the same prefix character or all SLIs **MUST NOT** include a prefix. The chosen prefix character **MUST** be a single character that is not a valid character in the scope, key, or placeholder syntax. Specifically, the characters `a-z`, `0-9`, `_`, `/`, and `.` **MUST NOT** be used as the prefix. If a prefix is used, it **MUST** appear at the start of the identifier.

The prefix **MAY** be used to facilitate automatic extraction of SLIs from source code or templates. Common choices for the prefix include `@`, `~`, or `#`.

### 3.2 Scope

The scope is an optional element that groups related SLIs and provides semantic context to prevent naming collisions. It represents a stable functional boundary, such as a feature module, a major UI section, a delivery channel, or a message category. Using a scope allows keys to remain short while still being unambiguous within their context.

The scope **MUST** consist of one or more components separated by slashes (`/`), with each component composed of ASCII lowercase letters (`a-z`), digits (`0-9`), or underscores (`_`). Scope components **MUST NOT** be empty (for example, `ui//button`), and the scope **MUST NOT** begin or end with a slash. When present, the scope **MUST** be followed by a literal dot (`.`) that separates it from the key.

Top-level scope components **SHOULD** be limited and stable. They **MAY** map to stable project boundaries (features, major UI areas, or channels). Ad hoc scope components and deep nesting **SHOULD** be avoided, as they increase fragmentation and make reuse harder. Scope components **MAY** be used to reflect presentation structure, such as `label`, `button`, or `tooltip`, but **SHOULD NOT** encode implementation-specific variants or technical details (for example, prefer `button` over `texture_button` or `animated_button`). Scope component names **SHOULD** use singular nouns (for example, `button` instead of `buttons`).  
If the localization framework supports pluralization or grammatical gender natively, those mechanisms **SHOULD** be used. Scope components for gender variants (for example, `female`, `male`, or `neutral`) or pluralization (for example, `one` or `other`) **SHOULD NOT** be used unless the framework lacks native support. The set of plural scope components **SHOULD** cover all CLDR plural categories required across all target languages of the project. For example, while English requires only `one` and `other`, Arabic requires `zero`, `one`, `two`, `few`, `many`, and `other`. Languages that do not distinguish all defined forms **MAY** map multiple scope components to identical translations.

### 3.3 Key

The key is a required element that uniquely identifies a translatable string within its scope. Placeholders embedded in the key represent dynamic runtime values. Embedding placeholders helps translators identify the required dynamic values directly from the identifier and enables direct fallback substitution. When no translation is available, the raw SLI remains functional and testable. The placeholder syntax is framework-agnostic and may therefore vary between projects.

Keys **MUST** consist of ASCII lowercase letters (`a-z`), digits (`0-9`), underscores (`_`), and placeholders. Keys **MUST NOT** be empty. Placeholders in the key **MUST** identify only the dynamic runtime value and **MUST NOT** contain formatting instructions. Translations **MAY** include formatting instructions within their placeholders. Within a project, all keys with dynamic values **MUST** consistently either embed all their placeholders or contain none at all. Embedding placeholders in keys **SHOULD** be preferred, especially when the raw SLI may be rendered directly as fallback text. If a key embeds placeholders, every associated translation **MUST** contain exactly those placeholders. The placeholders **MAY** be reordered within the translation text to accommodate different grammatical structures.  
A project **MUST** use a single, consistent placeholder syntax throughout. The localization framework's native placeholder syntax **SHOULD** be used, but it **MUST NOT** contain the structural characters `/` and `.`, nor the chosen prefix character. Names for named placeholders **MUST** contain only ASCII lowercase letters (`a-z`), digits (`0-9`), or underscores (`_`) and **MUST NOT** be empty.

Keys **SHOULD** remain concise and descriptive, revealing their intent and meaning (what the string represents) rather than implementation details or UI element types. Information already conveyed by the scope **SHOULD NOT** be duplicated. Underscores **MAY** be used to separate semantic words for readability. Variants of the same string that differ in detail or length **MAY** use semantic suffixes such as `_short` or `_long` directly in the key, for example, a brief inline error message versus an expanded detailed explanation of the same error.  
Placeholders **SHOULD** have clear, descriptive names reflecting the content they represent. While the localization framework's native placeholder syntax is preferred, any syntax **MAY** be used or placeholders **MAY** be used in translations only and omitted from keys entirely. The placeholder syntax used in the key and translation **MAY** differ, provided that every placeholder in the key can still be unambiguously identified and matched.

## 4 Regex

`PLACEHOLDER_REGEX` is a meta-variable representing the regular expression that validates the placeholder in the key.
`PREFIX_CHAR` is a meta-variable representing the single character used as a prefix in the project, if any.

```regex
^(?<prefix>PREFIX_CHAR)(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|(?<placeholder>PLACEHOLDER_REGEX))+)$
```

The regex captures the prefix, scope, and key of an SLI. It ensures that the identifier adheres to the specified character sets and structure. This regex **MAY** be used to validate or parse individual SLIs. It is not designed to validate a collection of SLIs, as it cannot enforce consistent prefix usage or placeholder delimiter syntax across a set of identifiers.

Regex with named placeholders using single curly braces `{}` as delimiters (`PLACEHOLDER_REGEX`: `\{[a-z0-9_]+\}`) and `@` as the prefix character:

```regex
^(?<prefix>@)(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|(?<placeholder>\{[a-z0-9_]+\}))+)$
```

Regex with printf-style unnamed placeholders with positional arguments (`PLACEHOLDER_REGEX`: `%(?:\d+\$)?[dfsu]`) and `#` as the prefix character:

```regex
^(?<prefix>#)(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|(?<placeholder>%(?:\d+\$)?[dfsu]))+)$
```

## 5 Backus-Naur Form Grammar

`<placeholder>` and `<prefix>` are project-defined tokens and **MUST** be specified by each project according to its chosen placeholder and prefix syntaxes.

```bnf
<SLI> ::= <prefix> ( <scope> "." )? <key>
<scope> ::= <scope_comp> ( "/" <scope_comp> )*
<scope_comp> ::= ( [a-z] | [0-9] | "_" )+
<key> ::= ( <key_char> | <placeholder> )+
<key_char> ::= [a-z] | [0-9] | "_"
```

Backus-Naur Form with named placeholders using single curly braces `{}` as delimiters and `@` as the prefix character:

```bnf
<SLI> ::= <prefix> ( <scope> "." )? <key>
<scope> ::= <scope_comp> ( "/" <scope_comp> )*
<scope_comp> ::= ( [a-z] | [0-9] | "_" )+
<key> ::= ( <key_char> | <placeholder> )+
<key_char> ::= [a-z] | [0-9] | "_"

<prefix> ::= "@"
<placeholder> ::= "{" ( [a-z] | [0-9] | "_" )+ "}"
```

## 6 Examples

Each example below is independent and represents a separate project.

### 6.1 Basic

- `cancel`: `Cancel` — Common short button label.
- `welcome_message`: `Welcome to our app.` — Generic welcome text.

### 6.2 Scopes

- `ui/common/button.cancel`: `Cancel` — Common cancel button in shared UI elements.
- `ui/common/button.submit`: `Submit` — Primary submit action for forms.
- `ui/dashboard/greeting.welcome_{username}`: `Welcome, {username}!` — Personalized dashboard greeting.
- `ui/settings.graphics`: `Graphics` — Title of the graphics settings page.
- `marketing/campaign/email_subject.welcome`: `Welcome to our project!` — Welcome email subject.
- `marketing/footer.unsubscribe_instructions_{unsubscribe_link}`: `To unsubscribe, click {unsubscribe_link}.` — Footer with link placeholder.

### 6.3 With prefix

- `~cancel`: `Cancel` — Cancel button text. With `~` prefix.
- `@ui/common/button.cancel`: `Cancel` — Cancel button text. Scoped with `@` prefix.
- `#notification/inbox/message.no_new`: `You have no new messages.` — Inbox empty-state. `#` prefix variant.

### 6.4 Placeholders

- `product/detail.price_value_{currency}_{amount}`: `{currency} {amount}` — Product price display, for example `EUR 19.99`. Using curly braces as placeholder syntax.
- `product/detail.price_value_{{currency}}_{{amount}}`: `{{currency}} {{amount}}` — Product price display, for example `EUR 19.99`. Using double curly braces as placeholder syntax.
- `order/confirmation.number_{order_id}`: `Order #{order_id} confirmed.` — Order confirmation with id.
- `order/confirmation.number_%1$d`: `Order #%1$d confirmed.` — Using printf-style unnamed placeholder syntax with positional argument.
- `~ui/button.cancel_@username@`: `Cancel @username@` — The placeholder uses `@` character as a delimiter, while `~` serves as the prefix.
- `auth/error.invalid_login_{attempts}`: `Sign in failed. {attempts} attempt(s) remaining.` — Error with dynamic count.
- `help/message.contact_us_{email_link}`: `For support, please contact us at {email_link}.` — The `{email_link}` placeholder becomes an inline clickable label when rendered.
- `dialog/prologue/merchant.welcome`: `Welcome, {player_name}, to my shop!`, `dialog/prologue/merchant.bye`: `Bye for now, take care {player_name}!` — Project uses translation-only placeholders (no placeholders in keys).

### 6.5 Plural / gender / cardinality as scope components

- `product/review/one.summary`: `Based on 1 review.` — Singular plural form as scope component.
- `product/review/other.summary_{count}`: `Based on {count} reviews.` — Plural form with count placeholder.
- `profile/greeting/female.welcome_{last_name}`: `Welcome, Ms. {last_name}.` — Gender-specific scope component example.
- `profile/greeting/male.welcome_{last_name}`: `Welcome, Mr. {last_name}.` — Gender-specific scope component example.
- `profile/greeting/neutral.welcome_{last_name}`: `Welcome, {last_name}.` — Neutral fallback scope component.

### 6.6 Variants

- `error/message.server_connection_failed_short`: `Disconnected.` — Brief system status.
- `error/message.server_connection_failed_long`: `Could not connect to the server. Please check your internet connection.` — Detailed troubleshooting message.
- `notification/email_subject.welcome_short`: `Welcome!` — Short email subject.
- `notification/email_subject.welcome_long`: `Welcome to ExampleApp — let’s get you started` — Longer subject for marketing variant.

### 6.7 Common UI labels / links / tooltips

- `ui/user_profile/label.email_address`: `Email address` — Label for email input.
- `auth/link.forgot_your_password`: `Forgot your password?` — Auth screen link text.
- `ui/modal/prompt.delete_confirmation`: `Are you sure you want to delete this?` — Modal confirmation prompt.
- `legal/link.terms_of_service`: `Terms of Service` — Legal link text.

### 6.8 Invalid examples

- `UI/Button.Cancel`: `Cancel` — Uses uppercase letters in scope and key (they **MUST** be lowercase).
- `ui//button.cancel`: `Cancel` — Contains an empty scope component (`//` is not allowed).
- `ui/user_profile/label.email address`: `Email address` — Contains a space in key (only `a-z`, `0-9`, `_`, and placeholders allowed).
- `ui/button..cancel`: `Cancel` — Malformed (consecutive dots are not allowed).
- `ui/button.cancel_{size-number}`: `Cancel {size-number}` — Hyphen used inside placeholder-like token, placeholder names **MUST** contain only `a-z`, `0-9`, `_`.
- `ui/button.cancel_{}`: `Cancel` — Empty placeholder (`{}`) is not allowed.
- `ui/button.cancel_{UserName}`: `Cancel {UserName}` — The placeholder contains uppercase letters, which **MUST** be lowercase.
- `ui/button.cancel_{username}`: `Cancel {user}` — Placeholders do not match between key and translation (`{username}` vs `{user}`).
- `@ui/button.cancel` and `ui/button.ok` — Do not mix prefixed and unprefixed SLIs within the same project.
- `@ui/button.cancel_{username}` and `ui/button.ok_{{username}}` — Do not mix different placeholder delimiter syntaxes within the same project.
- `order/confirmation.number_{order_id}`: `Order #{order_id} confirmed.`, `order/cancellation.number`: `Order #{order_id} cancelled.` — Do not mix keys that embed placeholders with keys that have placeholders only in translations.
- `@ui/button.cancel_@username@`: `Cancel @username@` — Do not use prefix characters in placeholder syntax.

### 6.9 Discouraged examples

- `ui/buttons.cancel`: `Cancel` — Uses plural form of `button` in scope.
- `product/detail.price_value_{currency}_{amount}`: `%currency% %amount%` — Placeholders in key and translation use different syntax.

## 7 Definitions

**Framework** — The system (library, platform, or toolchain) responsible for interpreting the identifier.

**Key** — The required part of an SLI that identifies a specific translatable string within its scope.

**Placeholder** — A named or unnamed slot embedded in the key and its associated strings that is substituted with a dynamic value at runtime.

**Prefix** — An optional single character at the start of an SLI used to mark it as a translatable identifier for tools and scripts.

**Project** — The entire codebase, application, or product that uses SLIs for localization.

**Scope** — An optional element of an SLI that groups related keys under a stable functional boundary to prevent naming collisions and provide context.

**Semantic Localization Identifier (SLI)** — A concise, machine-parsable identifier that maps to exactly one translatable unit in a project.

**Translation** — A language-specific string associated with an SLI, which **MAY** include markup or formatting syntax that is interpreted at display time.
