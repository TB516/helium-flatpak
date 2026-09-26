# Helium Flatpak

An unofficial Flatpak package for [Helium](https://helium.computer/) on
x86_64 and aarch64 Linux. Flatpak handles installation, updates, and the app
menu entry. Helium runs on your host system using the official Linux tarball.

## Install

Install it with your software store using the
[Helium install link](https://helium.thomasberrios.com/net.imput.helium.flatpakref),
or from a terminal:

```sh
flatpak install --system https://helium.thomasberrios.com/net.imput.helium.flatpakref
```

Accept the prompt to add the Helium repository to receive updates through
your software store or `flatpak update --system`. Launch Helium from your app
menu or with `flatpak run net.imput.helium`.

## Host integration

Helium has the same access to your system as a native install. This package
does not disable Chromium's sandbox. It uses host libraries, so compatibility
depends on your distribution.

Your browser profile and other XDG data live under
`~/.var/app/net.imput.helium/`, separate from a native Helium profile.
The launcher links your host's `user-dirs.dirs` file when available, keeping
Downloads and other user folders at their configured locations.

Uninstall with `flatpak uninstall net.imput.helium`. Your browser data
is kept unless you also request its deletion.

Maintainers can find update and release instructions in
[Publishing](docs/publishing.md). For local builds, see
[Building](docs/building.md).
