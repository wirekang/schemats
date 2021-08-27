# Schemats

Before anything, I would like to give a massive thank you to [sweetiq](https://www.npmjs.com/package/schemats) and their contributors for giving me a huge head start.

The reason I have created a new repo instead of a fork is because I don't support mysql and have some breaking changes due to how this library is consumed by [postgres-typed](https://github.com/vramework/postgres-typed) and [vramework](https://vramework.io/).

I have kept the name and based off their MIT license as means of attribution and thanks.

## Why Schemats

Because being able to make a change to your database structure and have it:

- validate through your node backend APIs
- get verified against automatically generate JSON schemas
- raise errors in your frontend application 

Is just a great developer experience in my opinion.

This allows us to some pretty amazing things when it comes to refactoring and maintaining codebases, and
also provide the meta-data to help with libraries like [postgres-typed](https://github.com/vramework/postgres-typed). 

## Quickstart

### Installing

```bash
yarn add -d @vramework/schemats || npm install -d @vramework/schemats
```

### Generating the type definition from schema

Assuming you have the following schema (this is a bit of a random one):

```sql
CREATE SCHEMA "pet_store";

CREATE TYPE "pet_store"."animal" AS enum (
  'cat',
  'dog'
);

CREATE TABLE "pet_store"."user" (
  "uuid" uuid PRIMARY KEY default gen_random_uuid(),
  "name" text NOT NULL
);

CREATE TABLE "pet_store"."pet" (
  "uuid" uuid PRIMARY KEY default gen_random_uuid(),
  "owner" uuid REFERENCES "pet_store"."user",
  "type" pet_store.animal NOT NULL,
  "name" text NOT NULL,
  "birthdate" date,
  "last_seen_location" point,
  "random_facts" jsonb
);
COMMENT ON COLUMN pet_store.pet.random_facts is '@type {RandomPetFacts}';
```

You can now generate a bunch of different schema definitions.

My personal favourite is the following:

```bash
schemats postgres postgres://postgres@localhost/database -f ./db-custom-types.ts -s pet_store -c -e -o db-types.ts
```

While will result in the following typescript file: 

```typescript

/**
 * AUTO-GENERATED FILE @ Fri, 27 Aug 2021 08:26:50 GMT - DO NOT EDIT!
 *
 * This file was automatically generated by schemats v.0.0.8
 * $ schemats generate postgres://username:password@localhost:5432/schemats -C -s pet_store
 *
 */

import { RandomPetFacts } from './db-custom-types'



export enum Animal {
	Cat = 'cat',
	Dog = 'dog' 
}

export interface User { 
	uuid: string
	name: string 
}

export interface Pet { 
	uuid: string
	owner?: string | null
	type: Animal
	name: string
	birthdate?: Date | null
	lastSeenLocation?: { x: number, y: number } | null
	randomFacts?: RandomPetFacts | null
	moreRandomFacts?: unknown | null 
}

export interface Tables {
    user: User,
	pet: Pet
}

export type CustomTypes = RandomPetFacts
```

But you have quite a bit of flexbility:

```bash
Usage: schemats postgres [options] [connection]

Generate a typescript schema from postgres

Arguments:
  connection                   The connection string to use, if left empty will use env variables

Options:
  -s, --schema <schema>        the schema to use (default: "public")
  -t, --tables <tables...>     the tables within the schema
  -f, --typesFile <typesFile>  the file where jsonb types can be imported from
  -c, --camelCase              use camel case for enums and table names
  -e, --enums                  use enums instead of types
  -o, --output <output>        where to save the generated file relative to the current working directory
  --no-header                  don't generate a header
  -h, --help                   display help for command
```

## Features

### Camel Case

This automatically turns all your tables and Enums / Types to camelcase, which is the default
experience for javascript and is more consistent to use

### Enums

Using enums turns all postgres enums into Enums instead of normal types, which is just a
preference aspect for developers since renaming enum values or order will change the Enum
key and value.

### Types File

This is a VERY useful feature for jsonb fields. Normally a jsonb field type is unknown, 
however if you provide a types json file this will get the type out of the comment 
of a field and assign it to the value.

The structure of a custom type file could either be from another file:

```typescript
export type { RandomPetFacts }  from './somewhere-else'
```

or it could just be defined straight in the file.

```typescript
export type RandomPetFacts = Record<string, string>
```

### Tables | Custom Types

These types are automatically generated to power typed-postgres

## Using in typescript

You can import all your interfaces / enums from the file:

```typescript
import * as DB from './db-types'

// And then you can start picking how you want your APIs to be used:
type updatePetLocation = Pick<DB.Pet, 'lastSeenAt'>
```

## Tests

So where are the tests? The original schemats library has an amazing 100% coverage and this one has 0.

To be honest, I'm using this library in a few of my current projects and any error in it throws dozens 
in the entire codebase, so it sort of tests itself. That being said I will be looking to add some in again,
but in terms of priorties not my highest.

However for manual testing and experimenting you can easily replicate this project by:

```bash
# Clone the repo
git clone git@github.com:vramework/schemats.git
# Enter repo
cd schemats
# Install dependencies
yarn install
# Run the example, which will run create the schemats library and generate the db-types library
yarn run example
```

