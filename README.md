# fawkes-images

Workflows that build and publish the images and artifacts of
[`platform9/fawkes`](https://github.com/platform9/fawkes), the PCD installer
and upgrade operator, to `ghcr.io/dmiller-pf9` for testing. This repository
holds no Dockerfile, build context or chart: each workflow checks out one ref
of `platform9/fawkes` and builds exactly what that ref contains, with that
repository's own `make` targets.

Every workflow is manual (`workflow_dispatch`) and takes one input, `ref`,
the `platform9/fawkes` ref to build (default `main`).

| Workflow | Builds | Pushes |
|---|---|---|
| `images.yml` | the operator image (`Dockerfile`) and the Job image (`Dockerfile.job`, checked with `build/job/check-image.sh`) | `ghcr.io/dmiller-pf9/fawkes:<sha>` and `ghcr.io/dmiller-pf9/fawkes-job:<sha>`; also `latest` when `ref` is `main`, and `<ref>` when it is a tag |
| `release.yml` | the PCD Community Edition release artifact and its charts (`make test-ce`, then `make ce-publish`) | `ghcr.io/dmiller-pf9/pcd-release:<releases/ce/VERSION>` and `ghcr.io/dmiller-pf9/charts/<name>:<version>` for every chart |

## Running one

```
gh workflow run images.yml -R dmiller-pf9/fawkes-images -f ref=main
gh workflow run release.yml -R dmiller-pf9/fawkes-images -f ref=main
gh run watch -R dmiller-pf9/fawkes-images
```

## Secrets

- `GITHUB_TOKEN` (implicit): pushes to `ghcr.io` under this repository's
  owner. The job asks for `packages: write` and `contents: read` only.
- `FAWKES_REPO_READ` (maintained by hand): a token with read access to the
  private `platform9/fawkes`, used only by `actions/checkout`.

## Packages

A package is private on its first push. Make `fawkes` and `fawkes-job`
public once, in the GitHub UI (the package's settings, "Change visibility"),
so that the test cluster pulls them without a pull secret. The registry is
temporary: when the images move to their permanent registry, the workflows
and the artifact builder's registry base move with them.

`pcd-release` and every `charts/*` package may need the same one-time step:
`make ce-publish` ends by fetching each pushed reference anonymously and
fails, naming the package, if one is private. A package pushed by this
repository's own workflow token is expected to come out public, as the
images did; a new chart name pushed from a developer's token creates a
private package.
