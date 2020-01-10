![shhh](https://github.com/smallwat3r/shhh/blob/master/shhh/static/img/logo.png)  

Shhh is a tiny Flask app to write encrypted secrets and share them with people using a secure link.  

The sender has to set up an expiration date along with the passphrase to access the secret. After this date the secret will be removed from the database.  

The secrets are encrypted in order to make the data anonymous, especially in MySQL.  
_Encryption method used: Fernet with password, random salt value and strong iteration count (100 000)._  

**demo:**    
![shhh](https://i.imgur.com/xI7uvfK.gif)

## ⚙️ Set up & Dependencies

### MySQL

Create a MySQL database and run the following script to generate the table `links` that will store our data.

```sql
CREATE TABLE `links` (
  `slug_link` text,
  `encrypted_text` text,
  `date_created` datetime DEFAULT NULL,
  `date_expires` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

In MySQL run the following to activate the `event_scheduler`. We need to make sure the `event_scheduler` is activated to schedule database clean-ups, in order to remove the records that have expired. In order to persist this setting in the event the MySQL server is restarted, you will need to either

1. adjust your MySQL Server's `my.cnf` or `my.ini` file, or
2. adjust the command that is used to start the MySQL Server instance, adding the `--event_scheduler=on` option

```sql
SET GLOBAL event_scheduler = ON;
```

Then we need to schedule a run at least every 2 hours to remove our expired records.

```sql
CREATE EVENT AutoDeleteExpiredRecords
ON SCHEDULE
EVERY 2 HOUR
DO 
  DELETE FROM links WHERE date_expires <= now();
```

These MySQL queries can also be executed against the MySQL server instance via
the `mysql/initialize.sql` file.

### Launch Shhh

#### Natively Using Flask

We recommend that you create a virtual environment for this project, so you can
install the required dependencies.

```sh
virtualenv -p python3 venv --no-site-package
source venv/bin/activate
pip install -r requirements.txt
```

You then need to set up a few environment variables. These will be used to
configure Flask, as well as the app's connection to an instance of MySQL.

```sh
export FLASK_APP=shhh
export FLASK_ENV=<development/production>
export HOST_MYSQL=<localhost>
export USER_MYSQL=<username>
export PASS_MYSQL=<password>
export DB_MYSQL=<name>
```

Finally, run the below command to launch Shhh on http://localhost:5000/.

```sh
python3 -m flask run --host='0.0.0.0'
```

#### Docker Compose

For development instances of Shhh, this repo contains a docker-compose
configuration. The configuration defines default settings for Shhh, as well as
some default settings for a containerized instance of MySQL server. To build and
run Shhh via docker-compose:

```sh
docker-compose up -d --build app
```

or via Makefile:

```sh
make dc-start    # start app
                 
                 # other commands
                 # --------------
make dc-stop     # stop app
make dc-reboot   # reboot app
make dc-cleanup  # clean
```

Once the container image has finished building and starting, Shhh will be
available via http://localhost:5000/.

## 💡 Idea credits  

- [OneTimeSecret](https://github.com/onetimesecret/onetimesecret)
- [PasswordPusher](https://github.com/pglombardo/PasswordPusher)
