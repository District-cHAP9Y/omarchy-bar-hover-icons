# Bar hover icons (local workaround)

**Lives in this private repo, not in the published theme.**

Cloning `omarchy.indicators` is machine-global. The restyle keeps
winning after you switch themes (including light themes). Undo before
changing themes:

```bash
omarchy plugin remove <your-username>.indicators --yes
```

Omarchy 4.x has no theme key for the status-bar hover icons (screen
recording, dictate, night light, DND, stay awake, reminders). Those
icons sit just left of the clock. Stock Omarchy paints them as **bar
text at 45% opacity**, so on A Stranger Theme they look like faded red.

The screenshot used steel blue **`#7399b4`** at full strength. That
color cannot ship inside a theme. Each user applies it once on their
machine.

## Why this is not automatic

`shell.bar.toml` only has `background`, `text`, and `active`. Inactive
hover icons always use `text` at a hardcoded 0.45 fade. Extra keys in
the theme are ignored. `omarchy theme set` also cannot install shell
plugins.

Do **not** edit anything under `/usr/share/omarchy/`. Updates would
overwrite it.

## Steps

### 1. Clone the built-in indicators widget

```bash
omarchy plugin clone omarchy.indicators
```

This copies the widget to
`~/.config/omarchy/plugins/<your-username>.indicators/` and switches
the bar to that copy. It survives Omarchy updates.

### 2. Edit `Indicators.qml`

Open:

```bash
~/.config/omarchy/plugins/<your-username>.indicators/Indicators.qml
```

Find the `IndicatorLoader` component’s `Loader { ... }` (search for
`onLoaded`). Replace that `Loader` block, the `Connections` under it,
and `function injectProps()` with the following.

Keep the `function syncActiveState()` that already follows
`injectProps` — do not delete it.

```qml
    Loader {
      id: indicatorSource

      anchors.fill: parent
      source: indicatorSlot.indicatorId ? Qt.resolvedUrl("indicators/" + indicatorSlot.indicatorId + ".qml") : ""
      onLoaded: {
        indicatorSlot.injectProps()
        indicatorSlot.syncActiveState()
        indicatorSlot.applyStrangerStyle()
      }
      onStatusChanged: if (status === Loader.Error) console.warn("Indicator loader error", indicatorSlot.indicatorId, source)
    }

    Connections {
      target: indicatorSource.item
      ignoreUnknownSignals: true
      function onActiveChanged() {
        indicatorSlot.syncActiveState()
        Qt.callLater(indicatorSlot.applyStrangerStyle)
      }
      function onEffectiveActiveChanged() { Qt.callLater(indicatorSlot.applyStrangerStyle) }
      function onInactiveRevealedChanged() { Qt.callLater(indicatorSlot.applyStrangerStyle) }
      function onBelongsInBlockChanged() { Qt.callLater(indicatorSlot.applyStrangerStyle) }
      function onOpacityChanged() { Qt.callLater(indicatorSlot.applyStrangerStyle) }
    }

    function injectProps() {
      var target = indicatorSource.item
      if (!target) return
      if ("bar" in target) target.bar = root.bar
      if ("moduleName" in target) target.moduleName = indicatorId
      if ("settings" in target) target.settings = indicatorSettings
      if ("indicatorBlock" in target) target.indicatorBlock = indicatorBlock
      if ("indicatorHost" in target) target.indicatorHost = root
      if ("activeOverride" in target) target.activeOverride = indicatorBlock === "active" ? true : null
      applyStrangerStyle()
    }

    function applyStrangerStyle() {
      var target = indicatorSource.item
      if (!target) return

      target.dimmed = false

      var active = target.effectiveActive === true
      var nextFg = active ? (root.bar ? root.bar.barForeground : "#eb0000") : "#7399b4"
      if (String(target.foreground) !== String(nextFg))
        target.foreground = nextFg

      var nextOp = (target.belongsInBlock && (active || target.inactiveRevealed)) ? 1 : 0
      if (Math.abs(target.opacity - nextOp) > 0.01)
        target.opacity = nextOp
    }
```

Leave the rest of the file alone. Do not add a separate
`StrangerIndicator.qml` — that approach does not load.

### 3. Restart the shell

Saving under `~/.config/omarchy/plugins/` usually hot-reloads. If the
icons do not change:

```bash
omarchy restart shell
```

### 4. Check

Hover just left of the clock. Off/hover icons should be steel blue
`#7399b4`. Icons that are actually on (recording, night light enabled,
etc.) stay bar red `#eb0000`.

## Undo

```bash
omarchy plugin remove <your-username>.indicators --yes
```

The bar returns to stock `omarchy.indicators`.
