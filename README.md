README.md
Quién tiene acceso

Propiedades del sistema
Tipo
Texto
Tamaño
5 kB
Almacenamiento usado
5 kB
Ubicación
Vetlucent
Propietario
Andrés Hernández
Modificado
31 ene 2022 por Andrés Hernández
Creado
20:47
Sin descripción
Los usuarios con acceso de lectura pueden descargar este elemento
# Vetlucent

Vetlucent is an online wiki and course provider for all things medical imaging related.


Required Environment / Minimum Setup
----------------------------------------------

* Ruby 2.6.3 (`rbenv install 2.6.3`) (or `rvm` equivalent)
* Postgresql (`brew install postgresql@11`) 
  * Configure bundler to use the installed formula (`bundle config build.pg --with-pg-config="$(brew --prefix postgresql@11)/bin/pg_config"`)
* Imagemagick (`brew install imagemagick@6`)
* Redis (`brew install redis`)
* Elasticsearch (`brew install elasticsearch@6.8`)
  * If you are upgrading from a previous version of ElasticSearch, ensure that you
    remove all data along with the binaries for the previous version (found in
    `/usr/local/var/elasticsearch`) when you upgrade.
* Other dependencies (`brew install libmagic jpeginfo`)
* nodejs/yarn (`brew install node yarn`)
* wkhtmltopdf (`brew cask install wkhtmltopdf`)
* Headless chrome and chromedriver (`brew cask install chromedriver`)


Notable Deviations
----------------------------------------------

Currently there is no seed file to add initial data. Fastest way is to get a db backup from
staging and restore it to your local machine.

Getting a copy of the database requires the `sentinel` tool.

* `sentinel db-dump --list` to get a list of current database dumps

Downloading a copy of the DB requires the `sentinel` tool.

* `sentinel db-dump --pick` will let you select a dump to download.


Accessing the Site
----------------------------------------------

* [production](https://vetlucent.com)
* [staging](https://staging.vetlucent.com/)


Configuration
----------------------------------------------

* Clone the repository

  ```
  git clone git@github.com:candelavet/vetradiology.git
  cd vetradiology
  ```

* Run the setup script

  ```
  bin/setup
  ```

* Update the values in `.env` for your local development/test environment


Walkthrough / Smoke Test
----------------------------------------------


```
brew services start elasticsearch
bundle exec rake searchkick:reindex:all
bundle exec rake one_off:create_subscription_products
foreman start -f Procfile.dev
```


Testing
----------------------------------------------

* Local

  ```
  bundle exec rake db:test:prepare
  bundle exec rake
  ```

  This can take 80+ minutes on a good day.


Staging Environment
----------------------------------------------

* Obtain sentinel

  Contact operations branch and get the sentinel utility.
  See https://downloads.opscare.io/sentinel/readme.html for details
  
  You will need to generate an SSH key pair for the Sentinel tool:
  `pushd ~/.ssh && ssh-keygen -t rsa -b 4096 -f ./radiopaedia_sentinel && popd`
  
  You will need to email `~/.ssh/radiopaedia_sentinel.pub` to ReInteractive.

* Deploy

  ```
  cp OpsCare.yml.sample OpsCare.yml

  sentinel deploy staging
  ```

* SSH

  ```
  sentinel ssh staging --role web
  ```

Production Environment
----------------------------------------------

* Obtain sentinel

  Contact operations branch and get the sentinel utility.

* Deploy

  ```
  cp OpsCare.yml.sample OpsCare.yml

  sentinel deploy production
  ```

* SSH

  ```
  sentinel ssh production --role web
  ```
  
Production Asset Build
----------------------------------------------

In order to complete a full, production-equivalent Rake `assets:precompile`,
you will need to:

- Add a `production` key to the following files:
  - `config/access.yml`
  - `config/settings.yml`
  - `config/database.yml`
  - `config/secrets.yml`
- rename `bin/yarn`
  - This assumes you have already done `yarn install`
- run `MAILCHIMP_API_KEY=placeholder-us4 MAILCHIMP_USER_TOKEN=placeholder bin/rake assets:precompile`

This will create a complete, production-ready, compiled Rails and Webpack
assets tree under `tmp/cache` and `public/`. You are now ready to test.

Workflow
----------------------------------------------

- Set up `candelavet/vetradiology` as `origin`, that's where you pull from, and where code is deployed from
- Make branches from `develop` according to git flow format e.g. `feature/my-feature`
- Push your changes up to `origin`, create a PR to `develop` branch for code review
- Travis CI setup, watching the repo, so once a PR to `develop` is created you should be able to see the whole test suite being run (it might take an hour)
- Once your code has been reviewed and CI has passed, merge your feature branch into `develop`
- Deploy `develop` branch to staging by `sentinel deploy staging`, or `master` to production by `sentinel deploy production`

This workflow is important, as it keeps the repos in sync. We do not want to end up with one repo weeks behind the other.

If you must break this workflow for any reason, make sure that when you are finished both repos are in sync.

Design
----------------------------------------------


Extended Resources
----------------------------------------------

See additional documentation in the [doc](https://github.com/radiopaedia/Radiopaedia/tree/master/doc) directory.


Browser support
----------------------------------------------

* Internet Explorer 11+ (for new features)
  * IE 8 is technically not supported anymore but we should try
    to avoid breaking it horribly until the end of 2016.
* Current and previous version of:
  * Chrome
  * Firefox
  * Safari

Mobile browsers that should render normally
----------------------------------------------

* Samsung
* Chrome
* Safari
