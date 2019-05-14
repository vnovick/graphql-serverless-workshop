# Wellcome to Architecting Scalable Serverless Architectures with GraphQL Api workshop

### Workshop structure:

# App Overview

## Frontend app overview

- `cd app-boilerplate`
- `yarn install`
- `yarn start`

## Backend overview

- go to [graphiql.graphql](https://learn.hasura.io/graphql/graphiql) to explore API


# Setting up Hasura

- Go to [hasura.io](hasura.io)
- Click on `Deploy to Heroku button`
- Enter your app name
- Click on Open app


## Excercise: 

Setup Hasura with the following data structure:

- todos table
  - id - integer - primary key 
  - title - text
  - is_completed - boolean, default: false
  - created_at - timestamp
  - is_public - boolean
  - user_id - text - foreign key to users.id

> relationships - object relationship user
- users table
  - id - text - primary key 
  - name - text 
  - created_at - timestamp - now() 
  - last_seen - timestamp - nullable

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

Egghead video for [reference](https://egghead.io/lessons/graphql-create-your-graphql-api-auto-generated-with-hasura): 
