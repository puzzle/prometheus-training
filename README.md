# Prometheus Basics Training

Interactive Prometheus Basics Training: [prometheus-training.puzzle.ch](https://prometheus-training.puzzle.ch/)


## Content Sections

The training content resides within the [content](content) directory.

The main part are the labs, which can be found at [content/en/docs](content/en/docs).

## Hugo

This site is built using the static page generator [Hugo](https://gohugo.io/).

The page uses the [docsy theme](https://github.com/google/docsy) which is included as a Git Submodule.
Docsy is being enhanced using [docsy-plus](https://github.com/puzzle/docsy-plus/) as well as [docsy-puzzle](https://github.com/puzzle/docsy-puzzle/) and [docsy-acend](https://github.com/puzzle/docsy-acend/) for brand specific settings.

After cloning the main repo, you need to initialize the submodules:

```bash
git submodule update --init --recursive
```

In order to update all submodules, run the following command:

```bash
git pull --recurse-submodules
```

The default configuration uses the Puzzle setup from [config/_default](config/_default/config.toml).
Alternatively you can use the acend setup from [config/acend](config/acend/config.toml), which is enabled with `--environment acend`.

### Docsy Theme Usage

* [Official docsy documentation](https://www.docsy.dev/docs/)
* [Docsy Plus](https://github.com/puzzle/docsy-plus/)


## Build using Docker

Build the image:

```bash
docker build -t acend/prometheus-basics-training:latest .
```

Run it locally:

```bash
docker run -i -p 8080:8080 acend/prometheus-basics-training
```


### Using Buildah and Podman

Build the image:

```bash
buildah build-using-dockerfile -t acend/prometheus-basics-training:latest .
```

Run it locally with the following command. Beware that `--rmi` automatically removes the built image when the container stops, so you either have to rebuild it or remove the parameter from the command.

```bash
podman run --rm --rmi --interactive --publish 8080:8080 localhost/acend/prometheus-basics-training
```


## How to develop locally

To develop locally we don't want to rebuild the entire container image every time something changed, and it is also important to use the same hugo versions like in production.
We simply mount the working directory into a running container, where hugo is started in the server mode.

```bash
export HUGO_VERSION=$(grep "FROM klakegg/hugo" Dockerfile | sed 's/FROM klakegg\/hugo://g' | sed 's/ AS builder//g')
docker run --rm --interactive --publish 8080:8080 -v $(pwd):/src klakegg/hugo:${HUGO_VERSION} server -p 8080 --bind 0.0.0.0
```

Using Podman
```bash
export HUGO_VERSION=$(grep "FROM klakegg/hugo" Dockerfile | sed 's/FROM klakegg\/hugo://g' | sed 's/ AS builder//g')
podman run --rm --interactive --publish 8080:8080 -v $(pwd):/src:Z klakegg/hugo:${HUGO_VERSION} server -p 8080 --bind 0.0.0.0
```


## Linting of Markdown content

Markdown files are linted with [markdownlint](https://github.com/DavidAnson/markdownlint).
Custom rules are in [markdownlint.json](markdownlint.json).
There's a GitHub Action [github/workflows/markdownlint.yaml](github/workflows/markdownlint.yaml) for CI.
For local checks, you can either use Visual Studio Code with the corresponding extension, or the command line like this:

```bash
npm install
node_modules/.bin/markdownlint content
```


## Github Actions


### Build

The [build action](.github/workflows/build.yaml) is fired on Pull Requests does the following

* builds all PR Versions (Linting and Docker build)
* deploys the built container images to the container registry
* Deploys a PR environment in a k8s test namespace with helm
* Triggers a redeployment
* Comments in the PR where the PR Environments can be found


### PR Cleanup

The [pr-cleanup action](.github/workflows/pr-cleanup.yaml) is fired when Pull Requests are closed and does the following

* Uninstalls PR Helm Release


### Push Main

The [push main action](.github/workflows/push-main.yaml) is fired when a commit is pushed to the main branch (eg. a PR is merged) and does the following, it's very similar to the Build Action

* builds main Versions (Linting and Docker build)
* deploys the built container images to the container registry
* Deploys the main Version on k8s using helm
* Triggers a redeployment


## Helm

Manually deploy the training Release using the following command:

```bash
helm install --repo https://acend.github.io/helm-charts/  <release> acend-training-chart --values helm-chart/values.yaml -n <namespace>
```

For debugging purposes use the `--dry-run` parameter

```bash
helm install --dry-run --repo https://acend.github.io/helm-charts/  <release> acend-training-chart --values helm-chart/values.yaml -n <namespace>
```


## Contributions

If you find errors, bugs or missing information, please help us improve and have a look at the [Contribution Guide](CONTRIBUTING.md).
