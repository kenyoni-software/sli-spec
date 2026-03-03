# Semantic Localization Identifier (SLI) Specification

**Version: 1.0-DRAFT**  
**Authored by** Iceflower <iceflower@iceflower.eu>  
**Maintainer:** Kenyoni Software <software@kenyoni.eu>  
**License:** CC BY 4.0

## 1 Overview & Purpose

A **Semantic Localization Identifier (SLI)** is a structured, readable key for a translatable string.

SLIs provide unambiguous, collision-resistant identifiers for each localizable string, offering sufficient semantic context for developers, reviewers and translators to understand the content without needing to inspect the source code. This predictable structure further supports efficient grouping, filtering and automated validation throughout localization pipelines.

This document defines the normative syntax, validation rules and recommended practices for creating and maintaining SLIs.  
This specification assumes that the localization framework and programming language in use can accommodate the SLI syntax without conflicts. If the framework's native placeholder syntax conflicts with reserved characters (`@`, `~`, `#`, `/` or `.`) and curly braces `{}` also carry special meaning within that framework, this specification cannot be applied without modification at the framework level.

Terminology in this document follows RFC-2119:

- **must** / **must not** / **shall** / **shall not** indicate normative requirements
- **should** / **should not** indicate recommendations
- **may** / **may not** indicate purely optional behavior

## 2 Semantic Localization Identifier

A Semantic Localization Identifier (SLI) **must** follow this structure:

```text
[prefix][scope.]key
```

- `prefix` is optional.
- `scope.` (including the trailing dot) is optional; when present it separates scope from key.
- `key` is required.

The fully qualified SLI **must** be unique within a product.
SLIs are stable identifiers. They **must** remain unchanged in any released, distributed or otherwise published version of the product, as changing them breaks the link between code and translations. Translation content **may** be updated freely at any time. Between releases, SLIs **may** be changed freely.  
SLIs **must not** encode presentation rules, such as capitalization, punctuation or specific formatting instructions, as part of the identifier itself. Such elements **must** be handled in a separate UI/UX style guide to maintain a clean separation between string identity and visual display. SLIs **may**, however, reflect presentation context indirectly through scope or key structure.  
SLIs **should** be kept as short as possible while remaining semantically clear. Excessively long identifiers reduce readability and can cause issues with tooling or platforms that impose key length limits.  
SLIs **should** be human-readable and semantically meaningful to developers and translators. Cryptic abbreviations or opaque identifiers **should** be avoided. Context **should** be provided via scope or descriptive keys rather than relying on external documentation.

## 3 Prefix, Scope, Key

### 3.1 Prefix

The prefix is an optional component at the start of an SLI. Its purpose is to syntactically mark identifiers as translatable and provide a consistent token that allows tools and scripts to reliably extract SLIs from source code or templates. When used, the prefix does not change the semantic meaning of the string; it is purely structural.

Prefixes **must** be exactly one character from the set `@`, `~` or `#` and appear at the very beginning of the identifier. Within a product, either all SLIs **must** use the same prefix character or all SLIs **must not** include a prefix. Mixing different prefixes or mixing prefixed and unprefixed SLIs **must not** occur.

A prefix **should** be used when the localization system requires clear differentiation between translatable identifiers and other keys or strings or when scanning source code for SLIs automatically.

### 3.2 Scope

The scope is an optional component that groups related SLIs and provides semantic context to prevent naming collisions. It represents a stable functional boundary, such as a feature module, a major UI section, a delivery channel or a message category. Using a scope allows keys to remain short while still being unambiguous within their context.

Scopes **must** consist of one or more components separated by slashes (`/`), with each component composed of lowercase letters (`a–z`), digits (`0–9`) or underscores (`_`). Empty components **must not** be used (for example, `ui//buttons`) and a scope **must not** begin or end with a slash. When present, the scope **must** be followed by a literal dot (`.`) that separates it from the key.

Top-level scopes **should** be limited and stable. Scopes **may** map to durable product boundaries (features, major UI areas or channels). Ad-hoc scopes and deep nesting **should** be avoided, as they increase fragmentation and make reuse harder. Sub-scopes **may** be used to reflect presentation structure, such as `labels`, `buttons` or `tooltips`, but **should not** encode implementation-specific variants or technical details (for example, prefer `buttons` over `texture_buttons` or `animated_buttons`).  
If the localization framework supports pluralization or grammatical gender natively, those mechanisms **should** be used. Sub-scopes for gender variants (for example `female`, `male` or `neutral`) or pluralization (for example `one` or `other`) **may** be used only when the framework provides no native support for these forms. The set of plural sub-scopes **should** cover all CLDR plural categories required across all target languages of the product. For example, while English requires only `one` and `other`, Arabic requires `zero`, `one`, `two`, `few`, `many` and `other`. Languages that do not distinguish all defined forms **may** map multiple sub-scopes to identical translations.

