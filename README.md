# Test D9 Site

This is a test D9 site which can be used for practicing test writing and running. This is to be used in tandem with the [Drupal Testing Crash Course](https://2020.drupalcampcolorado.org/trainings/drupal-testing-crash-course) training at DrupalCamp Colorado.

## Dependencies

This project uses ddev. Follow these [instructions for installing ddev on your local system](https://ddev.readthedocs.io/en/stable/#installation).

## Getting Started

1. Fork this repo.
2. Clone your fork locally and change into that directory.
3. Start `ddev`
    ```bash
    ddev start
    ```
4. Install dependencies
    ```bash
    ddev composer install
    ```

    _Note: This injects a `.env` and `phpunit.xml` file into your `web/core` folder based on the templates in `templates/core.env` and `templates/core.phpunit.xml` respectively. This is for convenience for this project. In a real project, you would want to ensure those files are handled securely. See `web/core/.env.example` and `web/core/phpunit.xml.dist` for more information about those files._
5. Perform a basic site install and setup some basic users
    ```bash
    ddev exec drush si my_profile --config-dir=../config/sync --account-pass=admin -y
    ddev exec drush user:create bobby --mail="bobby@example.com" --password="bobby"
    ddev exec drush user:create carol --mail="carol@example.com" --password="carol"
    ddev exec drush user:create david --mail="david@example.com" --password="david"
    ddev exec drush user-add-role "administrator" admin
    ddev exec drush user-add-role "super_secret" bobby
    ddev exec drush user-add-role "yet_another_role" carol    
    ```
8. Back in your normal terminal, open the site in your browser:
    ```bash
    ddev launch
    ```

You can then use the following sample credentials to test out various roles:

1. `admin`/`admin` - Has all the privileges
2. `bobby`/`bobby` - Has the `super_secret` role
3. `carol`/`carol` - Has the `yet_another_role` role
4. `david`/`david` - Has no roles set

## What We're Testing

Most of what we will be working with is located in a custom module called
[_my_testing_module_](web/modules/custom/my_testing_module).

This module is intentionally broken for the sake of demonstrating test-driven development.

This module _should_ do the following when you navigate to `/my-message`:
  1. Shows a message for any authenticated users that says: "You are logged in"
      - It's actually showing "You _might be_ logged in" instead.
  2. Shows a message for users with the _my super secret privilege_ permission that says: "You are super special."
      - It's actually showing "You _aren't all that_ special." instead. Well that's not very nice.
  3. Shows a message for users with the _yet another privilege_ permission that says "You have yet another privilege."
      - This one is working as expected.
  4. If multiple scenarios apply, it should show all of the above messages.
      - It's actually only showing one of these messages.
  5. If a user is not logged in, they should get an access forbidden error.
      - It's actually showing them the message shown to authenticated users.


## Test Runners

### PHPUnit (PHP Testing)

[PHPUnit](https://phpunit.de/) is a tool for testing PHP functionality in Drupal.

There are a [few different ways to run PHPUnit tests](https://www.drupal.org/docs/testing/phpunit-in-drupal/running-phpunit-tests). This training uses the `core/scripts/run-tests.sh` method, as that is what DrupalCI uses.

#### Run all PHPUnit tests:
If you want to run all PHPUnit-based tests for your module, you can do so with one of the following commands:

- *From inside the ddev container (i.e. after running `ddev ssh`):*
  ```bash
  php core/scripts/run-tests.sh --color --verbose --sqlite /tmp/a.sqlite my_testing_module
  ```
- *From outside the ddev container:*
  ```bash
  ddev exec php core/scripts/run-tests.sh --color --verbose --sqlite /tmp/a.sqlite my_testing_module
  ```

_Note: We're using the `--sqlite` flag. When the test runner bootstraps Drupal, it will save results in a sqlite database, that we're just storing in the `/tmp` directory for now. Alternatively we could download the SimpleTest module (contrib as of D9) and use mysql DB instead._

#### Run specific PHPUnit tests:

Running every test, all the time, can sometimes take a while. If you want to focus on a specific test you can do so by using the `--class` flag instead:

- *From inside the ddev container (i.e. after running `ddev ssh`):*
  ```bash
  php core/scripts/run-tests.sh --color --verbose --sqlite /tmp/a.sqlite --class 'Drupal\Tests\my_testing_module\Functional\MyFunctionalTest'
  ```
- *From outside the ddev container (_Note you have to escape the slashes_):*
  ```bash
  ddev exec php core/scripts/run-tests.sh --color --verbose --sqlite /tmp/a.sqlite --class 'Drupal\\Tests\\my_testing_module\\Functional\\MyFunctionalTest'
  ```

### Nightwatch.js (Javascript Testing)

[Nightwatch.js](https://nightwatchjs.org/) is a tool used for javascript testing in Drupal.

To add support for nightwatch testing in DDEV, a [docker-compose.chromedriver.yml]() file must be added into your `.ddev` directory.

Read Matt Glaman's [Running Drupal's Nightwatch test suite on DDEV](https://glamanate.com/blog/running-drupals-nightwatch-test-suite-ddev) article for more information about setting this up.

#### Run all Nightwatch.js tests

- *From inside the ddev container (i.e. after running `ddev ssh`):*
  ```bash
  cd /var/www/html/web/core
  yarn test:nightwatch ../modules/custom/my_testing_module/tests/src/Nightwatch
  ```

- *From outside the ddev container (_Note: You have to specify which directory to run inside using the `-d` flag, and all other paths are relative to that_):*
  ```bash
  ddev exec -d /var/www/html/web/core yarn test:nightwatch ../modules/custom/my_testing_module/tests/src/Nightwatch
  ```

#### Run specific Nightwatch.js tests

- *From inside the ddev container (i.e. after running `ddev ssh`):*
```bash
cd /var/www/html/web/core
yarn test:nightwatch ../modules/custom/my_testing_module/tests/src/Nightwatch/MyNightwatchTest.js
```
- *From outside the ddev container (_Note you have to specify which directory to run inside using the `-d` flag, and all other paths are relative to that_):*
```bash
ddev exec -d /var/www/html/web/core yarn test:nightwatch ../modules/custom/my_testing_module/tests/src/Nightwatch/MyNightwatchTest.js
```

### Behat (Behavioral Testing w/ Cucumber)

[Behat](https://docs.behat.org/en/latest/) is a tool for behavorial testing in Drupal.

#### Run all behat tests
- *From inside the ddev container (i.e. after running `ddev ssh`):*
```bash
cd /var/www/html
vendor/bin/behat
```
- *From outside the ddev container (_Note you have to specify which directory to run inside using the `-d` flag, and all other paths are relative to that_):*
```bash
ddev exec -d /var/www/html vendor/bin/behat
```

#### Run specific behat tests
- *From inside the ddev container (i.e. after running `ddev ssh`):*
```bash
cd /var/www/html
vendor/bin/behat features/drupal/cache.feature
```
- *From outside the ddev container (_Note you have to specify which directory to run inside using the `-d` flag, and all other paths are relative to that_):*
```bash
ddev exec -d /var/www/html vendor/bin/behat
```