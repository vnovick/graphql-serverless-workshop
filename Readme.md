# Wellcome to Hasura Amsterdam workshop 

### Workshop structure:

# App Overview

## Frontend app overview

- `cd app-boilerplate`
- `yarn install`
- `yarn start`

## Backend overview

- go to [graphiql.graphql](http://graphiql.graphql-tutorials.org/) to explore API


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

Egghead video for [reference](https://egghead.io/lessons/graphql-create-your-graphql-api-auto-generated-with-hasura): 


---

# Setup Auth

- Setup `HASURA_GRAPHQL_ADMIN_KEY`
- Setup Auth0 account and new app
- Add custom rule to Auth0 app

```
/* TODO: Add real value of access key */
/* TODO: Add real endpoint value */
function(user, context, callback) {
  const userId = user.user_id;
  const nickname = user.nickname;

  request.post(
      {
        headers:
            {'content-type': 'application/json', 'x-hasura-access-key': ''},
        url: 'https://your.app.endpoint',
        body:
            `{\"query\":\"mutation($userId: String!, $nickname: String) {\\n          insert_users(\\n            objects: [{ id: $userId, name: $nickname }]\\n            on_conflict: {\\n              constraint: users_pkey\\n              update_columns: [last_seen, name]\\n            }\\n          ) {\\n            affected_rows\\n          }\\n        }\",\"variables\":{\"userId\":\"${
                userId}\",\"nickname\":\"${nickname}\"}}`
      },
      function(error, response, body) {
        console.log(body);
        callback(null, user, context);
      });
}

```
- Setup JWT Secret env variable
`HASURA_GRAPHQL_JWT_SECRET: {"type":"RS256", "key": "<the-certificate-data-in-one-line>"}`
obtain config [here](https://hasura.io/jwt-config) for domain name `graphql-tutorials.auth0.com`

- Auth solutions explained in detail [here](
  https://dev.to/hasurahq/hasura-authentication-explained-2c95)

# Add event triggers and remote schemas
  
- Add starwars API Remote schema
`https://graphql-bootcamp-swapi.herokuapp.com/`

- Add echo event trigger Lambda

- https://rin37aq9a7.execute-api.us-west-2.amazonaws.com/default/graphql-tutorials

The code looks like this:

```
// Lambda which just echoes back the event data

exports.handler = (event, context, callback) => {
    let request;
    try {
        request = JSON.parse(event.body);
    } catch (e) {
        return callback(null, {statusCode: 400, body: "cannot parse hasura event"});
    }

    let response = {
        statusCode: 200,
        body: ''
    };
    console.log(request);

    if (request.table.name === "todos" && request.event.op === "INSERT") {
        response.body = `New todo ${request.event.data.new.id} inserted, with data: ${request.event.data.new.todo}`;
    }
    else if (request.table.name === "todos" && request.event.op === "UPDATE") {
        response.body = `Todo ${request.event.data.new.id} updated, with data: ${request.event.data.new.todo}`;
    }
    else if (request.table.name === "todos" && request.event.op === "DELETE") {
        response.body = `Todo ${request.event.data.old.id} deleted, with data: ${request.event.data.old.todo}`;
    }

    callback(null, response);
};
```

