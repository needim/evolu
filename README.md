# Evolu

React Hooks library for [local-first apps](https://www.inkandswitch.com/local-first/) with end-to-end encrypted backup and sync using [SQLite](https://sqlite.org/) and [CRDT](https://crdt.tech/).

Evolu is designed for privacy, ease of use, and no vendor lock-in.

- SQLite in all browsers, Electron, and React Native
- E2E encrypted sync and backup with CRDT (merging changes without conflicts)
- Free Evolu server for testing (paid production-ready soon, or you can run your own)
- Typed database schema with branded types (`NonEmptyString1000`, `PositiveInt`, etc.)
- Reactive queries with React Suspense support
- Real-time experience via revalidation on focus and network recovery
- Schema evolving via `filterMap` ad-hoc migration
- No signup/login, no email collection, only Bitcoin-like mnemonic (12 words)
- Fetching nested objects and arrays in a single SQL query
- JSON support with automatic parsing and stringifying

## Local-first apps

Local-first apps allow users to own their data. Evolu stores data in the user's device(s), so Evolu apps can work offline and without a specific server. How is it different from keeping files on disk? Files are not the right abstraction for apps and are complicated to synchronize among devices. That's why client-server architecture rules the world. But as with everything, it has trade-offs.

### The trade-offs of the client-server architecture

Client-server architecture provides us with easy backup and synchronization, but all that depends on the ability of a server to fulfill its promises. Internet is offline, companies go bankrupt, users are banned, and errors occur. All those things happen all the time, and then what? Right, that's why the world needs local-first apps. But until now, writing local-first apps has been challenging because of the lack of libraries and design patterns. That's why I created Evolu.

## Requirements

- TypeScript 5.0 or newer
- The `strict` flag enabled in your `tsconfig.json` file
- The `exactOptionalPropertyTypes` flag enabled in your `tsconfig.json` file

```
{
  // ...
  "compilerOptions": {
    // ...
    "strict": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Getting Started

```
npm install evolu
```

Note Evolu uses several peer dependencies that are installed automatically with NPM and PNPM. If you are using Yarn, install them manually.

The complete Next.js example is [here](https://github.com/evoluhq/evolu/tree/main/apps/web).

### Define Data

To start using Evolu, define schemas for your database and export React Hooks.

```ts
import * as Schema from "@effect/schema/Schema";
import * as Evolu from "evolu";

const TodoId = Evolu.id("Todo");
type TodoId = Schema.To<typeof TodoId>;

const TodoTable = Schema.struct({
  id: TodoId,
  title: Evolu.NonEmptyString1000,
  isCompleted: Evolu.SqliteBoolean,
});
type TodoTable = Schema.To<typeof TodoTable>;

const Database = Schema.struct({
  todo: TodoTable,
});

export const {
  useQuery,
  useMutation,
  useOwner,
  useOwnerActions,
  useEvoluError,
} = Evolu.create(Database);
```

### Validate Data

Learn more about [Schema](https://github.com/effect-ts/schema).

```ts
import * as Schema from "@effect/schema/Schema";
import * as Evolu from "evolu";

Schema.parse(Evolu.String1000)(title);
```

### Mutate Data

Mutation API is designed for local-first apps to ensure changes are always merged without conflicts.

```ts
const { create, update } = useMutation();

create("todo", { title, isCompleted: false });
update("todo", { id, isCompleted: true });
```

### Query Data

Evolu uses type-safe TypeScript SQL query builder [kysely](https://github.com/koskimas/kysely), so autocompletion works out-of-the-box.

```ts
const { rows } = useQuery(
  (db) => db.selectFrom("todo").select(["id", "title"]).orderBy("updatedAt"),
  // (row) => row
  ({ title, ...rest }) => title && { title, ...rest },
);
```

### Protect Data

Evolu encrypts data with Mnemonic, a safe autogenerated password based on [bip39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki).

```ts
const owner = useOwner();

alert(owner.mnemonic);
```

### Delete Data

Leave no traces on a device.

```ts
const ownerActions = useOwnerActions();

if (confirm("Are you sure? It will delete all your local data."))
  ownerActions.reset();
```

### Restore Data

Restore data elsewhere. Encrypted data can only be restored with a Mnemonic.

```ts
const ownerActions = useOwnerActions();

ownerActions.restore(mnemonic).then((either) => {
  if (either._tag === "Left") alert(JSON.stringify(either.left, null, 2));
});
```

### Handle Errors

Evolu `useQuery` and `useMutation` never fail, it's the advantage of local first apps, but Evolu, in rare cases, can.

```ts
const evoluError = useEvoluError();

useEffect(() => {
  // eslint-disable-next-line no-console
  if (evoluError) console.log(evoluError);
}, [evoluError]);
```

And that's all. Minimal API is the key to a great developer experience.

## Privacy

Evolu uses end-to-end encryption and generates strong and safe passwords for you. Evolu sync and backup server see only timestamps.

## Trade-offs

> “There are no solutions. There are only trade-offs.” ― Thomas Sowell

Evolu is not P2P. For reliable syncing and backup, there needs to be a server. Evolu server is very minimal, and everyone can run their own. While it's theoretically possible to have P2P Evolu, I have yet to see a reliable solution. It's not only a technical problem; it's an economic problem. Someone has to be paid to keep your data safe. Evolu provides a free server for testing. Soon we will provide a paid server for production usage.

All table columns except for ID are nullable by default. It's not a bug; it's a feature. Local-first data are meant to last forever, but schemas evolve. This design decision was inspired by GraphQL [nullability](https://graphql.org/learn/best-practices/#nullability) and [versioning](https://graphql.org/learn/best-practices/#versioning). Evolu provides a handy `filterMap` helper for queries.

## Community

The Evolu community is on GitHub Discussions, where you can ask questions and voice ideas.

To chat with other community members, you can join the [Evolu Discord](https://discord.gg/2J8yyyyxtZ).

[![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/evoluhq.svg?style=social&label=Follow%20%40evoluhq)](https://twitter.com/evoluhq)

## FAQ

### Is Evolu ready for production?

It should be. The CRDT message format is stable.

### What is the SQLite database size limit?

[Storage quotas and eviction criteria](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)

### How can I check the current database filesize?

Use OPFS Explorer Chrome DevTools extension.

## Contributing

Evolu monorepo uses [pnpm](https://pnpm.io).

Install the dependencies with:

```
pnpm install
```

Build Evolu monorepo:

```
pnpm build
```

Start developing and watch for code changes:

```
pnpm dev
```

Lint and tests:

```
pnpm lint test
```

Describe changes for release log:

```
pnpm changeset
```
