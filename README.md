# Omarchy bar hover-icon theming

Private notes and a local workaround for one Omarchy 4.x gap: the
status-bar hover icons (dictation, screen recording, reminder, night
light, DND, stay awake) have no theme key.

This repository is **not** A Stranger Theme and is **not** part of the
omarchy.org listing. The listing theme stays at `a-stranger-theme/`.

## Current behaviour (Omarchy 4.0.4 and current master)

`[bar]` in `shell.toml` only has `background`, `text`, and `active`.
Inactive hover icons use bar `text` at a hardcoded **0.45** opacity
(`dimmed: !effectiveActive` in `BarIndicator.qml`). Extra keys in a
theme's `shell.bar.toml` are ignored. `omarchy theme set` cannot install
shell plugins.

A plugin clone of `omarchy.indicators` can restyle the icons, but that
clone is **machine-global**. It keeps winning after the user switches
to another theme (including light themes).

## Intended upstream fix

Add `[bar]` keys that default to today's look:

```toml
[bar]
text            = "{{ foreground }}"
active          = "{{ red }}"
inactive        = "{{ foreground }}"
inactive-alpha  = 0.45
```

Wire `Color.bar.inactive` the same way as other `composed()` tokens, and
have `BarIndicator.qml` use that instead of the hardcoded fade. Existing
themes stay identical. A theme that wants steel-blue hover icons at full
strength then ships:

```toml
inactive       = "#7399b4"
inactive-alpha = 1.0
```

`omarchy theme set` would swap those with the theme. No plugin clone.

Omarchy wants feature ideas in Discussions → Suggestions first; issues
are for verified bugs. Work in a fork of Omarchy, never `/usr/share/omarchy/`.

Nearby work that is **different**: omacom/omarchy#8177 and #11659
(loading third-party indicators), not inactive color.

## Local workaround

See `bar-hover-icons.md`. Read the warning at the top before cloning
anything. Undo with `omarchy plugin remove` before switching themes.
