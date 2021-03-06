# Releasing Git LFS

The core team of Git LFS maintainers publishes releases on a cadence of their
determining.

## Release Naming

We follow Semantic Versioning standards as follows:

  * `MAJOR` releases are done on a scale of 1-2 years. These encompass breaking,
    incompatible API changes, or command-line interface changes that would
    cause existing programs or use-cases scripted against Git LFS to break.

  * `MINOR` releases are done on a scale of 1-2 months. These encompass new
    features, bug fixes, and other "medium"-sized changes into a semi-regular
    release schedule.

  * `PATCH` releases are done on the scale of 1-2 weeks. These encompass
    critical bug fixes, but lack new features. They are amended to a `MINOR`
    release "series", or, if serious enough (e.g., security vulnerabilities,
    etc.) are backported to previous versions.

## Release Artifacts

We package several artifacts for each tagged release. They are:

  1. `git-lfs-@{os}-v@{release}-@{arch}.tar.gz` for the following values:

      |     | operating system | architecture |
      | --- | ---------------- | ------------ |
      | git-lfs-darwin-386-v@{version}.tar.gz | darwin | 386 |
      | git-lfs-darwin-amd64-v@{version}.tar.gz | darwin | amd64 |
      | git-lfs-freebsd-386-v@{version}.tar.gz | freebsd | 386 |
      | git-lfs-freebsd-amd64-v@{version}.tar.gz | freebsd | amd64 |
      | git-lfs-linux-386-v@{version}.tar.gz | linux (generic) | 386 |
      | git-lfs-linux-amd64-v@{version}.tar.gz | linux (generic) | amd64 |

  2. `git-lfs-windows-v@{release}-@{arch}.zip` for the following values:

      |     | operating system | architecture |
      | --- | ---------------- | ------------ |
      | git-lfs-windows-386-v@{version}.zip | windows | 386 |
      | git-lfs-windows-amd64-v@{version}.zip | windows | amd64 |

  3. `git-lfs-windows-v@{release}.exe`, a signed Windows installer that contains
     copies of both `-x86` and `-x64` copies of Git LFS.

  4. `*.deb`, and `*.rpm` packages for all of the distributions named in
     `script/packagegcloud.rb`.

## Development Philosophy

We do all major development on the `master` branch, and assume it to be passing
tests at all times. New features are added via the feature-branch workflow, or
(optionally) from a contributor's fork.

This is done so that `master` can progress and grow new features, while
historical releases, such as `v2.n.0` can receive bug fixes as they are applied
to master, eventually culminating in a `v2.n.1` (and so on) release.

## Building a release