### 3.3 Key

The key is a required component that uniquely identifies a translatable string within its scope. Placeholders are embedded within the key and represent dynamic runtime values. Embedding placeholders in the key allows translators to identify required dynamic values directly from the identifier and enables direct fallback substitution when no translation is available, the raw SLI remains functional and testable without any translation present.

Keys **must** consist of lowercase letters (`a–z`), digits (`0–9`), underscores (`_`) and placeholders. The characters forming the chosen placeholder delimiter syntax are permitted solely within placeholder tokens and **must not** appear outside of them. Keys **must not** be empty.  
Placeholders represent dynamic runtime values. Embedding placeholders in keys **should** be preferred, especially when the raw SLI may be rendered directly as fallback text. If the product always resolves translations before display, placeholders **may** instead appear only in the translation values. Within a product, placeholders **must** either always be embedded in keys or never, mixing both approaches **must not** occur.  
By default, placeholders **must** be enclosed in curly braces `{}`. If the localization framework provides a native placeholder syntax that does not use characters reserved by the SLI syntax (`@`, `~`, `#`, `/` or `.`), that syntax **should** be used instead. Within a product, all SLIs **must** use the same placeholder delimiter syntax consistently. If the framework's syntax conflicts with reserved characters and curly braces also carry special meaning within that framework, this specification cannot be applied without modification.  
Regardless of the delimiter syntax used, placeholder names **must** contain only lowercase letters (`a–z`), digits (`0–9`) or underscores (`_`) and **must not** be empty. Every placeholder defined in the key **must** appear in every translation. Translations **must not** introduce additional placeholders or omit any placeholder present in the key. Placeholders **may** be reordered within the translation text to accommodate different grammatical structures in various languages. Translations **may** extend placeholder tokens with framework-specific formatting metadata (for example `{value, number, percent}` in ICU MessageFormat), provided the placeholder name itself matches exactly.

Keys **should** remain concise and semantic, revealing their intent and meaning (what the string represents) rather than implementation details or UI element types. Information already conveyed by the scope **should not** be duplicated. Underscores **may** be used to separate semantic words for readability. Variants of the same string that differ in detail or length **may** use semantic suffixes such as `_short` or `_long` directly in the key, for example, a brief inline error message versus an expanded detailed explanation of the same error.  
Placeholders **should** have clear, semantic names that describe the content they represent.

## 4 Regex

`KEY_REGEX` is a placeholder for the actual regex that captures the allowed characters and structure of the key, including placeholders.

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:KEY_REGEX)+)$
```

The regex captures the optional prefix, optional scope and required key components of an SLI. It ensures that the identifier adheres to the specified character sets and structure. This regex **may** be used to validate or parse single SLIs. It is not designed to validate multiple SLIs, as it cannot not enforce consistent prefix usage or placeholder delimiter syntax across a set of identifiers.

Regex with single curly braces `{}` as placeholder delimiters:

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|\{[a-z0-9_]+\})+)$
```

Regex with double curly braces `{{}}` as placeholder delimiters:

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|\{\{[a-z0-9_]+\}\})+)$
```

## 5 Examples

Each example line below represents a separate product.

### 5.1 Basic

- `cancel`: `Cancel` — Common short button label.
- `welcome_message`: `Welcome to our app.` — Generic welcome text.

### 5.2 Scopes

- `ui/common/buttons.cancel`: `Cancel` — Common cancel button in shared UI components.
- `ui/common/buttons.submit`: `Submit` — Primary submit action for forms.
- `ui/dashboard/greetings.welcome_{username}`: `Welcome, {username}!` — Personalized dashboard greeting.
- `ui/settings/pages.title`: `Settings` — Title of the settings page.
- `marketing/campaigns/email_subjects.welcome`: `Welcome to our product!` — Welcome email subject.
- `marketing/footers.unsubscribe_instructions`: `To unsubscribe, click {unsubscribe_link}.` — Footer with link placeholder.

### 5.3 With prefix

- `~cancel`: `Cancel` — Cancel button text. With `~` prefix.
- `@ui/common/buttons.cancel`: `Cancel` — Cancel button text. Scoped with `@` prefix.
- `#notifications/inbox/messages.no_new`: `You have no new messages.` — Inbox empty-state. `#` prefix variant.

