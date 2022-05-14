<!--
SPDX-FileCopyrightText: 2021 Andre 'Staltz' Medeiros

SPDX-License-Identifier: CC0-1.0
-->

:warning: **This repo was moved to https://github.com/ssbc/ssb-index-feed-writer.** This archival will remain in this GitHub org `ssb-ngi-pointer` to demonstrate the outcome of the work done by the SSB NGI Pointer team during 2020 and 2021. The SSB NGI Pointer team is no longer active because we completed our grant project.

# ssb-index-feed-writer

A secret-stack plugin that uses [ssb-meta-feeds](https://github.com/ssb-ngi-pointer/ssb-meta-feeds)
to create and update [index feeds](https://github.com/ssb-ngi-pointer/ssb-secure-partial-replication-spec#indexes).

## Installation

**Prerequisites:**

- Requires **Node.js 10** or higher
- Requires `ssb-db2`
- Requires `ssb-meta-feeds`

```
npm install --save ssb-index-feed-writer
```

Add this plugin like this:

```diff
 const sbot = SecretStack({ appKey: caps.shs })
     .use(require('ssb-db2'))
     .use(require('ssb-meta-feeds'))
+    .use(require('ssb-index-feed-writer'))
     // ...
```

## Example usage

There is one primary API this plugin provides: `start()`. You pass it a
`ssb-ql-0` query and it will detect whether it needs to create an index feed for
that query, or update an existing index feed.

```js
sbot.indexFeedWriter.start({ author: sbot.id, type: 'vote', private: false }, (err, index) => {
  console.log('The index feed is ' + index.subfeed)
})
```

From this point onwards, whenever the query gets more results (as the log gets
appended), the index feed will be automatically written to.

To stop the index feed from being written to, use `stop()` with the same query
input.

## API

### `sbot.indexFeedWriter.start(query, cb)` (async)

_Begins updating an index feed for the given query_

`query` must be an `ssb-ql-0` query, either as stringified JSON or as an object.

The callback `cb` will be called right before the index feed would be written
to. If there are no updates to be written on the index feed, this callback will
anyway be called. The 1st argument is the possible error, and the 2nd argument
is the "subfeed object" containing details on the index feed that matches the
query. The subfeed object has the shape `{ feedpurpose, subfeed, keys, metadata }`,
the same shape as returned by `ssb-meta-feeds` APIs.

### `sbot.indexFeedWriter.doneOld(query, cb)` (async)

_Informs you when the index feed for the given query has processed the existing database_

`query` must be an `ssb-ql-0` query, either as stringified JSON or as an object.

The callback `cb` will be called as soon as the *writing* of the index feed has
finished processing all "old" messages on the log. The callback is called with
zero arguments.

### `sbot.indexFeedWriter.stop(query)` (sync)

_Cancels the updating of the index feed for the given query, if it had started_

`query` must be an `ssb-ql-0` query, either as stringified JSON or as an object.

Does not return anything as a response.

## Configuration

Some behaviors of this module can be configured by the user or by application
code through the conventional [ssb-config](https://github.com/ssbc/ssb-config)
object. The possible options are listed below:

```js
{
  indexFeedWriter: {
    /**
     * If `autostart` is defined as an array, it informs ssb-index-feed-writer
     * to automatically call `start()` with each of the array items, as soon as
     * this plugin is initialized.
     *
     * The array items are just ssb-ql-0 objects, except without the `author`
     * property, because that one will always be implicitly equal to `sbot.id`.
     */
    autostart: [
      { type: 'vote', private: false },
      { type: 'post', private: false },
      { type: null, private: true },
    ]
  }
}
```

## License

LGPL-3.0
