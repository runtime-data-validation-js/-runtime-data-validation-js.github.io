---
layout: article.html.ejs
title: Data formats for validation, and storing data
---

As of this writing, the validation decorators and validation functions primarily validate string data.  A few of the functions act on numbers, and in that case can take either `string` or `number` data.  Obviously there are many other data types we might want to validate, so let's discuss what to do.

This package is based on `validator.js` which validates `string` data items.  It has a long list of validation functions, all of which validate strings.

A simple example is:

```ts
import {
    ValidateAccessor, ValidateParams,
    IsAlpha
} from 'runtime-data-validation';

class Example {
    #title: string;

    @ValidateAccessor<string>()
    @IsAlpha('en-US')
    set title(ny: string) { this.#title = ny; }
    get title() { return this.#title; }
}
```

The `IsAlpha` decorator ensures that the string assigned to `#title` only contains alphabetic characters.  This particular example says that not only can the text only contain alphabetic characters, but from the US English locale.

To see that in action:

```ts
const ee = new Example();

ee.title = 'AManForAllSeasons';
console.log(ee.title);

// Error: Value 'A Man For All Seasons' is not alpha en-US
ee.title = 'A Man For All Seasons';
console.log(ee.title);

// Error: Value 'diplomaticăîntrsurpriză' is not alpha en-US
ee.title = 'diplomaticăîntrsurpriză';
console.log(ee.title);
```

The first assignment solely uses alphabetic characters, and works great.  The second has non-alphabetic characters, the spaces, and fails with the error message.  The third has some Romanian text, and also fails the validation.

In this case the data is assigned to the `set` function as a `string`, it arrives as a `string` to the accessor function, and is stored as a `string` in the `#title` property.

# Validating as a string, then converting to another type

In many cases we can receive data as a `string` but want to store it as its innate data format.

```ts
import {
    ValidateAccessor, ValidateParams,
    IsURL,
    IsFQDN
} from 'runtime-data-validation';

class ImgRef {
    #src: URL;

    @ValidateAccessor<string>()
    @IsURL()
    set src(nurl: string) { this.#src = new URL(nurl); }
    get src() { return this.#src?.toString(); }

    url()  { return this.#src; }
    toString() { return this.#src?.toString(); }
    get protocol() { return this.#src?.protocol; }
    get pathname() { return this.#src?.pathname; }

    @ValidateAccessor<string>()
    @IsFQDN()
    set host(nh: string) { this.#src.host = nh; }
    get host() { return this.#src?.host; }
}
```

The example is to store a URL for an image.  The `set` accessor receives the URL as a string, but we can use `new URL(url)` to parse that to an URL object, and store it as a URL.  We can then implement other `get` accessors to retrieve fields from the URL, and a `toString` method for conversion back to string format.  There is a second `set` accessor which lets us change the `host` field of the URL.  That accessor is protected by `@IsFQDN` which ensures that the string is a fully qualified domain name.

```ts
const ir = new ImgRef();

ir.src = 'https://example.com/logo.jpg';
console.log(ir.toString());
console.log(ir.url());

ir.host = 'foo.bar';
console.log(ir.toString());

// Error: Value 'diplomaticăîntrsurpriză' is not a fully qualified domain name
ir.host = 'diplomaticăîntrsurpriză';
console.log(ir.toString());
```

The first few lines of the test execute as expected.  Since the `url` function returns the URL in its native format, the output in that case is the object structure.  The host portion of the URL changes from `https://example.com/logo.jpg` to `https://foo.bar/logo.jpg`, which is just what we'd expect.

Since the last portion does not use a domain name, the error shown here is thrown.

A similar example:

```ts
class Speed {
    #viteza: number;

    @ValidateAccessor<string | number>()
    @IsFloatRange(0, 99999)
    set speed(ns: string | number) { this.#viteza = ToFloat(ns); }
    get speed() { return this.#viteza; }

    toString() { return this.#viteza?.toString(); }
}
```

This is the same strategy of receiving data as a `string`, and converting it to the native data format.  Notice that the type supplied to the `set` accessor and to `@ValidateAccessor` is `string | number`, so that TypeScript knows to accept either data format.

The `@IsFloatRange` uses the `isFloat` function from `validator.js`, which only validates `string` data.  This means `@IsFloatRange` tests its input, and if it is a `number` it handles the validation itself rather than using `validator.js`.
