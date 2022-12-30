

## Setup

Build the PHP container

- Build the custom PHP image & start the containers
  ```shell
  git clone https://github.com/michaelkeiluweit/dev_stack.git --branch=php
  rm -rf .git/
  
  docker build --tag=mkphp .
  docker-compose up -d
  docker-compose exec php bash
  ```
- Install and initiate Codecept
  ```shell
  composer require symfony/dotenv
  composer require codeception/codeception --dev
  ./vendor/bin/codecept bootstrap
  ./vendor/bin/codecept generate:helper AuthenticationHelper
  ```
  
- Open _app/tests/Support/Helper/AuthenticationHelper.php_ and add:
  ```php
  protected array $requiredFields = ['username', 'password'];
  
  public function readUsernameFromEnvironment(): string
  {
      return $this->config['username'];
  }
  
  public function readPasswordFromEnvironment(): string
  {
      return $this->config['password'];
  }
  ```
- Open _app/tests/Acceptance.suite.yml and add:
  ```yaml
  modules:
      enabled:
          - \Tests\Support\Helper\AuthenticationHelper:
              username: "%username%"
              password: "%password%"
  ```
- Open _codeception.yml_ and add:
  ```yaml
  params:
      - .env.testing
  ```
Create _.env.testing_ (at the same place as codeception.yml is located):
  ```env
  username=dev_environment_username
  password=dev_environment_password
  ```
- Use the environment variables in a Cest:
  ```php
  $I->amOnPage('/');
  $I->submitForm('#loginForm', [
      'login' => $I->readUsernameFromEnvironment(),
      'password' => $I->readPasswordFromEnvironment()
  ]);
  ```
## Troubleshooting
- The apache webserver expects the directory `public` within the folder `app`, as it is the document root.  
In case the entrypoint of your project is different to public, a workaround could be to create a symlink: `ln -s src/public/ public`.

## Further Documentation 
- https://codeception.com/quickstart
- https://codeception.com/docs/ModulesAndHelpers