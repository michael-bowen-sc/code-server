# ci

This directory contains scripts used for code-server's continuous integration infrastructure.

Some of these scripts contain more detailed documentation and options
in header comments.

Any file or directory in this subdirectory should be documented here.

- [./ci/lib.sh](./lib.sh)
  - Contains code duplicated across these scripts.

## Publishing a release

Make sure you have `$GITHUB_TOKEN` set and [hub](https://github.com/github/hub) installed.

1. Update the version of code-server and make a PR.
   1. Update in `package.json`
   2. [README.md](../README.md) and [guide.md](../doc/guide.md) install examples
   3. [install.sh](../install.sh)
2. GitHub actions will generate the `npm-package`, `release-packages` and `release-images` artifacts.
3. Run `yarn release:github-draft` to create a GitHub draft release from the template with
   the updated version.
   1. Summarize the major changes in the release notes and link to the relevant issues.
4. Wait for the artifacts in step 2 to build.
5. Run `yarn release:github-assets` to download the `release-packages` artifact and then
   upload them to the draft release.
6. Run some basic sanity tests on one of the released packages.
7. Make sure the github release tag is the commit with the artifacts.
8. Publish the release and merge the PR.
   1. CI will automatically grab the artifacts and then:
      1. Publish the NPM package from `npm-package`.
      2. Publish the Docker Hub image from `release-images`.
9. Update the homebrew and AUR packages.

## dev

This directory contains scripts used for the development of code-server.

- [./ci/dev/container](./dev/container)
  - See [./doc/CONTRIBUTING.md](../doc/CONTRIBUTING.md) for docs on the development container.
- [./ci/dev/fmt.sh](./dev/fmt.sh) (`yarn fmt`)
  - Runs formatters.
- [./ci/dev/lint.sh](./dev/lint.sh) (`yarn lint`)
  - Runs linters.
- [./ci/dev/test.sh](./dev/test.sh) (`yarn test`)
  - Runs tests.
- [./ci/dev/ci.sh](./dev/ci.sh) (`yarn ci`)
  - Runs `yarn fmt`, `yarn lint` and `yarn test`.
- [./ci/dev/vscode.sh](./dev/vscode.sh) (`yarn vscode`)
  - Ensures [./lib/vscode](../lib/vscode) is cloned, patched and dependencies are installed.
- [./ci/dev/patch-vscode.sh](./dev/patch-vscode.sh) (`yarn vscode:patch`)
  - Applies [./ci/dev/vscode.patch](./dev/vscode.patch) to [./lib/vscode](../lib/vscode).
- [./ci/dev/diff-vscode.sh](./dev/diff-vscode.sh) (`yarn vscode:diff`)
  - Diffs [./lib/vscode](../lib/vscode) into [./ci/dev/vscode.patch](./dev/vscode.patch).
- [./ci/dev/vscode.patch](./dev/vscode.patch)
  - Our patch of VS Code, see [./doc/CONTRIBUTING.md](../doc/CONTRIBUTING.md#vs-code-patch).
  - Generate it with `yarn vscode:diff` and apply with `yarn vscode:patch`.
- [./ci/dev/watch.ts](./dev/watch.ts) (`yarn watch`)
  - Starts a process to build and launch code-server and restart on any code changes.
  - Example usage in [./doc/CONTRIBUTING.md](../doc/CONTRIBUTING.md).

## build

This directory contains the scripts used to build and release code-server.
You can disable minification by setting `MINIFY=`.

- [./ci/build/build-code-server.sh](./build/build-code-server.sh) (`yarn build`)
  - Builds code-server into `./out` and bundles the frontend into `./dist`.
- [./ci/build/build-vscode.sh](./build/build-vscode.sh) (`yarn build:vscode`)
  - Builds vscode into `./lib/vscode/out-vscode`.
- [./ci/build/build-release.sh](./build/build-release.sh) (`yarn release`)
  - Bundles the output of the above two scripts into a single node module at `./release`.
- [./ci/build/build-static-release.sh](./build/build-static-release.sh) (`yarn release:static`)
  - Requires a node module already built into `./release` with the above script.
  - Will build a static release with node and node_modules bundled into `./release-static`.
- [./ci/build/clean.sh](./build/clean.sh) (`yarn clean`)
  - Removes all build artifacts.
  - Will also `git reset --hard lib/vscode`.
  - Useful to do a clean build.
- [./ci/build/code-server.sh](./build/code-server.sh)
  - Copied into static releases to run code-server with the bundled node binary.
- [./ci/build/test-static-release.sh](./build/test-static-release.sh) (`yarn test:static-release`)
  - Ensures code-server in the `./release-static` directory works by installing an extension.
- [./ci/build/build-packages.sh](./build/build-packages.sh) (`yarn package`)
  - Packages `./release-static` into a `.tar.gz` archive in `./release-packages`.
  - If on linux, [nfpm](https://github.com/goreleaser/nfpm) is used to generate `.deb` and `.rpm`.
- [./ci/build/nfpm.yaml](./build/nfpm.yaml)
  - Used to configure [nfpm](https://github.com/goreleaser/nfpm) to generate `.deb` and `.rpm`.
- [./ci/build/code-server-nfpm.sh](./build/code-server-nfpm.sh)
  - Entrypoint script for code-server for `.deb` and .rpm`.
- [./ci/build/code-server.service](./build/code-server.service)
  - systemd user service packaged into the `.deb` and `.rpm`.
- [./ci/build/release-github-draft.sh](./build/release-github-draft.sh) (`yarn release:github-draft`)
  - Uses [hub](https://github.com/github/hub) to create a draft release with a template description.
- [./ci/build/release-github-assets.sh](./build/release-github-assets.sh) (`yarn release:github-assets`)
  - Downloads the release-package artifacts for the current commit from CI.
  - Uses [hub](https://github.com/github/hub) to upload the artifacts to the release
    specified in `package.json`.
- [./ci/build/npm-postinstall.sh](./build/npm-postinstall.sh)
  - Post install script for the npm package.
  - Bundled by`yarn release`.

## release-container

This directory contains the release docker container.

- [./release-container/build.sh](./release-container/build.sh)
  - Builds the release container with the tag `codercom/code-server-$ARCH:$VERSION`.
  - Assumes debian releases are ready in `./release-packages`.

## container

This directory contains the container for CI.

## steps

This directory contains the scripts used in CI.
Helps avoid clobbering the CI configuration.

- [./steps/fmt.sh](./steps/fmt.sh)
  - Runs `yarn fmt` after ensuring VS Code is patched.
- [./steps/lint.sh](./steps/lint.sh)
  - Runs `yarn lint` after ensuring VS Code is patched.
- [./steps/test.sh](./steps/test.sh)
  - Runs `yarn test` after ensuring VS Code is patched.
- [./steps/release.sh](./steps/release.sh)
  - Runs the release process.
  - Generates the npm package at `./release`.
- [./steps/release-static.sh](./steps/release-static.sh)
  - Takes the output of the previous script and generates a static release and
    release packages into `release-packages`.
- [./steps/publish-npm.sh](./steps/publish-npm.sh)
  - Grabs the `npm-package` release artifact for the current commit and publishes it on npm.
- [./steps/build-docker-image.sh](./steps/build-docker-image.sh)
  - Builds the docker image and then saves it into `./release-images/code-server-$ARCH-$VERSION.tar`.
- [./steps/push-docker-manifest.sh](./steps/push-docker-manifest.sh)
  - Loads all images in `./release-images` and then builds and pushes a multi architecture
    docker manifest for the amd64 and arm64 images to `codercom/code-server:$VERSION` and
    `codercom/code-server:latest`.
