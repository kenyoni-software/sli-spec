# Semantic Localization Identifier (SLI) Specification

**Version: 1.0-DRAFT**  
**Authored by** Iceflower <iceflower@iceflower.eu>  
**Maintainer:** Kenyoni Software <software@kenyoni.eu>  
**Repository:** [github.com/kenyoni-software/sli-spec](https://github.com/kenyoni-software/sli-spec)  
**License:** CC BY 4.0

## 1 Overview & Purpose

A **Semantic Localization Identifier (SLI)** is a structured, readable key for a translatable string.

SLIs provide unambiguous, collision-resistant identifiers for each translatable string, offering sufficient semantic context for developers, reviewers, and translators to understand the content without inspecting the source code. This predictable structure further supports efficient grouping, filtering, and automated validation throughout localization pipelines.

This document defines the normative syntax, validation rules and recommended practices for creating and maintaining SLIs.  

Terminology in this document follows RFC 2119:

- **MUST** / **MUST NOT** indicate normative requirements
- **SHOULD** / **SHOULD NOT** indicate recommendations
- **MAY** / **MAY NOT** indicate purely optional behavior

## 2 Semantic Localization Identifier

A Semantic Localization Identifier (SLI) **MUST** follow this structure:

```text
[prefix][scope.]key
```

- `prefix` is optional.
- `scope.` (including the trailing dot) is optional; when present it separates scope from key.
- `key` is required.

Each SLI **MUST** be unique within a project.  
SLIs are stable identifiers. They **MUST** remain unchanged in any released, distributed, or otherwise published version of the project, as changing them breaks the link between code and translations. Translations **MAY** be updated at any time. During development prior to release, SLIs **MAY** be changed freely.  
SLIs **MUST NOT** encode presentation rules, such as capitalization, punctuation, or specific formatting instructions, as part of the identifier itself. Such elements **MUST** be handled in a separate UI/UX style guide to maintain a clean separation between string identity and visual display. SLIs **MAY**, however, reflect presentation context indirectly through scope or key structure.  
SLIs **SHOULD** be kept as short as possible while remaining semantically clear. Excessively long identifiers reduce readability and can cause issues with tooling or platforms that impose key length limits.  
SLIs **SHOULD** be human-readable and semantically meaningful to developers and translators. Cryptic abbreviations or opaque identifiers **SHOULD** be avoided. Context **SHOULD** be provided via scope or descriptive keys rather than relying on external documentation.

## 3 Prefix, Scope, Key

### 3.1 Prefix

The prefix is an optional component at the start of an SLI. Its purpose is to syntactically mark identifiers as translatable and provide a consistent token that allows tools and scripts to reliably extract SLIs from source code or templates. When used, the prefix does not change the semantic meaning of the string. It is purely structural.

Prefixes **MUST** be exactly one character from the set `@`, `~`, or `#` and **MUST** appear at the very beginning of the identifier. Within a project, either all SLIs **MUST** use the same prefix character or all SLIs **MUST NOT** include a prefix. Mixing different prefixes or mixing prefixed and unprefixed SLIs **MUST NOT** occur.

A prefix **SHOULD** be used when the localization system requires clear differentiation between translatable identifiers and other keys or strings or when scanning source code for SLIs automatically.

### 3.2 Scope

The scope is an optional component that groups related SLIs and provides semantic context to prevent naming collisions. It represents a stable functional boundary, such as a feature module, a major UI section, a delivery channel, or a message category. Using a scope allows keys to remain short while still being unambiguous within their context.

Scopes **MUST** consist of one or more components separated by slashes (`/`), with each component composed of lowercase letters (`a–z`), digits (`0–9`), or underscores (`_`). Empty components **MUST NOT** be used (for example, `ui//button`) and a scope **MUST NOT** begin or end with a slash. When present, the scope **MUST** be followed by a literal dot (`.`) that separates it from the key.

Top-level scopes **SHOULD** be limited and stable. Scopes **MAY** map to durable project boundaries (features, major UI areas, or channels). Ad hoc scopes and deep nesting **SHOULD** be avoided, as they increase fragmentation and make reuse harder. Sub-scopes **MAY** be used to reflect presentation structure, such as `label`, `button`, or `tooltip`, but **SHOULD NOT** encode implementation-specific variants or technical details (for example, prefer `button` over `texture_button` or `animated_button`). Scope names **SHOULD** use singular nouns (for example `button` instead of `buttons`).  
If the localization framework supports pluralization or grammatical gender natively, those mechanisms **SHOULD** be used. Sub-scopes for gender variants (for example `female`, `male`, or `neutral`) or pluralization (for example `one` or `other`) **MAY** be used only when the framework provides no native support for these forms. The set of plural sub-scopes **SHOULD** cover all CLDR plural categories required across all target languages of the project. For example, while English requires only `one` and `other`, Arabic requires `zero`, `one`, `two`, `few`, `many`, and `other`. Languages that do not distinguish all defined forms **MAY** map multiple sub-scopes to identical translations.

### 3.3 Key

The key is a required component that uniquely identifies a translatable string within its scope. Placeholders embedded in the key represent dynamic runtime values. Embedding placeholders in the key helps translators identify required dynamic values directly from the identifier and enables direct fallback substitution. When no translation is available, the raw SLI remains functional and testable. Placeholders are framework-agnostic and thus the placeholder syntax may differ between projects.

Keys **MUST** consist of lowercase letters (`a–z`), digits (`0–9`), underscores (`_`), and placeholders. Keys **MUST NOT** be empty. Embedding placeholders in keys **SHOULD** be preferred, especially when the raw SLI may be rendered directly as fallback text. Within a project, placeholders **MUST** either always be embedded or never embedded. When placeholders are embedded, every translation **MUST** contain all placeholders defined in the key, neither omitting nor adding any. Placeholders **MAY** be reordered within the translation text to accommodate different grammatical structures in various languages.  
The localization framework's native placeholder syntax **SHOULD** be used. Characters used in the placeholder syntax **MUST** be used only within the placeholder. If the framework's syntax conflicts with SLI reserved characters (`@`, `~`, `#`, `/`, or `.`), curly braces `{}` **SHOULD** be used as delimiters. If neither the native syntax nor curly braces are viable, an arbitrary syntax **MAY** be used or placeholders **MAY** be used in translations only and omitted from keys entirely. Placeholders **MUST NOT** contain framework-specific formatting metadata. Names for named placeholders **MUST** contain only lowercase letters (`a–z`), digits (`0–9`), or underscores (`_`) and **MUST NOT** be empty.

Keys **SHOULD** remain concise and semantic, revealing their intent and meaning (what the string represents) rather than implementation details or UI element types. Information already conveyed by the scope **SHOULD NOT** be duplicated. Underscores **MAY** be used to separate semantic words for readability. Variants of the same string that differ in detail or length **MAY** use semantic suffixes such as `_short` or `_long` directly in the key, for example, a brief inline error message versus an expanded detailed explanation of the same error.  
Placeholders **SHOULD** have clear, semantic names that describe the content they represent.

## 4 Regex

`KEY_BODY` is a placeholder representing the regular expression that validates the key, including placeholders.

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:KEY_BODY)+)$
```

The regex captures the optional prefix, optional scope, and required key components of an SLI. It ensures that the identifier adheres to the specified character sets and structure. This regex **MAY** be used to validate or parse single SLIs. It is not designed to validate multiple SLIs, as it cannot enforce consistent prefix usage or placeholder delimiter syntax across a set of identifiers.

Regex with named placeholders using single curly braces `{}` as delimiters:

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|\{[a-z0-9_]+\})+)$
```

