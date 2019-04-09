### Backend links:

1. GraphiQL: http://graphiql.graphql-tutorials.org
2. Hasura: http://backend.graphql-tutorials.org

### To setup Apollo:

In src/components/App.js

```javascript
...
import OnlineUsersWrapper from './OnlineUsers/OnlineUsersWrapper';

import ApolloClient from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { ApolloProvider } from 'react-apollo';

const createApolloClient = (authToken) => {
  return new ApolloClient({
    link: new HttpLink({
      uri: 'http://backend.graphql-tutorials.org/v1alpha1/graphql',
      headers: {
        Authorization: `Bearer ${authToken}`
      }
    }),
    cache: new InMemoryCache()
  });
};

const App = ({auth}) => {
  const client = createApolloClient(auth.idToken);
  return (
  ...
 
```

