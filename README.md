# Local Dependency Management

## Introduction

This is an example of a best practice to keep code related dependencies inside of the module, theme or profile through
Composer. The big advantage of this is that all your dependencies will be handled correctly. And you will not
unknowingly remove some package that is required by another one.

## Howto

### Use a lib folder structure

You can put your custom modules, themes and profiles into a lib folder.

```shell
root@itr01428:~/PhpstormProjects/tips$ tree -L 4 lib/
lib/
└── custom
    ├── modules
    │   └── project_core
    │       ├── composer.json
    │       ├── project_core.info.yml
    │       ├── project_core.module
    │       └── src
    ├── profiles
    │   └── project_profile
    │       ├── composer.json
    │       ├── config
    │       ├── project_profile.info.yml
    │       └── project_profile.profile
    └── themes
        └── project_theme
            ├── composer.json
            ├── project_theme.info.yml
            ├── project_theme.theme
            └── src

10 directories, 9 files
```

### Put a composer.json file in each local package.

Act as if each package is published by placing a composer.json with minimum properties:

* name
* description
* type
* require (if you have code dependencies)

```json
{
  "name": "custom/project_core",
  "description": "Project core",
  "type": "drupal-custom-module",
  "require": {
    "drupal/smart_date": "^3.0"
  }
}
```

## Make the local modules discoverable by Composer

Make sure you have the following packages installed into your project:

```shell
composer require composer/installers drupal/core-composer-scaffold
```

And put the following or similar configuration into your extra section:

```json
{
    "extra": {
        "drupal-scaffold": {
            "allowed-packages": [
                "drupal/core"
            ],
            "locations": {
                "web-root": "./docroot"
            }
        },
        "installer-paths": {
            "docroot/core": [
                "type:drupal-core"
            ],
            "docroot/modules/contrib/{$name}": [
                "type:drupal-module"
            ],
            "docroot/modules/custom/{$name}": [
                "type:drupal-custom-module"
            ],
            "docroot/profiles/contrib/{$name}": [
                "type:drupal-profile"
            ],
            "docroot/profiles/custom/{$name}": [
                "type:drupal-custom-profile"
            ],
            "docroot/themes/contrib/{$name}": [
                "type:drupal-theme"
            ],
            "docroot/themes/custom/{$name}": [
                "type:drupal-custom-theme"
            ]
        }
    },
    "repositories": {
        "local": {
            "type": "path",
            "url": "lib/custom/*/*"
        }
    }
}
```
Then just install them as any other package. But either allow * or @dev.

```shell
composer require custom/project_core:@dev custom/project_theme:@dev custom/project_profile:@dev
```

### Sidenote

You may need to adjust the build process if you want to have the source files inside of the docroot for deployment.
Otherwise the packages will be symlinked.

```shell
root@itr01428:~/PhpstormProjects/tips$ tree -L 4 docroot/modules/ docroot/themes/ docroot/profiles/
docroot/modules/
├── contrib
├── custom
│   └── project_core -> ../../../lib/custom/modules/project_core
└── README.txt
docroot/themes/
├── custom
│   └── project_theme -> ../../../lib/custom/themes/project_theme
└── README.txt
docroot/profiles/
├── custom
│   └── project_profile -> ../../../lib/custom/profiles/project_profile
└── README.txt

```


For example you will have to run:

```shell
# If you already built the code base with a regular composer install.
# Then you need to remove the docroot folders first or the symlinks wont
# be updated to a mirror of the files.
rm -rf docroot/modules docroot/themes docroot/profiles
# Run composer install --no-dev with path repos set to mirrored.
COMPOSER_MIRROR_PATH_REPOS=1 composer install --no-dev
```







