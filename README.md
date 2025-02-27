# docs-release-action GitHub Action

This GitHub action registers new stable version of documentation.

## Inputs

- `revision` (required) - The revision or version identifier for the documentation to be built.
- `storage-bucket` (required) - The name of the storage bucket where the built documentation will be stored. After the documentation is successfully built, it will be uploaded to this specified bucket.
- `storage-access-key-id` (required) - The access key ID associated with the account that has permission to access and upload to the specified storage bucket.
- `storage-secret-access-key` (required) - The secret access key associated with the account.
- `version` (default: empty string) - The documentation version name.
- `update-only-version` (default: 'false') - Specifies whether to update the version information without updating the head. Set it to 'true' if only the version should be updated.
- `server` (default: 'https://viewer.diplodoc.com') - The root URL of the server.

## Usage

Create a file named `.github/workflows/release.yml` in your repo.

This workflow performs the following:
- Checks for changes to run the build for documentation only
- Builds the documentation
- Uploads build output to the storage
- Registers new stable version of documentation

```yaml
name: Release

on:
  workflow_dispatch:
  pull_request:
    types: [closed]
    paths: 'docs/**'
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    permissions: write-all
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build
        uses: diplodoc-platform/docs-build-action@v3
        with:
          revision: "${{ github.sha }}"
          src-root: "./docs"
  upload:
    needs: build
    runs-on: ubuntu-latest
    permissions: write-all
    steps:
      - name: Upload
        uses: diplodoc-platform/docs-upload-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          storage-endpoint: ${{ vars.DIPLODOC_STORAGE_ENDPOINT }}
          storage-region: ${{ vars.DIPLODOC_STORAGE_REGION }}
          storage-bucket: ${{ vars.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
  release:
    needs: upload
    runs-on: ubuntu-latest
    steps:
      - name: Release
        uses: diplodoc-platform/docs-release-action@v2
        with:
          revision: "${{ github.sha }}"
          storage-bucket: ${{ vars.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```

### Release version

Basic example, the head and version number will be updated:
```yaml
name: Release tag

on:
  push:
    paths: 'docs/**'
    tags:
      - 'v*.*.*'

jobs:
  <...>
  release:
    needs: upload
    runs-on: ubuntu-latest
    steps:
      - name: Release
        uses: diplodoc-platform/docs-release-action@v2
        with:
          revision: "${{ github.sha }}"
          version: "${{ github.ref_name }}"
          storage-bucket: ${{ secrets.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```

Advanced usage, you can choose releases of which branch will be considered default and will be updated together with the head, and in which releases only the version will be updated.
```yaml
name: Release tag

on:
  push:
    branches:
      - 'main'
      - 'stable-**'
    paths: 
      - 'docs/**'
  workflow_dispatch:

jobs:
  <...>
  release:
    needs: upload
    runs-on: ubuntu-latest
    concurrency:
      group: release-documentation-${{ github.ref }}
      cancel-in-progress: true
    steps:
      - name: Extract version
        shell: bash
        run: echo "version=${GITHUB_HEAD_REF:-${GITHUB_REF#refs/heads/}}" | sed -e 's|stable-|v|g' -e 's|-|.|g' >> $GITHUB_OUTPUT
        id: extract_version

      - name: Set Default Version
        id: set-default-version
        shell: bash
        env:
          DEFAULT_VERSION: "main"
        run: |
          BRANCH_NAME=${GITHUB_REF##*/}
          if [[ "$BRANCH_NAME" == "$DEFAULT_VERSION" ]]; then
            echo "UPDATE_ONLY_VERSION=false" >> $GITHUB_ENV
          else
            echo "UPDATE_ONLY_VERSION=true" >> $GITHUB_ENV
          fi

      - name: Release
        uses: diplodoc-platform/docs-release-action@v2
        with:
          revision: "${{ github.sha }}"
          version: "${{ steps.extract_version.outputs.version }}"
          storage-bucket: ${{ secrets.DOCS_PROJECT_NAME }}
          storage-access-key-id: ${{ secrets.DOCS_AWS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DOCS_AWS_SECRET_ACCESS_KEY }}
          update-only-version: "${{ env.UPDATE_ONLY_VERSION }}"
```

### Release on custom server

```yaml
name: Release

on:
  workflow_dispatch:
  pull_request:
    paths: 'docs/**'
    types: [closed]
    branches:
      - main

jobs:
  <...>
  release:
    needs: upload
    runs-on: ubuntu-latest
    steps:
      - name: Release
        uses: diplodoc-platform/docs-release-action@v2
        with:
          server: "https://my.custom.docs.server.com"
          revision: "${{ github.sha }}"
          storage-bucket: ${{ secrets.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```