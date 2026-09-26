# Building the Flatpak

The manifest repackages official, checksum-pinned Helium tarballs for x86_64
and aarch64. It preserves the upstream desktop translations and actions,
changing the executable and icon references to match this package. It disables
startup notification because the browser runs on the host, outside Flatpak's
launch tracking.

The host needs Flatpak, Flatpak Builder, and the per-user
`org.freedesktop.Platform//25.08` and `org.freedesktop.Sdk//25.08` for its
architecture. The build script does not install or update those dependencies.

```sh
./scripts/build-flatpak build
```

This builds for the host architecture and writes
`build-flatpak/net.imput.helium.flatpak`. Builds validate the shell
syntax and desktop entry, and compose AppStream metadata. They do not launch
the browser or install the package.

To build and install for your user, then launch:

```sh
./scripts/build-flatpak install
./scripts/build-flatpak run
```

The `run` command forwards additional arguments to Helium. Build output and
cache stay in `build-flatpak/`.

## Launcher design

`flatpak/package/helium-flatpak` reads the running instance's `app-path` from
`/.flatpak-info`. This identifies the exact files mounted at `/app`, including
when multiple branches or installations exist. It starts `helium-host` at
that host path using `flatpak-spawn --host`.

`helium-host` keeps the host environment except for the four XDG base
directories and `HELIUM_DESKTOP`, which identifies the exported desktop entry.
It links the host user-folder settings when the app has none, then executes
the upstream wrapper with the original arguments. There is no runtime copy
of the payload and no cleanup process.

## Manual checks

After installing a build, check launch from the app menu, the new-window and
incognito actions, and opening a URL. Check the profile path at
`chrome://version` and sandbox status at `chrome://sandbox`. Try a download,
a file picker, and Wayland screen sharing on the target desktop.

These checks cover behavior that a successful package build cannot verify.