### 5.4 Placeholders

- `product/details.price_value_{currency}_{amount}`: `{currency} {amount}` — Product price display, e.g. `EUR 19.99`. Using curly braces as placeholder syntax.
- `product/details.price_value_{{currency}}_{{amount}}`: `{{currency}} {{amount}}` — Product price display, e.g. `EUR 19.99`. Using double curly braces as placeholder syntax.
- `order/confirmation.number_{order_id}`: `Order #{order_id} confirmed.` — Order confirmation with id.
- `auth/errors.invalid_login_{attempts}`: `Sign in failed. {attempts} attempt(s) remaining.` — Error with dynamic count.
- `help/messages.contact_us_{email_link_text}`: `For support, please contact us at {email_link_text}.` — The `{email_link_text}` placeholder becomes an inline clickable label when rendered.

### 5.5 Plural / gender / cardinality as sub-scope

- `product/reviews/one.summary`: `Based on 1 review.` — Singular plural form as sub-scope.
- `product/reviews/other.summary_{count}`: `Based on {count} reviews.` — Plural form with count placeholder.
- `profile/greetings/female/{last_name}`: `Welcome back, Ms. {last_name}.` — Gender-specific sub-scope example.
- `profile/greetings/male/{last_name}`: `Welcome back, Mr. {last_name}.` — Gender-specific sub-scope example.
- `profile/greetings/neutral/{last_name}`: `Welcome back, {last_name}.` — Neutral fallback sub-scope.

### 5.6 Variants

- `error/messages.server_connection_failed_short`: `Disconnected.` — Brief system status.
- `error/messages.server_connection_failed_long`: `Could not connect to the server. Please check your internet connection.` — Detailed troubleshooting message.
- `notifications/email_subjects.welcome_short`: `Welcome!` — Short email subject.
- `notifications/email_subjects.welcome_long`: `Welcome to ExampleApp — let’s get you started` — Longer subject for marketing variant.

### 5.7 Common UI labels / links / tooltips

- `ui/user_profile/labels.email_address`: `Email address` — Label for email input.
- `auth/links.forgot_your_password`: `Forgot your password?` — Auth screen link text.
- `ui/modal/prompts.delete_confirmation`: `Are you sure you want to delete this?` — Modal confirmation prompt.
- `legal/links.terms_of_service`: `Terms of Service` — Legal link text.

### 5.8 Invalid / disallowed examples

- `UI/Buttons.Cancel`: `Cancel` — Uses uppercase letters in scope and key (keys/scopes **must** be lowercase).
- `ui//buttons.cancel`: `Cancel` — Empty scope component (no `//` allowed).
- `ui/buttons cancel`: `Cancel` — Contains a space in key (only `a–z`, `0–9`, `_` and placeholders allowed).
- `ui/buttons..cancel`: `Cancel` — Malformed (extra dot or empty key component).
- `ui/buttons.cancel{size-number}`: `Cancel` — Hyphen used inside placeholder-like token, placeholders names **must** contain only `a–z`, `0–9`, `_`.
- `ui/buttons.cancel_{}`: `Cancel` — Empty placeholder (`{}`) is not allowed.
- `ui/buttons.cancel_{UserName}`: `Cancel {UserName}` — Placeholder contains uppercase letters, **must** be lowercase.
- `ui/buttons.cancel_@UserName@`: `Cancel @UserName@` — Placeholder uses reserved characters (`@`).
- `ui/buttons.cancel_{username}`: `Cancel {user}` — Placeholders do not match between key and translation (`{username}` vs `{user}`).
- `@ui/buttons.cancel` and `ui/buttons.cancel` — Do not mix prefixed and unprefixed SLIs within the same product.
- `@ui/buttons.cancel_{username}` and `ui/buttons.ok_{{username}}` — Do not mix different placeholder delimiter syntaxes within the same product.

## 6 Definitions

**Key** — the required part of an SLI that identifies a specific translatable string within its scope.

**Placeholder** — a named slot embedded in a key and its associated strings, that is substituted with a dynamic value at runtime.

**Prefix** — an optional single character at the start of an SLI used to mark it as a translatable identifier for tools and scripts.

**Scope** — an optional component of an SLI that groups related keys under a stable functional boundary to prevent naming collisions and provide context.

**Semantic Localization Identifier (SLI)** — a concise, machine-parseable identifier that maps to exactly one translatable unit in a product.

**Translation** — a language-specific version associated with an SLI, which **may** include markup or formatting syntax that is interpreted at display time.
