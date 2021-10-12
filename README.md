# Backend for shop API

# Setup
- Clone project
- Run `./setup.sh`

## Run Server
- `poetry run python manage.py runserver`

## Note:
We are using `poetry` to manage requirements. So please make sure to add
any dependency using the following command:
- `poetry add <package-name>==<version>` like `poetry add django==3.1`
This will install the package to local python env as well as write it to `poetry.lock` file

### Database Setup

- Install Postgres: `brew install postgresql@10`
- Start the db: `pg_ctl -D /usr/local/var/postgresql@10 start` or connect to psql `psql postgres`
- Create DB User: `create role shop_admin with login superuser password 'password';`
- Creaete DB: `create database shop_api owner shop_admin;`

You can ask fellow engineer for their local data dump and load that into your db:
- To export data dump: `pg_dump -U shop_admin shop_api -h localhost > exported_data.sql`
- To import this data: `psql -U shop_admin shop_api -h localhost < exported_data.sql`

### Running Celery Worker

To run celery locally: Go to project directory and run `celery -A shop_api.tasks worker -l info`

### Testing

To run tests: `poetry run ./manage.py test common.tests` 

To run with coverage:  `coverage run --branch --source='.' --omit='*migrations*' manage.py test common.tests`

`coverage report` will show the coverage data in CLI after you run the above command.

`coverage html` will generate pretty coverage files that you can browse using `open htmlcov/index.html`


### Automatic tests
Tests will automatically be run via GitHub Actions on all `push` events. To skip running tests add one 
of the following strings to the commit message:

- `[skip actions]`
- `[actions skip]`

_eg:_ `git commit -m "my commit [skip actions]"`

### Deployment

We use Elastic Beanstalk to deploy the api to AWS. 
The EC2 server notes:
- virtualenv lives in `/opt/python/run/venv/bin/`: use `source /opt/python/run/venv/bin/activate` to activate venv
- the app code lives in `/opt/python/current/app`
- ELB puts all the env variable defined in the console at `/opt/python/current/env`
- Beanstalk will also configure and start celery worker. Configuration present in `Procfile`

### References
- [ELB Config](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html)

#### Running Cron

`*/15 * * * * source /opt/python/run/venv/bin/activate && cd /opt/python/current/app/ && source /opt/python/current/env && python manage.py process_emails >> /var/log/myjob.log 2>&1`


#### Flake8
You can add a pre-commit hook so git automatically checks for flake8 issues.

Run in your terminal:
```
flake8 --install-hook git
git config --bool flake8.strict true
```

You can use autopep8 to auto-fix some issues: `autopep8 -i <file>`

The above updates the code with the fix. To just view the changes it would do run `autopep8 -d <file>`