Regex with named placeholders using double curly braces `{{}}` as delimiters:

```regex
^(?<prefix>[@~#])?(?:(?<scope>[a-z0-9_]+(?:\/[a-z0-9_]+)*)\.)?(?<key>(?:[a-z0-9_]|\{\{[a-z0-9_]+\}\})+)$
```

## 5 Examples

Each example below is independent and represents a separate project.

### 5.1 Basic

- `cancel`: `Cancel` — Common short button label.
- `welcome_message`: `Welcome to our app.` — Generic welcome text.

### 5.2 Scopes

- `ui/common/button.cancel`: `Cancel` — Common cancel button in shared UI components.
- `ui/common/button.submit`: `Submit` — Primary submit action for forms.
- `ui/dashboard/greeting.welcome_{username}`: `Welcome, {username}!` — Personalized dashboard greeting.
- `ui/settings/page.title`: `Settings` — Title of the settings page.
- `marketing/campaign/email_subjects.welcome`: `Welcome to our project!` — Welcome email subject.
- `marketing/footer.unsubscribe_instructions_{unsubscribe_link}`: `To unsubscribe, click {unsubscribe_link}.` — Footer with link placeholder.

### 5.3 With prefix

- `~cancel`: `Cancel` — Cancel button text. With `~` prefix.
- `@ui/common/button.cancel`: `Cancel` — Cancel button text. Scoped with `@` prefix.
- `#notification/inbox/message.no_new`: `You have no new messages.` — Inbox empty-state. `#` prefix variant.

### 5.4 Placeholders

