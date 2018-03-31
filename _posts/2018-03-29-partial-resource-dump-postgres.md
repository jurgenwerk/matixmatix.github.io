---
category: posts
layout: single
mainTopic: 'programming'
title: "Partial, resource-related dumps in PostgreSQL"
# header:
#   image: "http://codeandtechno.com/images/lambda-post/cloudfront-map.png"
---

Sometimes we find ourselves in a need to copy and restore a specific set of data belonging to a common parent resource. For example, we want to connect to the production server, pull down fresh data related only to the specified user and restore it to our development database to debug an issue or just develop new features. When your database is tiny, you could just restore the whole thing, but once it grows up, the complete restoration is out of the question, especially when you need to do it often.

Here’s a bash script I wrote that downloads everything belonging to the application user from the production database, deletes stale user data from the local development database and replaces it with the fresh data.

```bash
#!/bin/sh
# Usage: ./script/restore_user 100
USER_ID=$1
DATE=`date +%Y-%m-%d`
SQL_FILENAME=data_user_${USER_ID}_${DATE}.sql

echo "-- Preparing a data dump file for the specified user..."
ssh admin@db.myapp.com /bin/sh <<EOT > $SQL_FILENAME
  echo "Copying user..." >&2
  echo "COPY users FROM STDIN;"
  psql -h db.myapp.com -U myusername mydatabase -c "COPY (
    SELECT users.*
    FROM users
    WHERE id = ${USER_ID}
  ) TO STDOUT;"
  echo "\\."

  echo "Copying user’s posts..." >&2
  echo "COPY posts FROM STDIN;"
  psql -h db.myapp.com -U myusername mydatabase -c "COPY (
    SELECT posts.*
    FROM posts
    WHERE user_id = ${user_id}
  ) TO STDOUT;"
  echo "\\."

  echo "Copying post’s tags ..." >&2
  echo "COPY tags FROM STDIN;"
  psql -h db.myapp.com -U myusername mydatabase -c "COPY (
    SELECT tags.*
    FROM tags
    INNER JOIN posts ON posts.id = tags.post_id
    WHERE posts.user_id = ${user_id}
  ) TO STDOUT;"
  echo "\\."
EOT
```

Let’s comment on what’s happening here. What we’re doing is essentially using Postgres’s `COPY` function which is useful for moving data around tables. Usually it’s used to produce individual CSV files separately for each table, which are then individually restored using `COPY FROM` command.

Since this function is also able to read from the `STDIN`, we bundle all `COPY` commands together (using the  `.\` which indicates the end of the input). Local PostgreSQL will then see sequences of `COPY FROM STDIN`, then data, then `.\`, indicating it’s done. We do this to avoid making multiple table-specific CSV files. Instead, we just put everything into one big SQL file.

This was just the first part of the script. Now that we have the necessary data in an SQL file, we have to make place for it and restore it to our local database.

```bash
echo "-- Deleting old user data..."
psql -d mydb_dev -c "
  DELETE FROM tags WHERE tags.id IN (
    SELECT tags.id
    FROM tags
    INNER join posts ON posts.id = tags.post_id
    WHERE user_id = ${user_id}
  );

  DELETE FROM posts where user_id = ${user_id};
  DELETE FROM users where id = ${user_id};

echo "-- Restoring fresh user data..."
psql -d mydb_dev < ${SQL_FILENAME}

echo "-- Deleting temp data..."
rm ${SQL_FILENAME}

echo "-- Done. Enjoy."
```

Since this is only a partial dump related to a common resource or ancestor (application user, in this case), we need to make sure we also have all other data in our development database that the application needs to run and doesn't belong to the user. Depending on your setup, you could easily restore those tables using `pg_dump` command where you explicitly specify the tables. But when you need to refresh only a subset of local data, a script like this comes in really handy.
