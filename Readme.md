# Wellcome to Hasura Amsterdam workshop 

### Workshop structure:

# Part 1 - App Overview

## Frontend app overview

- `cd app-boilerplate`
- `yarn install`
- `yarn start`

## Backend overview

- go to [graphiql.graphql](http://graphiql.graphql-tutorials.org/) to explore API

-----

# Part 2 - Setting up Hasura

- Go to [hasura.io](hasura.io)
- Click on `Deploy to Heroku button`
- Enter your app name
- Click on Open app

------

# Excercise: Setup Hasura with the following data structure:

- todos table
  - id - integer - primary key 
  - title - text
  - is_completed - boolean
  - created_at - timestamp
  - is_public - boolean
  - user_id - text - foreign key to users.id

> relationships - object relationship user
- users table
  - id - primary key 
  - name - text - nullable 
  - created_at - text - now() 
  - last_seen - text - now() 

> relationships - array relationship todos

- online_users 
```
CREATE OR REPLACE VIEW "public"."online_users" AS 
 SELECT users.id,
    users.last_seen
   FROM users
  WHERE (users.last_seen >= (now() - '00:00:30'::interval));
```

> relationships - object relationship user
