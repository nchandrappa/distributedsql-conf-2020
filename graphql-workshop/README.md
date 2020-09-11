# react-static-graphql

A sample app to get started with [react-static](https://github.com/nozzle/react-static) site generator, Hasura GraphQL engine and YugabyteDB YSQL as database.

[![Edit react-static](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/github/hasura/graphql-engine/tree/master/community/sample-apps/react-static-graphql?fontsize=14)

# Tutorial


- Clone this repo:
  ```bash
  git clone https://github.com/nchandrappa/distributedsql-conf-2020
  cd distributedsql-conf-2020/graphql-workshop/
  ```

- Deploy YugabyteDB on Kubernetes using helm 3:

```
$ kubectl create namespace yb-demo

$ helm install yb-demo-database yugabytedb/yugabyte \
--set resource.master.requests.cpu=1,resource.master.requests.memory=1Gi,\
resource.tserver.requests.cpu=2,resource.tserver.requests.memory=2Gi --namespace yb-demo
```
  Please checkout our [docs](https://docs.yugabyte.com/latest/deploy/kubernetes/) for other kubernetes deployment methods


- Deploy Hasura on Kubernetes

```
$ kubectl apply -f kubernetes/hasura-deployment.yaml

$ kubectl apply -f kubernetes/hasura-svc.yaml

```

Note the external IP for Hasura LoadBalancer after deploying Hasura GraphQL engine,

```
$ kubectl get svc hasura

NAME                  EXTERNAL-IP       PORT(S)
hasura                35.224.XX.XX      7001:31386/TCP
```

- Create `author` table:
  
  Open Hasura console: visit https://loadbalancer-ip:7001/console on a browser  
  Navigate to `Data` section in the top nav bar and create a table as follows:

  ![Create author table](./assets/add_table.jpg)

- Similarly, create an article table with the following data model:
table: `article`
columns: `id`, `title`, `content`, `author_id` (foreign key to `author` table's `id`)

  ![Create foreign key for author_id column to author's id](./assets/author_fk.png)

- Now create a relationship from article table to author table by going to the Relationships tab.


- Insert sample data into `author` and `article` table:

```
$ kubectl exec -n yb-demo -it yb-tserver-0 -- ysqlsh -h yb-tserver-0.yb-tservers.yb-demo

```
Copy the YSQL statements below into the shell and press Enter.

```
INSERT INTO author(name) VALUES ('John Doe'), ('Jane Doe'); 
INSERT INTO article(title, content, rating, author_id) 
VALUES ('Jane''s First Book', 'Lorem ipsum', 10, 2);
INSERT INTO article(title, content, rating, author_id) 
VALUES ('John''s First Book', 'dolor sit amet', 8, 1);
INSERT INTO article(title, content, rating, author_id) 
VALUES ('Jane''s Second Book', 'consectetur adipiscing elit', 7, 2);
INSERT INTO article(title, content, rating, author_id) 
VALUES ('Jane''s Third Book', 'sed do eiusmod tempor', 8, 2);
INSERT INTO article(title, content, rating, author_id) 
VALUES ('John''s Second Book', 'incididunt ut labore', 9, 1);
SELECT * FROM author ORDER BY id;
SELECT * FROM article ORDER BY id;
```

  Verify if the row is inserted successfully

  ![Insert data into author table](./assets/browse_rows.jpg)



- Install npm modules:
  ```bash
  npm install
  ```

- Open `src/apollo.js` and configure Hasura's GraphQL Endpoint as follows: 
  ```js

    import { ApolloClient } from 'apollo-client'
    import { HttpLink } from 'apollo-link-http'
    import { InMemoryCache } from 'apollo-cache-inmemory'
    import fetch from 'node-fetch'

    const client = new ApolloClient({
      link: new HttpLink({
        uri: 'http://loadbalancer-ip/v1/graphql',
        fetch
      }),
      cache: new InMemoryCache(),
    })

    export default client

  ```
Replace the `uri` argument with your Hasura GraphQL Endpoint.

- We have defined the graphql query in `src/graphql/queries/queries.js`. 
    - GraphQL query to fetch author information

    ```graphql

    query {
      author {
        id
        name
      }
    }

    ```

    - GraphQL query to fetch articles written by author

    ```graphql

    query($author: Int!) {
        article(where: {author_id: {_eq: $author}}) {
            id
            title
            content
            created_at
        }
    }

    ```

- In `pages/blog.js`, we do the templating for listing all the authors and in `containers/Post.js`, we do the templating for listing all the articles written by a selected author.

- Run the app:
  ```bash
  yarn start
  ```
- Test the app
  Visit [http://localhost:3000](http://localhost:3000) to view the app

- Bundle app
  ```bash
  yarn stage
  ```
- Serve Production app
  ```bash
  yarn serve
  ```

For detailed explanation on how things work, checkout [react-static docs](https://github.com/nozzle/react-static).
