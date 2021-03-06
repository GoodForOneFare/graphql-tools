# `graphql-typescript-definitions`

> Generate TypeScript definition files from .graphql documents

## Installation

```
npm install graphql-typescript-definitions --save-dev
```

or, with Yarn:

```
yarn add graphql-typescript-definitions --dev
```

## Usage

This package will generate matching `.d.ts` files for each `.graphql` file you specify. It will attempt to generate the type as smartly as possible, including the re-use of interfaces for any fragment spreads used in your documents. The following types are generated:

* A default export for the type that will be generated by a GraphQL loader (GraphQL’s `DocumentNode` type).

* An interface for each query, mutation, and fragment, named `<OpertionName><Query | Mutation | Fragment>Data`. For example, `query Home {}` becomes `export interface HomeQueryData {}`.

* A namespace for each operation that includes any nested types. For example, if we imagine the following GraphQL schema (using the [GraphQL IDL](https://www.graph.cool/docs/faq/graphql-idl-schema-definition-language-kr84dktnp0/)):

  ```graphql
  type Person {
    name: String!
    relatives: [Person!]!
  }

  type Query {
    person: Person
  }
  ```

  and the following query:

  ```graphql
  query Someone {
    person {
      name
    }
  }
  ```

  The following exports would be generated:

  ```typescript
  export interface SomeoneQueryData {
    person?: SomeoneQueryData.Person | null,
  }

  export namespace SomeoneQueryData {
    export interface Person {
      name: string,
    }
  }
  ```

  This allows you to use the full query’s type, as well as any of the subtypes that make up that query type. This is particularly useful for list or nullable types, where you may need to directly the access the underlying type:

  ```typescript
  import someoneQueryDocument, {SomeoneQueryData} from './Someone.graphql';

  let data: SomeoneQueryData;
  let person: SomeoneQueryData.Person;
  ```

### CLI

```sh
yarn run graphql-typescript-definitions 'src/**/*.graphql' -- --schema-path 'build/schema.json'
```

Optionally, you can pass the `--watch` flag in order to regenerate the TypeScript definition files on changes to the GraphQL files.

### Node

```js
const {Builder} = require('graphql-typescript-definitions');

const builder = new Builder({
  graphQLFiles: 'src/**/*.graphql',
  schemaPath: 'build/schema.json',
});

builder.on('build', (build) => {
  // See the source file for details on the shape of the object returned here
  console.log(build);
});

builder.on('error', (error) => {
  console.error(error);
});

// Optionally, you can pass {watch: true} here to re-run on changes
builder.run();
```