Let release `v2.n.m` denote the version that we are _releasing_. When `m` is
equal to 0, we say that we are releasing a MINOR version of Git LFS, in the
`v2.n`-series. Let `v2.n-1` denote the previous release series.

  1. First, we write the release notes and do the housekeeping required to
     indicate a new version. On a new branch, called `release-next`, do the
     following:

     * Run `script/changelog v2.n-1.0...HEAD` and categorize each merge commit
       as a feature, bug-fix, miscellaneous change, or skipped. Ensure that your
       `~/.netrc` credentials are kept up-to-date in order to make requests to
       the GitHub API.

       This will write a portion of the CHANGELOG to stdout, which you should
       copy and paste into `CHANGELOG.md`, along with an H2-level heading
       containing the version and release date (consistent with the existing
       style in the document.)

       * Optionally write 1-2 paragraphs summarizing the release, and calling out
         community contributions.

       * If you are releasing a MINOR version, and not a PATCH, and if there
         were non-zero PATCH versions released in the `v2.n-1` series, also
         include any changes from the latest CHANGELOG in that series, too.

     * Update `config/version.go` to refer to the latest version.

     * Update `debian/changelog` to refer to the latest version (optionally
       include all releases from the latest in `v2.n-1`'s, too).

     * Update `rpm/SPECS/git-lfs.spec` to refer to the latest version.

     * Update `versioninfo.json` to refer to the latest version.

  2. Then, create a pull request of your changes with head `release-next`. If
     you're building a MAJOR or MINOR release, set the base to `master`.
     Otherwise, set the base to `release-2.n`.

     Run Continuous Integration, and ensure that it passes.
     Notify the `@git-lfs/release` team, a collection of humans who are
     interested in Git LFS releases.

  3. Once approved and verified, merge the pull request you created in the
     previous step. Locally, create a GPG-signed tag on the merge commit called
     `v2.n.m`:

     ```ShellSession
     $ git show -q HEAD
     commit 9377560199b9d7cd2d3c38524a2a7f61aedc89db
     Merge: 3f3faa90 a55b7fd9
     Author: Taylor Blau <ttaylorr@github.com>
     Date:   Thu Jul 26 14:48:39 2018 -0500

         Merge pull request #3150 from git-lfs/release-next

             release: v2.n.0
     $ git tag -s v2.n.m
     $ git describe HEAD
     v2.n.m
     ```

  4. Begin building release artifacts.

     * To build `*.tar.gz` artifacts, run `make release`.

     * To build `*.zip` artifacts, run `make release` (as above), and follow the
       additional instructions in the section ["For Windows"](#for-windows)
       below.

     * To build `*.deb` and `*.rpm` artifacts, follow the additional
       instructions in ["For Linux"](#for-linux) below.

  5. Create a GitHub release for the new version number, including the
     changelog and the standard copy materials, as below:

     ```
     <CHANGELOG.md additions>

     ## Packages

     Up to date packages are available on
     [PackageCloud](https://packagecloud.io/github/git-lfs) and
     [Homebrew](http://brew.sh/).

     [RPM RHEL 6/CentOS 6](https://packagecloud.io/github/git-lfs/packages/el/6/...)
     [RPM RHEL 7/CentOS 7](https://packagecloud.io/github/git-lfs/packages/el/7/...)
     [Debian 7](https://packagecloud.io/github/git-lfs/packages/debian/wheezy/...)
     [Debian 8](https://packagecloud.io/github/git-lfs/packages/debian/jessie/...)
     [Debian 9](https://packagecloud.io/github/git-lfs/packages/debian/stretch/...)

     ## SHA-256 hashes:

     **</path/to/artifact>**
     <artifact SHA-256>
     ```

  6. Upload the `*.tar.gz`, `*.zip` and Windows Installer assets to the release
     notes, and rename them according to the following list:

     * Darwin 386
     * Darwin AMD64
     * FreeBSD 386
     * FreeBSD AMD64
     * Linux 386
     * Linux AMD64
     * Windows Installer
     * Windows 386
     * Windows AMD64

     GitHub does not allow you (at the time of writing) to edit the name of a
     release asset from the website interface. Instead, list all release
     artifacts by:

     ```ShellSession
     $ curl -n https://api.github.com/repos/git-lfs/git-lfs/releases |
       jq '.[0].assets | .[] | {"id","name","label"} '
     ```

     And update a single release asset's name by running:

     ```ShellSession
     $ curl -XPATCH -in \
       https://api.github.com/repos/git-lfs/git-lfs/releases/assets/8712424 \
       -d '{
         "name": "git-lfs-linux-amd64-v2.5.2.tar.gz",
         "label": "Linux AMD64"
       }'
     ```

     Make sure to also include the SHA-256 signatures of all of the artifact
     hashes, by running:

     ```ShellSession
     $ pwd
     /go/src/github.com/git-lfs/git-lfs/bin/releases
     $ shasum -a256 * | awk '{ print "**" $2 "**\n" $1 "\n" }'
     ```

  7. Push the tag, via:

     ```ShellSession
     $ git push origin v2.n.m
     ```

     And publish the release on GitHub.

  8. Move any remaining items out of the milestone for the current release to a
     future release and close the milestone.

  9. Update the `_config.yml` file in
     [`git-lfs/git-lfs.github.com`](https://github.com/git-lfs/git-lfs.github.com),
     similar to the following:

     ```diff
     diff --git a/_config.yml b/_config.yml
     index 03f23d8..6767f6f 100644
     --- a/_config.yml
     +++ b/_config.yml
     @@ -1,7 +1,7 @@
      # Site settings
      title: "Git Large File Storage"
      description: "Git Large File Storage (LFS) replaces large files such as audio samples, videos, datasets, and graphics with text pointers inside Git, while storing the file contents on a remote server like GitHub.com or GitHub Enterprise."
     -git-lfs-release: 2.5.1
     +git-lfs-release: 2.5.2

      url: "https://git-lfs.github.com"
     ```

     Then update [our fork](https://github.com/git-lfs/Homebrew-core) of
     `Homebrew/homebrew-core` similar to
     [Homebrew/homebrew-core#32161](https://github.com/Homebrew/homebrew-core/pull/32161),
     then celebrate.

### Building `v2.n.0` (MINOR versions)

When building a MINOR release, we introduce a new `release-2.n` branch which
will receive all new features and bug fixes since `release-2.n-1`. The change
set described by `v2.n-1.0` and `v2.n.0` is as reported by `git log
v2.n-1.0...master` at the time of release.

  1. To introduce this new branch (after creating and merging `release-next`
     into `master`), simply run:

     ```ShellSession
     $ git branch
     * master
     $ git checkout -b release-2.n
     ```

  2. Then, proceed to follow the guidelines above.

### Building `v2.n.m` (PATCH versions)

When building a PATCH release, follow the same process as above, with the
additional caveat that we must cherry-pick merges from master to the release
branch.

  1. To begin, checkout the branch `release-2.n`, and ensure that you have the
     latest changes from the remote.

  2. Gather a set of potential candidates to "backport" to the older release
     with:

     ```ShellSession
     $ git log --merges --first-parent v2.n.m-1...master
     ```

   3. For each merge that you want to backport, run:

      ```ShellSession
      $ git cherry-pick -m1 <SHA-1>
      ```

      To cherry-pick the merge onto your release branch, adoption the first
      parent as the mainline.

   4. Then, proceed to follow the guidelines above.

### For Windows

We distribute three key artifacts for Windows builds, the `-x86`, and `-x64`
binaries embedded in a ZIP file, and the Windows installer. Before beginning,
ensure that you have a Windows VM (or access to a Windows computer) with the
following installed:

  1. Git for Windows
  2. Go (matching the version the release is built with)
  3. InnoSetup (available from: http://www.jrsoftware.org/isinfo.php)
  4. The Git LFS signing certificate (ask a maintainer for access)
  5. signtool.exe (available from:
     https://docs.microsoft.com/en-us/windows/desktop/seccrypto/signtool)

Once you have the above installed, boot up your Windows VM and run the
following in a checkout of Git LFS:

  1. First, build the unsigned x64 and x86 versions of the Git LFS binary:

     ```ShellSession
     $ make -B GOARCH=amd64 && cp ./bin/git-lfs.exe ./git-lfs-x64.exe
     $ make -B GOARCH=386 && cp ./bin/git-lfs.exe ./git-lfs-x86.exe
     ```

  2. Then, sign each using the following command:

     ```PowerShell
     $ signtool.exe sign /sha1 CERT_SHA1 /fd sha256 /tr http://timestamp.digicert.com /td sha256 /v git-lfs-x64.exe
     $ signtool.exe sign /sha1 CERT_SHA1 /fd sha256 /tr http://timestamp.digicert.com /td sha256 /v git-lfs-x86.exe
     ```

  3. Then, launch InnoSetup and open the file
     `script/windows-installer/inno-setup-git-lfs-installer.iss`, and build the
     Git LFS installer.

  4. Sign the Git LFS installer with the same command as above.

  5. Re-package the `git-lfs-x64.exe` and `git-lfs-x86.exe` artifacts as
     `git-lfs.exe` inside of the Windows ZIP artifacts that you built with `make
     release`, and upload them to the release page.

  6. Upload the signed `git-lfs-windows-v2.n.m.exe` installer to the release
     page.

### For Linux

To build Git LFS `*.deb` and `*.rpm` packages, we first build them in
specialized Docker containers, and we upload them to PackageCloud.io.

  1. To build custom versions of the Docker containers, clone
     `git@github.com:git-lfs/build-dockers.git`, and run `build_dockers.bsh`.
     Then, run `./docker/run_dockers.bsh` in the Git LFS repo with
     `DOCKER_AUTOPULL=0`.

     Otherwise, run `./docker/run_dockers.bsh` in the Git LFS repo with no
     arguments.

  2. Once you have built the `*.deb` and `*.rpm` files, they will appear in the
     `repos` directory, and you can upload them to PackageCloud with:

     ```ShellSession
     $ PACKAGECLOUD_TOKEN= ruby ./script/packagecloud.rb
     ```
