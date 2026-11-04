# Gingerminds Symfony recipes

[Symfony Flex](https://symfony.com/doc/current/setup/flex.html) recipes of the Gingerminds bundles:

| Package | Recipe |
|---|---|
| [`gingerminds/symfony-core`](https://packagist.org/packages/gingerminds/symfony-core) | `gingerminds/symfony-core/1.10` |
| [`gingerminds/symfony-multisite`](https://packagist.org/packages/gingerminds/symfony-multisite) | `gingerminds/symfony-multisite/1.3` |
| [`gingerminds/symfony-media-manager`](https://packagist.org/packages/gingerminds/symfony-media-manager) | `gingerminds/symfony-media-manager/1.2` |
| [`gingerminds/symfony-cms`](https://packagist.org/packages/gingerminds/symfony-cms) | `gingerminds/symfony-cms/1.1` |

On `composer require`, a recipe registers the bundle, copies its routes and default
configuration, adds its environment variables and prints the remaining steps (security,
migrations, permissions...). Flex cannot run commands nor merge into a file owned by another
recipe (`security.yaml`, `api_platform.yaml`...): those steps stay in the bundle documentation.

## Use the recipes in a project

Add this endpoint to the `composer.json` of the project, before the official ones:

```json
"extra": {
    "symfony": {
        "endpoint": [
            "https://raw.githubusercontent.com/gingerminds/symfony-recipes/flex/main/index.json",
            "flex://defaults"
        ]
    }
}
```

Then:

```bash
composer require gingerminds/symfony-core      # applies the recipe
composer recipes                                # lists the installed recipes and their updates
composer recipes:update gingerminds/symfony-core
composer recipes:install gingerminds/symfony-core --force -v   # re-apply (overwrites the files)
```

## Write a recipe

A recipe is a directory `<vendor>/<package>/<version>/`: Flex picks the highest recipe version
lower than or equal to the installed package version. Create a new version directory only when
the files of the recipe must change for a new version of the package.

| File | Role |
|---|---|
| `manifest.json` | The [configurators](https://github.com/symfony/recipes#creating-recipes): `bundles`, `copy-from-recipe`, `env`, `add-lines`... Always declare `bundles`: with a recipe, Flex no longer registers the bundle by itself. |
| `config/`, `assets/`... | Copied into the project by `copy-from-recipe` (never over an existing file). |
| `post-install.txt` | Printed after the installation (`post-install-output`). |

Flex writes the `env` variables to `.env` (and `.env.dist`), not to `.env.example`.

## Publishing

On every push to `main` (GitLab mirrored to GitHub), the GitHub workflow
`.github/workflows/flex-update.yml` lints the manifests, generates the endpoint with
[`symfony-tools/recipes-checker`](https://github.com/symfony-tools/recipes-checker) and
force-pushes it to the `flex/main` branch. Never commit to `flex/main` by hand.

Build the endpoint locally:

```bash
git clone https://github.com/symfony-tools/recipes-checker.git ../recipes-checker
composer install --working-dir=../recipes-checker --no-dev
php ../recipes-checker/run lint:manifests
mkdir -p /tmp/flex-endpoint
git ls-tree HEAD */*/* | php ../recipes-checker/run generate:flex-endpoint gingerminds/symfony-recipes main flex/main /tmp/flex-endpoint
```

`git ls-tree HEAD` only lists committed recipes: commit before generating.
