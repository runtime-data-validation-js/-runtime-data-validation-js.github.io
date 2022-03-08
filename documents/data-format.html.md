---
layout: article.html.ejs
title: Data formats for data validation decorators, and storing data
---

As of this writing, the validation decorators and validation functions primarily validate string data.  This is because the package is based on `validator.js` which validates `string` data items.  A few of the functions act on numbers, and in that case can take either `string` or `number` data.  Additionally, the package makes it easy to create custom validation decorators for custom data types.  Obviously there is a very long list of data types we might want to validate, so let's discuss what to do.

A simple example of validating `string` data is:

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

The `IsAlpha` decorator ensures that the string assigned to `#title` only contains alphabetic characters.  This particular example additionally says that the text must be in the US English locale.

This pattern, using a private property (`#title`), ensures the only ability to modify that property is via the `set` accessor.  By attaching validation decorators to this function, we are certain that this property is protected from holding invalid data.

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

The first assignment solely uses alphabetic characters, and works great.  The second has non-alphabetic characters, the spaces, and fails with the error message.  The third has some Romanian text, using characters outside of the US-ASCII character set, which also fails the validation.

In this case the data is assigned to the `set` function as a `string`, it arrives as a `string` to the accessor function, and is stored as a `string` in the `#title` property.

# Validating as a string, then converting to another type

In many cases we can receive data as a `string` but want to store it as its innate data format.  For instance, a URL has a common `string` representation, but it is really a data structure.  In JavaScript that is the URL object.  Let's create a class that holds URL objects, providing a few methods to manipulate the URL.

```ts
import {
    ValidateAccessor, ValidateParams,
    IsURL, IsFQDN
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

The example is to store a URL for an image.  The `set` accessor receives the URL as a `string` which is validated using `@IsURL`.  But, we use `new URL(url)` to parse that to an URL object, and store it as a URL.  We can then implement other `get` accessors to retrieve fields from the URL, and a `toString` method for conversion back to string format.  There is a second `set` accessor which lets us change the `host` field of the URL.  That accessor is protected by `@IsFQDN` which ensures that the string is a fully qualified domain name.

```ts
const ir = new ImgRef();

ir.src = 'https://example.com/logo.jpg';
console.log(ir.toString());
console.log(ir.url());

ir.host = 'foo.bar';
console.log(ir.toString());

// https://xn--diplomatic-qgb.ro/logo.jpg
ir.host = 'diplomatică.ro';
console.log(ir.toString());

// Error: Value 'diplomaticăîntrsurpriză' is not a fully qualified domain name
ir.host = 'diplomaticăîntrsurpriză';
console.log(ir.toString());
```

The first few lines of the test execute as expected.  Since the `url` function returns the URL object, the output in will be the object structure.  Upon assigning a value to `ir.host`, setting the `host` field of the URL, the URL changes from `https://example.com/logo.jpg` to `https://foo.bar/logo.jpg`, which is exactly what's expected.

Using some Romanian text along with the `.ro` top-level-domain (TLD) executes correctly, but the domain name is transliterated.

The last test does not use a domain name, but a simple string.  Therefore, the error shown here is thrown.

A similar example:

```ts
import {
    ValidateAccessor, IsFloatRange,
    conversions
} from 'runtime-data-validation';

const { ToFloat } = conversions;

export class SpeedExample {

    #speed: number;

    @ValidateAccessor<number | string>()
    @IsFloatRange(10, 70)
    set speed(nc: number | string) { 
        this.#speed = ToFloat(nc);
    }
    get speed() { return this.#speed; }

}
```

The `@IsFloatRange` decorator uses the `isFloat` function from `validator.js`, which only validates `string` data.  However, the implementation validates both `string` and `float`, hence the union type used here, `number | string`.  This is because `@IsFloatRange` only uses `isFloat` for string data, and if it is given a `number` it handles the validation itself.

Hence the `set` accessor can be given data in either format.  The `conversions.ToFloat` function takes care of converting the `string` to a `number` if needed.

To test it, try this:

```ts
const sp = new SpeedExample();

sp.speed = 30;
// { speed: 30, type: 'number' }
console.log({ speed: sp.speed, type: typeof sp.speed });

sp.speed = '30';
// { speed: 30, type: 'number' }
console.log({ speed: sp.speed, type: typeof sp.speed });

// Error: Value '300 miles/hr' not a float between 10 and 70
// sp.speed = 300;
// console.log(sp.speed);

// Error: Value '300 miles/hr' not a float between 10 and 70
sp.speed = '300 miles/hr';
console.log(sp.speed);
```

In the first, we assign a `number`, and in the second we assign a `string` that is numerical.  In both cases the stored value is a `number` with the correct value.

In the second pair we assign invalid data.  That causes an appropriate error to be thrown.

# Validation of non-string data

The last example showed that some of the decorators handle validation of both `number` and `string` values, while most of them solely validate `string` values.

Generalized, the pattern is to

* Receive and validate data as a `string`, or in some cases as a `number`
* If appropriate, convert to another object type by parsing the string

But, obviously, there are a zillion and one different data types to validate.  Sometimes we want to validate data in its native data type, rather than validating the string representation.

This project is welcome to receive new validator functions.  

Additionally, it provides a function making it easy to create new validation decorators.  See: [](custom-validator.html)
