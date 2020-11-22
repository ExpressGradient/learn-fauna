# Overview
Fauna GraphQL API supports Boolean, Date, Float, ID, Int, Long, String and Time Scalar Types.  
ID is a string representing a generic identifier. It is not intended to be human readable.  
Fauna already provides a unique identifier field for a document via the _id field, which represents the document's reference.  
You would typically use the ID type for documents that have an externally created identifier, such as documents imported from another database.  

## Limitations
FaunaDB GraphQL API currently doesn't support some features.  
Schemas do not support custom directives, custom interfaces, custom scalars and union types.  
No name can start with an underscore.  
Subscriptions are not supported.  

# GraphQL Endpoints
1. https://graphql.fauna.com/graphql accepts GraphQL queries and returns results in JSON format.

2. https://graphql.fauna.com/import accepts a GraphQL schema definition, which is translated into the equivalent FaunaDB collections and indexes.

When a GraphQL schema is imported, the GraphQL API automatically creates the following FaunaDB schema documents.

* One collection for each declared GraphQL type(collection's name = type's name), but not for embedded types. You can specify the collection name using the `@collection` directive.

* One index for each declared query(index's name = query name), but not for queries annotated with `@resolver` directive. You can specify the index name using the `@index` directive.

* When a `@relation` directive is used and the relationship is many-to-many, an associative collection is created, as well as three indexes, one for each side of the relationship and a third indexing the refs of both sides.

* One user-defined function for each mutation declared with `@resolver` directive, using the resolver name. The implementation of the mutation function is given via FQL.

## Modes
There are two modes of import: `merge` and `override`.  
In `merge` mode, the schema is updated with the new schema changes, new collections, indexes and functions are made.  
In `override` mode, the schema is deleted and updated with the new schema, existing collections, documents, indexes and functions are deleted.

## Authentication
Both endpoints require authentication with a specific Fauna database. This is achieved with a standard Fauna secret, which determines the database and permissions to be used.  

The secret can be provided through the HTTP `Authentication` request header, either as a `Bearer` token or via `HTTP Basic` authentication.

# Directives
## `@collection`
Use this directive to specify the name of a collection. By default, the collection name would be the type name.

## `@embedded`
Embedded types do not have their own associative collection; the data for an embedded type is stored within the parent type.

## `@index`
Use this directive to specify the name of an index. By default, the index name would be the query field name.

## `@relation`
Use this directive to specify a certain field is participating in a bi-directional relationship with the target type.

## `@resolver`
Use this directive to specify a certain query field is using a resolver, using the name property of the directive.  
Implementation of resolvers should be given via FQL.

## `@unique`
Use this directive to specify a certain field is unique. Unique fields have an associated index within the database. Use the index property on the directive to specify the index's name.

# Input Types
Use Input Types to convey complex arguments to queries or field resolvers, or simply to estabilish a common name for frequently used arguments.  

The Fauna GraphQL API automatically creates input types for user-defined types. For example, when you create a `User` type, the input type `UserInput` is automatically created for you.  

The Fauna GraphQL API also creates the mutations `createUser`, `updateUser` and `deleteUser` automatically. The first two use this auto-created `UserInput` input type.

# Relations
During the schema import, the GraphQL API creates the necessary collections and indexes to correlate the data based on the recognized relationships in the schema.  

Relationships can be made explicit by adding the `@relation` directive with the same relation name, at both fields involved in the relationship.

## One-to-one
A one-to-one relationship puts a low cardinality constraint in a relationship between two types.

## One-to-many
A one-to-many relationship has high cardinality at only one side of the relationship. In schema, the source has an array field that points to the target type, while target has a non-array field that points back to the source type.

## Many-to-one
A many-to-one relationship works in the same way as one-to-many. But array field can be omitted.  

The target is unaware is of its association with the source. The underlying database components and API behaviour are identical.

## Many-to-many
A many-to-many relationship has high cardinality on both sides of the relationship. The fields in the relationship both should be arrays.

## Combining multiple relationships
When combining multiple relationships use explicit `@relation` directive.

## Relational mutations
To run mutations on types containing relationships use relational mutations. There are three types of relational mutations: `create`, `connect` and `disconnect`.

## `create`
Use `create` mutation to create documents that are associated with the main target of the mutation.

## `connect`
Use `connect` mutation to connect to an existing document using the document's id.

## `disconnect`
`disconnect` is just the opposite of connect, allows you to disconnect a document using the document id.

...Some basic stuff about UDFs.