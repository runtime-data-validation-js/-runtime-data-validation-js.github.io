---
layout: article.html.ejs
title: Getting started with the runtime-data-validation package
---

The `runtime-data-validation` package is available for TypeScript projects running on Node.js.  It requires the decorator support provided by TypeScript.  In theory it can run in the browser, but that hasn't been implemented.

Installation:

```
$ npm install runtime-data-validation --save
```

To turn on decorator support, in `tsconfig.json` add this:

```json
{
    "compilerOptions": {
        ...
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true,
        ...
    }
}
```

For use on Node.js, it's also useful to use `es2021` and `esnext` in the `lib` setting, `es2021` as the `target`.  These two settings here turn on experimental decorator support, and enable necessary metadata support.

To use the decorators, add this to your code:

```js
import {
    IsIntRange, IsInt, IsFloatRange, IsFloat,
    ...
    ValidateParams, ValidateAccessor,
} from 'runtime-data-validation';
```

That is, `import` any needed decorator function.
