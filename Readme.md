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

- Auth solutions explained in detail [here](https://dev.to/hasurahq/hasura-authentication-explained-2c95)

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

# Excercise:

### Setup our client with Apollo:

- `yarn add apollo-boost apollo-link-ws graphql react-apollo subscriptions-transport-ws`

In src/components/App.js

```javascript
...

import ApolloClient from 'apollo-client';
import { WebSocketLink } from 'apollo-link-ws';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { ApolloProvider } from 'react-apollo';

const createApolloClient = (authToken) => {
  return new ApolloClient({
    link: new WebSocketLink({
      uri: 'ws://backend.graphql-tutorials.org/v1alpha1/graphql',
      options: {
        reconnect: true,
        connectionParams: {
          headers: {
            Authorization: `Bearer ${authToken}`
          }
        }
      }
    }),
    cache: new InMemoryCache()
  });
};


const App = ({auth}) => {
  const client = createApolloClient(auth.idToken);
  return (
    <ApolloProvider client={client}>
      <div>
        Your app
      </div>
    </ApolloProvider>
```

---

### Add Personal Todos Query

```
import gql from 'graphql-tag';
import {Query} from 'react-apollo';

```

- create GET_MY_TODOS query

```
const GET_MY_TODOS = gql`
  query getMyTodos {
    todos(where: { is_public: { _eq: false} }, order_by: { created_at: desc }) {
      id
      title
      created_at
      is_completed
  }
}`;
```

### Wrap with Query component and pass todos as props

```
  <Query query={GET_MY_TODOS}>

  </Query>
```

### Implement Add Todo mutation in TodoInput

```
const ADD_TODO = gql `
  mutation ($todo: String!, $isPublic: Boolean!) {
    insert_todos(objects: {title: $todo, is_public: $isPublic}) {
      affected_rows
      returning {
        id
        title
        created_at
        is_completed
      }
    }
  }
`;
```

> use Mutation component for that.

- use `updateCache` to update our cache on success `update={updateCache}`:

```
const updateCache = (cache, {data}) => {
    // If this is for the public feed, do nothing
    if (isPublic) {
      return null;
    }

    // Fetch the todos from the cache
    const existingTodos = cache.readQuery({
      query: GET_MY_TODOS
    });

    // Add the new todo to the cache
    const newTodo = data.insert_todos.returning[0];
    cache.writeQuery({
      query: GET_MY_TODOS,
      data: {todos: [newTodo, ...existingTodos.todos]}
    });
  };
```

add `resetInput` prop to clear and refocus your input

```
  const resetInput = () => {
    setTodoInput('');
    input.focus();
  };
```

### Add Remove and Toggle todo functionality

add remove todo

```
 const REMOVE_TODO = gql`
  mutation removeTodo ($id: Int!) {
    delete_todos(where: {id: {_eq: $id}}) {
      affected_rows
    }
  }

const removeTodo = (e) => {
    e.preventDefault();
    e.stopPropagation();
    client.mutate({
      mutation: REMOVE_TODO,
      variables: {id: todo.id},
      optimisticResponse: {},
      update: (cache, {}) => {
        const existingTodos = cache.readQuery({ query: GET_MY_TODOS });
        const newTodos = existingTodos.todos.filter(t => (t.id !== todo.id));
        cache.writeQuery({
          query: GET_MY_TODOS,
          data: {todos: newTodos}
        });
      }
    });
  };
```

- Add toggle todo

```
const TOGGLE_TODO = gql`
    mutation toggleTodo ($id: Int!, $isCompleted: Boolean!) {
      update_todos(where: {id: {_eq: $id}}, _set: {is_completed: $isCompleted}) {
        affected_rows
      }
    }
  `;

  const toggleTodo = () => {
    client.mutate({
      mutation: TOGGLE_TODO,
      variables: {id: todo.id, isCompleted: !todo.is_completed},
      optimisticResponse: {},
      update: (cache, {}) => {
        const existingTodos = cache.readQuery({ query: GET_MY_TODOS });
        const newTodos = existingTodos.todos.map(t => {
          if (t.id === todo.id) {
            return({...t, is_completed: !t.is_completed});
          } else {
            return t;
          }
        });
        cache.writeQuery({
          query: GET_MY_TODOS,
          data: {todos: newTodos}
        });
      }
    });
  };

```

# Handle Online users

- Add setInterval for online user to ping server of being online with the following mutation

```
const UPDATE_LASTSEEN_MUTATION=gql`
      mutation updateLastSeen ($now: timestamptz!) {
        update_users(where: {}, _set: {last_seen: $now}) {
          affected_rows
        }
      }`;
    this.client.mutate({
      mutation: UPDATE_LASTSEEN_MUTATION,
      variables: {now: (new Date()).toISOString()}
    });
```

- Wrap online users with Subscription component
