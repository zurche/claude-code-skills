# Locale And Release Notes Reference

Use this file only when generating `app/src/main/play/release-notes/<locale>/default.txt`.

## Constraints

- Write user-facing notes, not implementation details.
- Keep each locale file under 500 characters.
- Prefer 3-5 simple bullet lines.
- Mention new features, fixes, UX improvements, performance, or security.

## Locale Mapping

Map `app/src/main/res/values*` directories to Play locales:

| Android resource dir | Play locale |
|---|---|
| `values/` | `en-US` |
| `values-de` | `de-DE` |
| `values-es` | `es-ES` |
| `values-fr` | `fr-FR` |
| `values-pt-rBR` | `pt-BR` |
| `values-hi` | `hi-IN` |
| `values-id` | `id` |
| `values-it` | `it-IT` |
| `values-ja` | `ja-JP` |
| `values-ko` | `ko-KR` |
| `values-pl` | `pl-PL` |
| `values-tr` | `tr-TR` |
| `values-zh-rCN` | `zh-CN` |
| `values-zh-rHK` | `zh-HK` |
| `values-zh-rTW` | `zh-TW` |
| `values-es-rUS` | `es-US` |
| `values-es-r419` | `es-419` |

## Output Layout

```text
app/src/main/play/release-notes/
|-- en-US/default.txt
|-- de-DE/default.txt
`-- ...
```
