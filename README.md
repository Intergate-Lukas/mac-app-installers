# mac-app-installers

Small install packages for Macs managed through Apple Business.

Apple Business can't store files or run commands directly. It can send a Mac a package from a web link, and a package can carry a script that the Mac runs while installing. So each package here carries only a script. The script downloads the real app from its maker, checks it, and installs it. Only the small package is stored here, never the apps.

## Packages

| Package | What it installs | Checks before installing |
|---|---|---|
| `Install-Superwhisper-<version>.pkg` | Superwhisper 3.2.2, plus Google Chrome if the Mac doesn't have it | Superwhisper: the file's SHA-256 must match the one Homebrew pins, and it must be signed by SuperUltra, Inc. Chrome: Google's installer must carry Google's Developer ID signature. |

Anything already installed is skipped. The script writes what it did to `/var/log/intergate-app-install.log` on the Mac.

## How a package is made

Pushing a tag like `v1.0.0` runs `.github/workflows/build.yml` on one of GitHub's Macs:

1. Builds the package with `pkgbuild` and `productbuild`.
2. Prints Superwhisper's code requirement (used by the Accessibility profile) and Chrome's signer.
3. Installs the package on that Mac with Chrome and Superwhisper removed first, checks both apps arrived, then installs it again to check the skip path.
4. Only if all of that passes, attaches the package to a GitHub release and prints its SHA-256.

## Adding a package in Apple Business

Devices → macOS Packages → add:

- **URL**: the release asset link, for example `https://github.com/Intergate-Lukas/mac-app-installers/releases/download/v1.0.0/Install-Superwhisper-1.0.0.pkg`
- **Hash**: the SHA-256 printed by the build (also on the release page)
- **Bundle ID**: `com.superduper.superwhisper`

Each version has its own link and hash. To change what a package installs, make a new tag and add the new package. Never replace a file on an existing release.

## Accessibility profile

`superwhisper/profile/Superwhisper-Accessibility.mobileconfig` pre-approves Superwhisper's Accessibility permission, which it needs to type into other apps. macOS doesn't let any profile pre-approve the microphone, so the person clicks Allow the first time.
