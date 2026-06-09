# Nova Regression Checklist

FoxOne's `nova` branch tracks Firefox's Nova redesign, which is still in beta.
Nova ships rename churn: CSS custom-property tokens, element ids, and class
names change between beta uplifts with no notice. The robust parts of FoxOne
(the `@container` opt-in toggles, anchor positioning) do not break on a rename;
the fragile parts are the **id / class / token dependencies** listed below.

Run this checklist after every Nova beta update. It is not a full audit of the
theme: it is the short list of things Nova has already moved once and is likely
to move again. If a symptom below appears, the named dependency probably got
renamed: inspect the element in the Browser Toolbox, read the new
token/id/class, and update the matching rule (keeping the old name too, so the
theme still works on the previous channel).

## How to check

1. Open a fresh Nova profile with the current `userChrome.css`.
2. For each item, look for the **symptom**. No symptom = still fine.
3. If a symptom shows, open the Browser Toolbox (`Ctrl+Alt+Shift+I`), inspect
   the named element, and compare the **Computed** / **Rules** panel against the
   dependency FoxOne expects. The rename is usually visible as a struck-through
   or unmatched property.

## Colour tokens

| Check | Symptom if broken | FoxOne dependency |
|---|---|---|
| Toolbar / toolbox background | Bookmarks bar and sidebar drift to `#2b2a33` instead of base | `--toolbar-background-color`, `--toolbox-background-color` (Nova renamed from `--toolbar-bgcolor`); both pinned in the COLOR THEME block |
| Inactive tab hover | Grey hover background returns behind the title (issue #16) | `--tab-background-color-hover` (Nova flipped the word order from `--tab-hover-background-color`, `tab.tokens.css`); both names overridden |
| Bookmarks bar background | Bookmarks bar shows the default dark toolbar colour, not base | Nova sets the bookmarks-bar background directly instead of inheriting `--toolbar-bgcolor` |

## Tabs

| Check | Symptom if broken | FoxOne dependency |
|---|---|---|
| Active tab framing | A gradient "border" reappears around the active tab | Nova paints it via a layered `background-image` (`--tab-border-color-accent`); FoxOne overrides the active-tab background |

## URL bar

| Check | Symptom if broken | FoxOne dependency |
|---|---|---|
| URL field blending | The field stops blending into the toolbar, or shows a 1px deemphasized border | Nova rebuilt the urlbar as `<moz-urlbar>` and moved internals from id to class (`.urlbar-background`, `.urlbar-input-container`, `urlbar-searchbar.css`); both legacy id and Nova classes are reset |
| Results dropdown | The open results panel blends into the page with no separation | Nova ships the panel without border/shadow; FoxOne restores a 1px outline |
| Shield / security icon padding | The shield/trust icon padding looks off | `#urlbar-input-container` moved from id to class; FoxOne matches both `#urlbar-input-container` and `.urlbar-input-container` |
| Trust icon alignment | `#trust-icon` sits ~2px too high relative to the other urlbar icons | Nova added `#trust-icon` inside `#trust-icon-container` next to a flex `#trust-label`; nudged down 2px |
| Search-mode switcher | A violet frame appears on the engine-icon switcher | Nova's switcher is a `moz-button` whose visible button lives in the shadow DOM (`part="button"`) |
| Hide-urlbar-buttons toggle | With `--uc-hide-urlbar-buttons: 1`, the shield no longer hides | Shield is `#trust-icon-container` on Nova (pre-Nova: `#tracking-protection-icon-container`); both targeted |

## Chrome-block framing and layout

| Check | Symptom if broken | FoxOne dependency |
|---|---|---|
| Edge-to-edge chrome | Toolbox / sidebar / panels float inset with a margin and rounded corners | Nova frames every `.chrome-block` container with a 4px margin + 8px radius (`--chrome-block-radius`); `#navigator-toolbox`, `#sidebar-main` etc. carry `class="chrome-block"`; FoxOne flattens `.chrome-block` |
| Web content edge | The page is pushed off the window edge, or a 4px gap appears | Nova insets web content on `#browser` (flex hbox) via padding + a 4px flex gap |
| Content separator | Web content is pushed 1px down | Nova draws the 1px chrome-content separator on `.browserContainer` |

## Window controls / hamburger

| Check | Symptom if broken | FoxOne dependency |
|---|---|---|
| Hamburger position (non-macOS) | Hamburger overlaps or gaps from the window-control buttons | `anchor-name: --uc-winbtns` on `.titlebar-buttonbox-container`, `right: anchor(...)` on `#PanelUI-button`; falls back to `--uc-window-buttons-width` if anchor positioning regresses (issue #9) |
| macOS traffic lights | Traffic lights misaligned, or content lacks left inset | `.titlebar-buttonbox-container` positioned absolute left; `#nav-bar` left padding (`--uc-traffic-light-width`); both under `@media (-moz-platform: macos)` |

## When a rename is found

1. Inspect, read the new name, update the rule, keep the old name as well.
2. If a whole feature breaks (not just a token), treat it as a real bug and use
   the systematic-debugging approach: confirm the root cause before patching.
3. Note the rename in the relevant inline comment so the next uplift is faster.
