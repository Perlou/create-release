# GitHub Action - Releases API

This GitHub Action (written in JavaScript) wraps the [GitHub Release API](https://docs.github.com/en/rest/releases/releases), specifically the [Create a Release](https://docs.github.com/en/rest/releases/releases#create-a-release) endpoint, to allow you to leverage GitHub Actions to create releases.

## Usage

### Inputs

For more information on these inputs, see the [`action.yml`](/action.yml)

- `tag_name`: The name of the tag for this release.
- `release_name`: The name of the release.
- `body`: Text describing the contents of the release. Optional, and not needed if using `body_path`.
- `body_path`: A file with contents describing the release. Optional, and not needed if using `body`.
- `draft`: `true` to create a draft (unpublished) release, `false` to create a published one. Default: `false`
- `prerelease`: `true` to identify the release as a prerelease. `false` to identify the release as a full release. Default: `true` if the tag contains a hyphen.
- `commitish` : Any branch or commit SHA the Git tag is created from, unused if the Git tag already exists. Default: SHA of current commit.
- `owner`: The name of the owner of the repo. Used to identify the owner of the repository. Used when cutting releases for external repositories. Default: Current owner
- `repo`: The name of the repository. Used to identify the repository on which to release. Used when cutting releases for external repositories. Default: Current repository

#### `body_path`

The `body_path` is valuable for dynamically creating a `.md` within code commits and even within the Github Action steps leading up to the `create-release`.

### Outputs

For more information on these outputs, see the [API Documentation](https://developer.github.com/v3/repos/releases/#response-4) for an example of what these outputs look like.

- `id`: The release ID
- `html_url`: The URL users can navigate to in order to view the release. i.e. `https://github.com/octocat/Hello-World/releases/v1.0.0`
- `upload_url`: The URL for uploading assets to the release, which could be used by GitHub Actions for additional uses, for example the [`@actions/upload-release-asset`](https://www.github.com/actions/upload-release-asset) GitHub Action

### Example workflow - create a release

On every `push` to a tag matching the pattern `v*`, [create a release](https://developer.github.com/v3/repos/releases/#create-a-release):

```yaml
on:
  push:
    # Sequence of patterns matched against refs/tags
    tags:
      - 'v*' # Push events to matching v*, i.e. v1.0, v20.15.10

name: Create Release

jobs:
  build:
    name: Create Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Create Release
        id: create_release
        uses: perlou/create-release@v1
        env:
          # This token is provided by Actions, you do not need to create your own token
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Changes in this Release
            - First Change
            - Second Change
          draft: false
          prerelease: false
```

This will create a [Release](https://help.github.com/en/articles/creating-releases), as well as a [`release` event](https://developer.github.com/v3/activity/events/types/#releaseevent), which could be handled by a third party service, or by GitHub Actions for additional uses, for example the [`@actions/upload-release-asset`](https://www.github.com/actions/upload-release-asset) GitHub Action. This uses the `GITHUB_TOKEN` provided by the [virtual environment](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#github_token-secret), so no new token is needed.

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)
