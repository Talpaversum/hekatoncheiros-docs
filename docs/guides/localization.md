# Localization contract

Hekatoncheiros localization contract version 1 uses canonical, lowercase BCP 47
locale identifiers and flat UTF-8 JSON resources. Core and Web support `en`,
`cs`, `sk`, `de`, `fr`, and `es`. English is always the default and final
fallback. Every application must bundle English; applications may declare
additional canonical locales.

## Translation resources

Resource paths use `locales/<locale>.json` and the format identifier
`hc-flat-json-v1`. Keys are stable dot-separated identifiers such as
`account.profile.title`; they must not contain locale or presentation text.
Values are non-empty strings. Named placeholders use `{{name}}`, and every
translation of a key must contain exactly the same placeholder set as English.

```json
{
  "inventory.items.count": "{{count}} items",
  "inventory.items.title": "Items"
}
```

The contract version is independent of the application version. A new contract
version is required for breaking changes to key syntax, resource shape,
interpolation, or fallback semantics. Adding translations under the same rules
does not change the contract version.

## Resolution and fallback

Web loads the user's `preferred_locale`, stored per HC user, and defaults to
`en`. Platform strings are resolved in the selected locale, then in English,
then to the key as a visible diagnostic. For an application, Web passes both
the requested locale and an effective locale to its plugin. The effective
locale is the requested locale when declared by the application, otherwise
`en`. Applications resolve each missing key from their English resource.

Changing the language in Account settings persists the preference through
Core and updates the current Web session immediately.

## Application manifest

```json
{
  "localization": {
    "contract_version": 1,
    "default_locale": "en",
    "supported_locales": ["en", "cs"],
    "resources": [
      { "locale": "en", "path": "locales/en.json", "format": "hc-flat-json-v1" },
      { "locale": "cs", "path": "locales/cs.json", "format": "hc-flat-json-v1" }
    ]
  }
}
```

Each supported locale has exactly one resource, resource paths are unique, and
undeclared resource locales are rejected. Plugins receive `localization` in
their app context with `requested_locale`, effective `locale`,
`fallback_locale`, and the declared resource descriptors.

## Validation and failure handling

Invalid metadata, an unsupported contract version, a missing English resource,
duplicate keys, invalid keys, empty values, or placeholder mismatches fail
validation and must block publishing or installation. A non-English resource
may omit keys; this is intentionally incomplete and uses English per-key
fallback. A resource using a newer unsupported contract is outdated from the
instance's perspective and is rejected until the platform is upgraded.

Run the shared validator from the Core repository:

```shell
npm run validate:localization -- ../my-app/manifest/app-manifest.json
```

Core and Web developers add the English key first, copy its placeholders
unchanged to translations, and run build, lint, and localization tests. App
developers run the validator against the final packaged manifest so the same
resource paths and files that are shipped are checked.
