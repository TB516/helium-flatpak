# Building the Flatpak

The manifest repackages official, checksum-pinned Helium tarballs for x86_64
and aarch64. It preserves the upstream desktop translations and actions,
changing the executable and icon references to match this package.

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
that host path using `flatpak-spawn --host`. That drops the caller's
environment, so it forwards `XDG_ACTIVATION_TOKEN` when an app launcher sets
it. Helium uses the token to end the desktop's loading cursor and focus
its window.

`helium-flatpak` passes `--user-data-dir` and `--disk-cache-dir` so Helium's
profile and cache stay in the Flatpak data directory, and sets
`BREAKPAD_DUMP_LOCATION` so crash reports stay in the profile. Otherwise Helium keeps
the host's XDG directories like a native install, so desktop settings, default
apps, and web app shortcuts reach the host.

`helium-host` sets `HELIUM_DESKTOP`, which identifies the exported desktop
entry, and the same environment as the upstream `helium-wrapper`, then starts
Helium with the original arguments. `CHROME_WRAPPER` points at Flatpak's
exported launcher instead of the wrapper, so web app shortcuts and relaunches
keep working after updates. There is no runtime copy of the payload and no
cleanup process.

Helium runs xdg-utils commands on the host, which some hosts do not provide.
The manifest bundles the ones Helium uses under `/app/libexec/xdg-utils`, and
`helium-host` appends that directory to `PATH` so host copies take precedence.

## Manual checks

After installing a build, check launch from the app menu, the new-window and
incognito actions, and opening a URL. Check the profile path at
`chrome://version` and sandbox status at `chrome://sandbox`. Try a download,
a file picker, and Wayland screen sharing on the target desktop. Install a web
app and check that it appears in the app menu and that its window matches it.
Set Helium as the default browser from its settings.

These checks cover behavior that a successful package build cannot verify.