- `product/detail.price_value_{currency}_{amount}`: `{currency} {amount}` — Product price display, for example `EUR 19.99`. Using curly braces as placeholder syntax.
- `product/detail.price_value_{{currency}}_{{amount}}`: `{{currency}} {{amount}}` — Product price display, for example `EUR 19.99`. Using double curly braces as placeholder syntax.
- `order/confirmation.number_{order_id}`: `Order #{order_id} confirmed.` — Order confirmation with id.
- `order/confirmation.number_{{order_id]}`: `Order #{{order_id}} confirmed.` — Using double curly braces as placeholder delimiters.
- `order/confirmation.number_%1$d`: `Order #%1$d confirmed.` — Using printf-style unnamed placeholder syntax with positional argument.
- `auth/error.invalid_login_{attempts}`: `Sign in failed. {attempts} attempt(s) remaining.` — Error with dynamic count.
- `help/message.contact_us_{email_link_text}`: `For support, please contact us at {email_link_text}.` — The `{email_link_text}` placeholder becomes an inline clickable label when rendered.
- `dialog/prolog/merchant_greeting.welcome`: `Welcome, {player_name}, to my shop!` — In-game dialog with player name placeholder only in translation, not in key.

### 5.5 Plural / gender / cardinality as sub-scope

- `product/review/one.summary`: `Based on 1 review.` — Singular plural form as sub-scope.
- `product/review/other.summary_{count}`: `Based on {count} reviews.` — Plural form with count placeholder.
- `profile/greeting/female.welcome_{last_name}`: `Welcome, Ms. {last_name}.` — Gender-specific sub-scope example.
- `profile/greeting/male.welcome_{last_name}`: `Welcome, Mr. {last_name}.` — Gender-specific sub-scope example.
- `profile/greeting/neutral.welcome_{last_name}`: `Welcome, {last_name}.` — Neutral fallback sub-scope.

### 5.6 Variants

- `error/message.server_connection_failed_short`: `Disconnected.` — Brief system status.
- `error/message.server_connection_failed_long`: `Could not connect to the server. Please check your internet connection.` — Detailed troubleshooting message.
- `notification/email_subject.welcome_short`: `Welcome!` — Short email subject.
- `notification/email_subject.welcome_long`: `Welcome to ExampleApp — let’s get you started` — Longer subject for marketing variant.

### 5.7 Common UI labels / links / tooltips

- `ui/user_profile/label.email_address`: `Email address` — Label for email input.
- `auth/link.forgot_your_password`: `Forgot your password?` — Auth screen link text.
- `ui/modal/prompt.delete_confirmation`: `Are you sure you want to delete this?` — Modal confirmation prompt.
- `legal/link.terms_of_service`: `Terms of Service` — Legal link text.

### 5.8 Invalid / discouraged examples

- `UI/Button.Cancel`: `Cancel` — Uses uppercase letters in scope and key (keys/scopes **MUST** be lowercase).
- `ui//button.cancel`: `Cancel` — Empty scope component (no `//` allowed).
- `ui/button cancel`: `Cancel` — Contains a space in key (only `a–z`, `0–9`, `_` and placeholders allowed).
- `ui/button..cancel`: `Cancel` — Malformed (extra dot or empty key component).
- `ui/buttons.cancel`: `Cancel` — Uses plural form of `button` in scope, which is not recommended.
- `ui/button.cancel_{size-number}`: `Cancel {size-number}` — Hyphen used inside placeholder-like token, placeholder names **MUST** contain only `a–z`, `0–9`, `_`.
- `ui/button.cancel_{}`: `Cancel` — Empty placeholder (`{}`) is not allowed.
- `ui/button.cancel_{UserName}`: `Cancel {UserName}` — Placeholder contains uppercase letters, **MUST** be lowercase.
- `ui/button.cancel_@UserName@`: `Cancel @UserName@` — Placeholder uses reserved characters (`@`).
- `ui/button.cancel_{username}`: `Cancel {user}` — Placeholders do not match between key and translation (`{username}` vs `{user}`).
- `@ui/button.cancel` and `ui/button.ok` — Do not mix prefixed and unprefixed SLIs within the same project.
- `@ui/button.cancel_{username}` and `ui/button.ok_{{username}}` — Do not mix different placeholder delimiter syntaxes within the same project.
- `order/confirmation.number_{order_id}`: `Order #{order_id} confirmed.` and `order/cancellation.number`: `Order #{order_id} cancelled.` — Do not mix placeholder embedding in keys with placeholders only in translations.

## 6 Definitions

**Key** — the required part of an SLI that identifies a specific translatable string within its scope.

**Placeholder** — a named or unnamed slot embedded in a key and its associated strings that is substituted with a dynamic value at runtime.

**Prefix** — an optional single character at the start of an SLI used to mark it as a translatable identifier for tools and scripts.

**Project** — the entire codebase, application, or product that uses SLIs for localization.

**Scope** — an optional component of an SLI that groups related keys under a stable functional boundary to prevent naming collisions and provide context.

**Semantic Localization Identifier (SLI)** — a concise, machine-parseable identifier that maps to exactly one translatable unit in a project.

**Translation** — a language-specific version associated with an SLI, which **MAY** include markup or formatting syntax that is interpreted at display time.
