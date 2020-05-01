Simple heroku app with a bash script for capturing heroku database backups and copying to your s3 bucket.  Deploy this as a separate app within heroku and schedule the script to backup your production databases which exist within another heroku project.


## Installation


First, clone this project, then change directory into the newly created directory:

```
git clone https://github.com/kbaum/heroku-database-backups.git
cd heroku-database-backups
```

Create a project on heroku.

```
heroku create mc-ops-database-backups
```
Add the heroku-buildpack-cli:

```
heroku buildpacks:add https://github.com/heroku/heroku-buildpack-cli -a  mc-ops-database-backups
```

Next push this project to your heroku projects git repository.

```
heroku git:remote -a mc-ops-database-backups
git push heroku master
```

Now we need to set some environment variables in order to get the heroku cli working properly using the [heroku-buildpack-cli](https://github.com/heroku/heroku-buildpack-cli).

```
heroku config:add HEROKU_API_KEY=`heroku auth:token` -a mc-ops-database-backups
```

This creates a token that will quietly expire in one year. To create a long-lived authorization token instead, do this:

```
heroku config:add HEROKU_API_KEY=`heroku authorizations:create -S -d mc-ops-database-backups` -a mc-ops-database-backups
```

Next we need to add the amazon key and secret from the IAM user that you are using:

```
heroku config:add AWS_ACCESS_KEY_ID=123456 -a mc-ops-database-backups
heroku config:add AWS_DEFAULT_REGION=us-east-1 -a mc-ops-database-backups
heroku config:add AWS_SECRET_ACCESS_KEY=132345verybigsecret -a mc-ops-database-backups
```

And we'll need to also set the bucket and path where we would like to store our database backups:

```
heroku config:add S3_BUCKET_PATH=my-db-backup-bucket/backups -a mc-ops-database-backups
```  
Be careful when setting the S3_BUCKET_PATH to leave off a trailing forward slash.  Amazon console s3 browser will not be able to locate your file if your directory has "//" (S3 does not really have directories.).

Finally, we need to add heroku scheduler and call [backup.sh](https://github.com/kbaum/heroku-database-backups/blob/master/bin/backup.sh) on a regular interval with the appropriate database and app.

```
heroku addons:create scheduler -a mc-ops-database-backups
```

Now open it up, in your browser with:

```
heroku addons:open scheduler -a mc-ops-database-backups
```

And add the following command to run as often as you like:

```
APP=your-app DATABASE=HEROKU_POSTGRESQL_NAVY_URL /app/bin/backup.sh
```

In the above command, APP is the name of your app within heroku that contains the database.  DATABASE is the name of the database you would like to capture and backup.  In our setup, DATABASE actually points to a follower database to avoid any impact to our users.  Both of these environment variables can also be set within your heroku config rather than passing into the script invocation.


