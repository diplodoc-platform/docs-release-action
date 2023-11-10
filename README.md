# docs-release-action GitHub Action

This GitHub action registers new stable version of documentation.

## Inputs

- `revision` (required) - The revision or version identifier for the documentation to be built.
- `storage-bucket` (required) - The name of the storage bucket where the built documentation will be stored. After the documentation is successfully built, it will be uploaded to this specified bucket.
- `storage-access-key-id` (required) - The access key ID associated with the account that has permission to access and upload to the specified storage bucket.
- `storage-secret-access-key` (required) - The secret access key associated with the account.
- `version` (default: empty string) - The documentation version name.
- `server` (default: 'https://viewer.diplodoc.com') - The root URL of the server.

## Usage

Create a file named `.github/workflows/release.yml` in your repo.

This workflow performs the following:
- Builds the documentation
- Uploads build output to the storage
- Registers new stable version of documentation

```yaml
name: Release

on:
  workflow_dispatch:
  pull_request:
    types: [closed]
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
        uses: diplodoc-platform/docs-upload-action@v3
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
        uses: diplodoc-platform/docs-release-action@v3
        with:
          revision: "${{ github.sha }}"
          storage-bucket: ${{ vars.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```

### Release version

```yaml
name: Release tag

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  <...>
  release:
    needs: upload
    runs-on: ubuntu-latest
    steps:
      - name: Release
        uses: diplodoc-platform/docs-release-action@v3
        with:
          revision: "${{ github.sha }}"
          version: "${{ github.ref_name }}"
          storage-bucket: ${{ secrets.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```

### Release on custom server

```yaml
name: Release

on:
  workflow_dispatch:
  pull_request:
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
        uses: diplodoc-platform/docs-release-action@v3
        with:
          server: "https://my.custom.docs.server.com"
          revision: "${{ github.sha }}"
          storage-bucket: ${{ secrets.DIPLODOC_STORAGE_BUCKET }}
          storage-access-key-id: ${{ secrets.DIPLODOC_ACCESS_KEY_ID }}
          storage-secret-access-key: ${{ secrets.DIPLODOC_SECRET_ACCESS_KEY }}
```