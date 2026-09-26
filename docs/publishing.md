# Publishing the Flatpak repository

## Updating Helium

**Check for Helium releases** runs daily and can be started manually. It
uses Flatpak External Data Checker to update both architecture URLs and
SHA-256 checksums in `flatpak/net.imput.helium.yml`, and the release in
`flatpak/net.imput.helium.metainfo.xml`.

Each upstream release gets a draft PR on its own branch, such as
`automation/update-helium-0.18.1.1`. An existing open PR for that version is
left alone, so fixes under review are preserved. Updates never publish
automatically.

**Validate Flatpak update** builds both architectures on native GitHub
runners. It validates shell syntax, the desktop entry, and AppStream metadata.
It does not test a graphical browser session. Website-only changes skip
package validation and do not change the build cache key.

Enable **Allow GitHub Actions to create and approve pull requests** in the
repository's Actions settings. Approve validation runs on updater-created
PRs when GitHub requests it, or start validation manually on the PR branch.
Review the upstream release and both architecture builds before merging.

## First publication

The workflows target a public `TB516/helium-flatpak` GitHub repository and
`https://helium.thomasberrios.com`. Public repositories can use the native
`ubuntu-24.04-arm` runner for aarch64 builds.

1. Set **Settings → Pages → Source** to **GitHub Actions**.
2. Configure `helium.thomasberrios.com` as the Pages custom domain, with a
   DNS CNAME pointing to `tb516.github.io`, and enable HTTPS when available.
3. Add the repository signing key as described below.
4. Run **Publish Flatpak repository** with **first-publish** selected.
5. Check the website and install link for both architectures.

## Subsequent publications

The repository at `helium.thomasberrios.com` is initialized with Helium
0.18.1.1 for both architectures.

Run **Publish Flatpak repository** manually with **first-publish** off.
Each architecture job restores and verifies its published history before
building and signing its new commit. If restoration fails, publishing stops.

Once both builds succeed, the publishing job combines their application refs
and recent history, generates AppStream metadata for both architectures,
signs the repository summary, and deploys the website and repository together
as a Pages artifact. A failed architecture build prevents deployment.

The repository retains the current release and two parent commits per
architecture. Users can inspect these with `flatpak remote-info --log` and
select a previous build with `flatpak update --commit`. Debug extension refs
are not imported into the published repository.

The templates in `flatpak/pages/` receive the signing public key during
publishing. Changing the signing key requires a separate key migration;
**first-publish** is only for a genuinely new repository.

## Signing key

Use a dedicated GPG signing key without a passphrase for this unattended
workflow. Keep an encrypted backup in Bitwarden.

In **Settings → Secrets and variables → Actions → New repository secret**,
create `FLATPAK_GPG_PRIVATE_KEY` with the complete ASCII-armored private key
as its value. No key files belong in the repository, and local package builds
do not need the signing key.
