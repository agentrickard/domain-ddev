# Drupal 10 Development
A minimal project build for Drupal 10 module development.

## Contents

* [Getting started](#getting-started)
* [Common tasks](#common-tasks)
* [Sample workflows](#sample-workflows)
* [Developer notes](#developer-notes)

## Getting started

### Requirements
This project requires [DDEV](https://ddev.readthedocs.io/en/latest/users/install/) and [Composer 2](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) be installed before you begin.

### Setup

Checkout the project and run the following commands from project root:

* `ddev start`
* `composer install`
* `ddev auth ssh`

Now you are ready to install Drupal and test modules:

* `ddev install`

This command will install Drupal 10 plus the following useful modules: `devel`, `config_inspector`, `admin_toolbar`, and `admin_toolbar_tools`. You can login with `admin / admin` at https://d10.ddev.site/user.

#### Note on NFS:

If using NFS to speed up the container, see these steps.

* `ddev debug nfsmount` (optional)
  * See the [macOS NFS Setup section](https://ddev.readthedocs.io/en/latest/users/install/performance/#nfs) of the DDEV documentation.


### Drush

`drush 11` is installed by default and can be run with `ddev drush COMMAND`. You can use `ddev drush site:install` if you want to customize the install.

### Composer

The use of `composer require` is assumed to be used for maintenance of the core framework, not adding modules for testing.

You can use `composer require` to add modules that you need -- and to check dependencies. Do not commit those additions to the project.

### Install locations

Modules are installed to `web/modules/contrib`, which is leveraged by our [common ddev tasks](#common-tasks).

## Common tasks

We have automated common tasks as ddev commands to reduce dependencies. The following tasks are available.

### ddev check

**Command:** `ddev check MODULE`

**Example:** `ddev check admin_toolbar`

The `check` command will run code reviews using PHPCBF, PHPCS, PHPMD, and PHPStan against the selected module. PHPCBF may change the module's code as part of this action.

These commands can also be run individually as `ddev md`, `ddev sniff`, and `ddev stan`.

Note that PHPStan is pinned to [PHPStan level 2](https://phpstan.org/user-guide/rule-levels) in this command. Use [`ddev stan`](#ddev-stan) to override the level.

### ddev cleanup

**Command:** `ddev cleanup`

Removes test files generated by `ddev test`.

### ddev clone

**Command:** `ddev clone MODULE_NAME [-b BRANCH_NAME | -t TAG_NAME] [-c]`

**Example:** `ddev clone admin_toolbar`

**Example:** `ddev clone workbench_access -c`

**Example:** `ddev clone admin_toolbar -b 8.x-1.x`

Use `ddev clone` rather than `composer require` if you want to checkout a git project for development.

The `clone` command will checkout the default branch of the selected module to the `web/modules/contrib` directory.

Use the `-b` or `-t` flags to specify a branch or tag. The flags cannot be used together.

Adding the `-c` flag indicates you are a `project contributor`, not a `project maintainer`. The command will use https rather than ssh to checkout the project.

Note that this command will `delete` an existing copy of the module.

### ddev install

**Command:** `ddev install`

Installs the default drupal site.

### ddev md

**Command:** `ddev md MODULE`

**Example:** `ddev md admin_toolbar`

The `md` command will run code reviews using PHPMD against the selected module.

### ddev phpversion

**Command:** `ddev phpversion [-v VERSION] MODULE`

**Example:** `ddev phpversion admin_toolbar`

**Example:** `ddev phpversion -v 7.4 admin_toolbar`

The `phpversion` command will run PHPCS against the selected module using the `PHPCompatibility` coding standard.

Use the `-v` flag to specify a PHP version to test. By default, the version is `8.1`.

### ddev rector

**Command:** `ddev rector MODULE` or `ddev rector MODULE -d` or `ddev rector MODULE --dry-run`

**Example:** `ddev rector admin_toolbar`

**Example:** `ddev rector admin_toolbar -d`

The `rector` command will run Drupal Rector updates against the selected module, potentially rewriting the module's code. Using the `-d` or `--dry-run` flag will not perform the changes, but instead show the suggested changes.

### ddev remove

**Command:** `ddev remove MODULE`

**Example:** `ddev rector admin_toolbar`

The `ddev remove` command will uninstall a module and delete it from the `web/modules/contrib` directory.

### ddev sniff

**Command:** `ddev sniff MODULE`

**Example:** `ddev sniff admin_toolbar`

The `sniff` command will run code reviews using PHPCBF and PHPCS against the selected module. PHPCBF may change the module's code as part of this action.

### ddev stan

**Command:** `ddev stan MODULE`

**Example:** `ddev stan admin_toolbar`

**Example:** `ddev stan admin_toolbar -l 8`

The `stan` command will run code reviews using PHPStan against the selected module.

This command defaults to use [PHPStan level 2](https://phpstan.org/user-guide/rule-levels). You can pass a preferred level (`0-9`, or `max`) using the `-l` flag.

### ddev test

**Command:** `ddev test MODULE`

**Example:** `ddev test admin_toolbar`

The `ddev test` command runs all tests defined by a module according to Drupal's testing standards.

## Sample workflows

Let's assume you want to write a patch for the `workbench_tabs` module. Follow these steps:

### Creating a patch

* `ddev start`
* `composer install`
* `ddev install`
* `ddev clone workbench_tabs`
  * If you are a maintainer, use `ddev clone workbench_tabs -y`
* `ddev test workbench_tabs`
  * Run this step to ensure that existing tests pass.
  * Then run `ddev cleanup` to remove test files.
* `ddev drush en workbench_tabs`
* Now develop your new feature or bugfix in the module
  * Do not check your changes into git before creating the patch.
  * Create a patch
    * `cd web/modules/workbench_tabs`
    * `git diff > ISSUE#-MODULE-summary-COMMENT.patch`
  * Now you can upload the patch to the Drupal.org issue.

### Testing a patch

* `ddev start`
* `composer install`
* `ddev install`
* `ddev clone workbench_tabs`
  * If you are a maintainer, use `ddev clone workbench_tabs -y`
* `ddev test workbench_tabs`
  * Run this step to ensure that existing tests pass.
  * Then run `ddev cleanup` to remove test files.
* `ddev drush en workbench_tabs`
* Download the patch to `web/modules/workbench_tabs`:
  * `cd web/modules/workbench_tabs`
  * `wget https://drupal.org/PATH-TO-FILE/NAME-OF-FILE`
  * `patch -p1 < NAME-OF-FILE`
  * `ddev drush cr`
* Now you can test the patch in the site.

## Developer notes

We are deliberately not using other project dependencies (notably `the-build`, `phing`, and `drupal-skeleton`) on this project. We want this project template to be as independent as possible. This project also does not require integrations with CircleCI and web hosts.

Solr support is not provided by default. It can be added later if needed.

The `main` branch is locked against commits that are not in approved pull requests.
