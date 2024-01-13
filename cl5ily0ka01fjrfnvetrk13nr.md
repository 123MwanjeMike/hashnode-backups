---
title: "How different is CommonJs require from ES6 import?"
seoTitle: "How different is CommonJs require from ES6 import?"
seoDescription: "CommonJs is the original and default module system of Node.js which uses require and module.exports."
datePublished: Sat Aug 28 2021 09:28:36 GMT+0000 (Coordinated Universal Time)
cuid: cl5ily0ka01fjrfnvetrk13nr
slug: how-different-is-commonjs-require-from-es6-import
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1657655579463/tJ3qiRcPt.jpeg
tags: javascript, es6, require, commonjs, import

---

In JavaScript, you can use either ECMAScript 6(ES6) modules or CommonJs modules in your project and there are a few differences between these that do affect how your program modules are loaded. In this article, I explore how each works and how it may affect your program execution.

### CommonJs modules.
CommonJs is the original and default module system of Node.js which uses require and module.exports. Below is an example.

```javascript
// Importing modules
const fs = require('fs');
const fileDelete = require('./fileDeleter');
const fileName = require('./fileNamer');

const writeFile = (data) => {
  return fs.writeFileSync(fileName, data);
}

// Exporting writeFile module
modules.exports = writeFile;
```

With require, you can’t selectively load only the modules you need. This means even the fileDelete module from the example above will be imported even if it is not needed or used anywhere. Additionally, importing of the modules is synchronous which means that fileName module can’t be imported before fs and fileDelete modules are imported, and a failure to import fileDelete will cause run-time errors even if it is not used anywhere in our program. CommonJS modules are the choice for the node.js server.

### ECMAScript modules
ECMAScript modules are relatively newer and use import and export. Below is the transformation of our CommonJs example from above to ESM.

```javascript
// Importing modules
import fs from 'fs';
import fileDelete from './fileDeleter';
import fileName from './fileNamer';

const writeFile = (data) => {
  return fs.writeFileSync(fileName, data);
}

// Exporting writeFile module
export default function writeFile;
```

With import, you load only the modules you need. For example, the fileDelete module from the above will not be imported since it is not used anywhere. Additionally, the importing of the modules is asynchronous which means that both fs and fileName are imported at the same time. You generally want to use ESM for your new projects.

**…how about .cjs and .mjs?**
.cjs is a file extension for CommonJS modules while .mjs is a file extension for ECMAScript module. Node.js by default treats .js files as CommonJS modules. You can change this by adding "type": "module"to your package.json file so you can use ECMAScript modules (in your .mjs files) within a Node.js environment. This is what Google Chrome [V8](https://v8.dev/) recommends.

I hope this was helpful to you and for further reading, do checkout [JavaScript modules](https://v8.dev/features/modules).

*Happy coding!*