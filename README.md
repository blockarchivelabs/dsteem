# [dsteem](https://github.com)

Robust steem client library that runs in both node.js and the browser.

Needs test net urls, chain id

---

**note** As of version 0.7.0 WebSocket support has been removed. The only transport provided now is HTTP(2). For most users the only change required is to swap `wss://` to `https://` in the address. If you run your own full node make sure to set the proper [CORS headers](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) if you plan to access it from a browser.

---

## Browser compatibility

[![Build Status](https://saucelabs.com/browser-matrix/feruzm-dsteem.svg)](https://saucelabs.com/open_sauce/user/feruzm-dsteem)

## Installation

### Via npm

For node.js or the browser with [browserify](https://github.com/substack/node-browserify) or [webpack](https://github.com/webpack/webpack).

```
npm install @upvu/dsteem
```

### From cdn or self-hosted script

Grab `dist/dsteem.js` from git and include in your html:

```html
<script src="@upvu/dsteem.js"></script>
```

Or from the [unpkg](https://unpkg.com) cdn:

```html
<script src="https://unpkg.com/@upvu/dsteem@latest/dist/dsteem.js"></script>
```

Make sure to set the version you want when including from the cdn, you can also use `dsteem@latest` but that is not always desirable. See [unpkg.com](https://unpkg.com) for more information.

## Usage

### In the browser

```html
<script src="https://unpkg.com/@upvu/dsteem@latest/dist/dsteem.js"></script>
<script>
  var client = new dsteem.Client([
    "https://api.upvu.org",
    "https://api.steemit.com",
  ]);
  client.database
    .getDiscussions("trending", { tag: "writing", limit: 1 })
    .then(function (discussions) {
      document.body.innerHTML += "<h1>" + discussions[0].title + "</h1>";
      document.body.innerHTML += "<h2>by " + discussions[0].author + "</h2>";
      document.body.innerHTML +=
        '<pre style="white-space: pre-wrap">' + discussions[0].body + "</pre>";
    });
</script>
```

### In node.js

With TypeScript:

```typescript
import { Client } from "@upvu/dsteem";

const client = new Client(["https://api.upvu.org", "https://api.steemit.com"]);

for await (const block of client.blockchain.getBlocks()) {
  console.log(`New block, id: ${block.block_id}`);
}
```

With JavaScript:

```javascript
var dsteem = require("@upvu/dsteem");

var client = new dsteem.Client([
  "https://api.upvu.org",
  "https://api.steemit.com",
]);
var key = dsteem.PrivateKey.fromLogin("username", "password", "posting");

client.broadcast
  .vote(
    {
      voter: "username",
      author: "almost-digital",
      permlink: "dsteem-is-the-best",
      weight: 10000,
    },
    key
  )
  .then(
    function (result) {
      console.log("Included in block: " + result.block_num);
    },
    function (error) {
      console.error(error);
    }
  );
```

With ES2016 (node.js 7+):

```javascript
const { Client } = require("@upvu/dsteem");

const client = new Client(["https://api.upvu.org", "https://api.steemit.com"]);

async function main() {
  const props = await client.database.getChainProperties();
  console.log(`Maximum blocksize consensus: ${props.maximum_block_size} bytes`);
  client.disconnect();
}

main().catch(console.error);
```

With node.js streams:

```javascript
var dsteem = require("@upvu/dsteem");
var es = require("event-stream"); // npm install event-stream
var util = require("util");

var client = new dsteem.Client([
  "https://api.upvu.org",
  "https://api.steemit.com",
]);

var stream = client.blockchain.getBlockStream();

stream
  .pipe(
    es.map(function (block, callback) {
      callback(null, util.inspect(block, { colors: true, depth: null }) + "\n");
    })
  )
  .pipe(process.stdout);
```

## Bundling

The easiest way to bundle dsteem (with browserify, webpack etc.) is to just `npm install @upvu/dsteem` and `require('@upvu/dsteem')` which will give you well-tested (see browser compatibility matrix above) pre-bundled code guaranteed to JustWork™. However, that is not always desirable since it will not allow your bundler to de-duplicate any shared dependencies dsteem and your app might have.

To allow for deduplication you can `require('@upvu/dsteem/lib/index-browser')`, or if you plan to provide your own polyfills: `require('@upvu/dsteem/lib/index')`. See `src/index-browser.ts` for a list of polyfills expected.

---

_Share and Enjoy!_
