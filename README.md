# gatsby-firestore [![npm](https://img.shields.io/npm/v/@martinreiche/gatsby-firestore)](https://www.npmjs.com/package/@martinreiche/gatsby-firestore) ![node](https://img.shields.io/node/v/firebase-admin)

Gatsby plugin for connecting [Firebase Firestore](https://firebase.google.com/products/firestore)
as a data source. Supports subcollections.

## Usage

1. Generate and download a Firebase Admin SDK private key by accessing the
   [Firebase Project Console > Settings > Service Accounts](https://console.firebase.google.com/project/_/settings/serviceaccounts/adminsdk)

2. Rename and put the downloaded `.json` crendtial file somewhere in the
   GatsbyJS project (e.g. `./credentials.json`)

3. Add `gatsby-source-firestore` as a dependency by running using `npm` or `yarn`:

   ```sh
   npm i @martinreiche/gatsby-firestore
   # or
   yarn add @martinreiche/gatsby-firestore
   ```

4. Configure settings at `gatsby-config.js`, for example:

   ```js
   module.exports = {
     plugins: [
       {
         resolve: `@martinreiche/gatsby-firestore`,
         options: {
           // credential or appConfig
           credential: require(`./credentials.json`),
           appConfig: {
             apiKey: 'api-key',
             authDomain: 'project-id.firebaseapp.com',
             databaseURL: 'https://project-id.firebaseio.com',
             projectId: 'project-id',
             storageBucket: 'project-id.appspot.com',
             messagingSenderId: 'sender-id',
             appID: 'app-id',
           },
           types: [
             {
               type: `Book`,
               collection: `books`,
               map: doc => ({
                 title: doc.title,
                 isbn: doc.isbn,
                 author___NODE: doc.author.id,
               }),
             },
             {
               type: `Author`,
               collection: `authors`,
               map: doc => ({
                 name: doc.name,
                 country: doc.country,
                 books___NODE: doc.books.map(book => book.id),
               }),
             },
           ],
         },
       },
     ],
   };
   ```

   Note that you will need to have `books` and `authors` in Firestore matching
   this schema before Gatsby can query correctly, e.g `books__NODE` on `author`
   needs to be an array with `books` as a key of reference types to `book`
   documents.

   Test GraphQL query:

   ```graphql
   {
     allBooks {
       edges {
         node {
           title
           isbn
           author {
             name
           }
         }
       }
     }
   }
   ```

## Support for subcollections

To query subcollections, you have to specify the subCollection Array in the types configuration.
They can be nested infinitely deep as long as they exist in Firestore. For example, if `books` were
a subcollection of `authors` in Firestore, you could do the following:

```js
module.exports = {
  plugins: [
    {
      resolve: `@martinreiche/gatsby-firestore`,
      options: {
        // credential or appConfig
        credential: require(`./credentials.json`),
        appConfig: {
          apiKey: 'api-key',
          authDomain: 'project-id.firebaseapp.com',
          databaseURL: 'https://project-id.firebaseio.com',
          projectId: 'project-id',
          storageBucket: 'project-id.appspot.com',
          messagingSenderId: 'sender-id',
          appID: 'app-id',
        },
        types: [
          {
            type: `Author`,
            collection: `authors`,
            map: doc => ({
              name: doc.name,
              country: doc.country,
            }),
            subCollections: [
              {
                type: `Book`,
                collection: `books`,
                map: doc => ({
                  title: doc.title,
                  isbn: doc.isbn,
                }),
              },
            ],
          },
        ],
      },
    },
  ],
};
```

`books` now become children of `author` and you can query them like this:

```graphql
{
  allAuthor {
    edges {
      node {
        name
        childrenBook {
          title
          isbn
        }
      }
    }
  }
}
```

## Configurations

| Key                    | Description                                                                                                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `credential`           | Credential configurations from downloaded private key                                                                                                              |
| `types`                | Array of types, which require the following keys (`type`, `collection`, `map`)                                                                                     |
| `types.type`           | The type of the collection, which will be used in GraphQL queries, e.g. when `type = Book`, the GraphQL types are named `book` and `allBook`                       |
| `types.collection`     | The name of the collections in Firestore. Supports nested collections like `authors/<authorId>/books` where `<authorId>` is the ID of one specific `author` record |
| `types.map`            | A function to map your data in Firestore to Gatsby nodes                                                                                                           |
| `types.subCollections` | Optional: Array of types for the current type --see `types`                                                                                                        |

## Acknowledgement

- [ryanflorence/gatsby-source-firebase](https://github.com/ryanflorence/gatsby-source-firebase)
- [taessina/gatsby-source-firestore](https://github.com/taessina/gatsby-source-firestore)
