# WebDevBar fork notes

This is WebDevBar's maintained fork of [daydve/FancyTasksNG](https://github.com/daydve/FancyTasksNG).

**Why a fork:** local customizations to the installed plasmoid get silently wiped when the
widget updates via Discover / "Get New Widgets" or is reinstalled. The fork keeps our changes
under version control and lets us pull upstream updates deliberately.

## Branch

- `master` tracks upstream (`daydve/FancyTasksNG`).
- `webdevbar` carries our customizations on top of `master`. **This is the branch we install from.**

## Distinct plugin id

`package/metadata.json` `Id` is rebranded `io.github.daydve.fancytasksng` -> `com.webdevbar.fancytasksng`
and `Name` -> `Fancy Tasks NG (WebDevBar)`, so it installs as a separate widget and the upstream
one can be uninstalled.

Internal DBus + i18n strings (`io.github.daydve.fancytasksng.Bridge`, the `plasma_applet_...`
translation domain, etc.) are **intentionally left as upstream's** to minimise merge conflicts:
they are private to the package and consistent within it, and the upstream widget is uninstalled
so there is no DBus name clash. i18n falls back to the English source strings, which is fine.

## Customizations

- **`package/contents/ui/Badge.qml` - readable unread badge text.** The badge background is red
  (`negativeTextColor`) whenever shown, but the count/icon/symbol text was keyed off `isUrgent`
  (only "new/unseen" mail), so once mail was no longer new the text fell back to
  `Kirigami.Theme.textColor` (dark) -> **black on red, unreadable**. The Qt 6.11 / Plasma 6.7
  update made this worse (theme colors resolve late, causing a black-then-white flicker). Fix:
  force the on-red text to literal `"white"` (`showBackground ? "white" : Kirigami.Theme.textColor`)
  at the three color bindings (textIconColor ~L46, icon ~L86, count label ~L239). Literal white
  also kills the first-paint flicker.

## Updating from upstream

```bash
git fetch upstream
git checkout webdevbar
git merge upstream/master        # or: git rebase upstream/master
# resolve: keep our metadata.json Id/Name; re-check Badge.qml if upstream touched it
```

## Install / reinstall after edits

```bash
# first install
kpackagetool6 --type Plasma/Applet --install package
# upgrade in place after edits
kpackagetool6 --type Plasma/Applet --upgrade package
# then reload
systemctl --user restart plasma-plasmashell.service
```
