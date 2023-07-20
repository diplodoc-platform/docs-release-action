# docs-release-action GitHub action

## Inputs

- `revision` (required) - The revision or version identifier for the documentation to be built.
- `src-root` (required) - The root directory where the source documentation files are located. The action will use this directory as the base location to look for the source files that need to be built.
- `s3-bucket` (required) - The name of the Amazon S3 bucket where the built documentation will be stored. After the documentation is successfully built, it will be uploaded to this specified bucket.
- `s3-endpoint` (required) - The endpoint URL of the Amazon S3 service.
- `s3-access-key-id` (required) - The access key ID associated with the AWS account that has permission to access and upload to the specified S3 bucket.
- `s3-secret-access-key` (required) - The secret access key associated with the AWS account.
- `s3-region` (required) - The AWS region where the specified S3 bucket is located.
- `lint-root` (default: `./_docs-lint`) - The root directory for the linting process. This is an optional parameter, and if not specified, the default value will be used.
- `build-root` (default: `./_docs-build`) - The root directory for the built documentation. This is an optional parameter, and if not specified, the default value will be used.

## Example

```yaml
name: Release

on:
  workflow_dispatch:

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Release
        uses: diplodoc-platform/docs-release-action@v1
        with:
          revision: "${{ github.sha }}"
          src-root: './docs'
          s3-bucket: ${{ secrets.DOCS_AWS_BUCKET }}
          s3-endpoint: ${{ vars.DOCS_AWS_ENDPOINT }}
          s3-access-key-id: ${{ secrets.DOCS_AWS_KEY_ID }}
          s3-secret-access-key: ${{ secrets.DOCS_AWS_SECRET_ACCESS_KEY }}
          s3-region: ${{ vars.DOCS_AWS_REGION }}
```