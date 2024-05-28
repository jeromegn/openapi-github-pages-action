# openapi-github-pages-action
GitHub Action to generate and deploy API documentation to GitHub Pages, using OpenAPI specification files as input.

By default, API docs are generated at *username*.github.io/*repository-name*/*provided-api-doc-filepath*.

If you've set up a [GitHub Pages custom domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site) for your account, that will be used instead of *username*.github.io.

See example API docs generated by this GitHub Action:
* [test/docs.html](https://www.marksayson.com/openapi-github-pages-action/test/docs.html)
* [v1/docs.html](https://www.marksayson.com/openapi-github-pages-action/v1/docs.html)
* [v2/docs.html](https://www.marksayson.com/openapi-github-pages-action/v2/docs.html)

## Prerequisites

1. GitHub Pages requires that source repositories either be public or in [supported paid accounts](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages).
2. The repository must have GitHub Pages enabled with GitHub Actions, by going to "Settings" > "Pages" > "Build and deployment", and under "Source", selecting "GitHub Actions".
3. The repository must provide a pre-built OpenAPI JSON spec file.

## Usage
See [action.yml](action.yml)

Add the following permissions to your GitHub workflow, prior to the lines defining workflow jobs.

```yaml
# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write
```

Add the following steps to your GitHub workflow, replacing the input values:

```yaml
- name: Generate API docs and deploy to GitHub Pages
  uses: msayson/smithy-gh-pages-action@v2.0.0
  with:
    # JSON string configs for each OpenAPI spec to document
    # - branch: (Optional) Git branch to fetch spec file from, eg. origin/main
    #           If not provided, assumes file is available from current branch
    # - openapi-json-filepath: Filepath of OpenAPI JSON spec
    # - api-doc-filepath: Filepath for generated API doc, within the common directory defined by api-docs-dir
    api-configs: |-
      [
        {
          "branch": "YOUR_BRANCH",
          "openapi-json-filepath": "YOUR_OPENAPI_SPEC_FILEPATH",
          "api-doc-filepath": "YOUR_API_DOC_FILEPATH"
        }
      ]
    # Parent directory to use for generated API docs HTML
    api-docs-dir: 'YOUR_API_DOCS_FILEPATH'
```

## Scenarios

* [Generate API docs for main branch](#Generate-API-docs-for-main-branch)
* [Generate API docs for multiple releases](#Generate-API-docs-for-multiple-releases)

### Generate API docs for main branch

```yaml
name: Generate API docs and deploy to GitHub Pages

on:
  # Automatically trigger when push to main branch
  push:
    branches: ["main"]
  # Enable running workflow manually from GitHub Actions
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  generate-api-docs:
    name: Generate API Documentation
    runs-on: ubuntu-latest
    steps:
      - name: Generate API docs and deploy to GitHub Pages
        uses: msayson/openapi-github-pages-action@v1.0.0
        with:
          api-configs: |-
            [
              {
                "openapi-json-filepath": "openapi/ConsentManagementApi.openapi.json",
                "api-doc-filepath": "latest/docs.html"
              }
            ]
          api-docs-dir: docs
```

### Generate API docs for multiple releases

```yaml
name: Generate API docs and deploy to GitHub Pages

on:
  # Automatically trigger when push to any branch
  push:
  # Enable running workflow manually from GitHub Actions
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  generate-api-docs:
    name: Generate API Documentation
    runs-on: ubuntu-latest
    steps:
      - name: Generate API docs and deploy to GitHub Pages
        uses: msayson/openapi-github-pages-action@v1.0.0
        with:
          api-configs: |-
            [
              {
                "branch": "main",
                "openapi-json-filepath": "openapi/ConsentManagementApi.openapi.json",
                "api-doc-filepath": "latest/docs.html"
              },
              {
                "branch": "v1",
                "openapi-json-filepath": "openapi/ConsentManagementApi.openapi.json",
                "api-doc-filepath": "v1/docs.html"
              },
              {
                "branch": "v2",
                "openapi-json-filepath": "openapi/ConsentManagementApi.openapi.json",
                "api-doc-filepath": "v2/docs.html"
              }
            ]
          api-docs-dir: docs
```

## Technologies
[yq](https://github.com/mikefarah/yq) is used to convert JSON to YAML, since YAML is the required format for ReDoc, while many tools such as Smithy generate OpenAPI specs with the JSON format.

[ReDoc](https://github.com/Redocly/redoc) is used to automatically generate API documentation from the OpenAPI YAML specification.

[GitHub Actions](https://docs.github.com/en/actions) are used to automatically generate and deploy HTML API documentation to [GitHub Pages](https://pages.github.com/).
