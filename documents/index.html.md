---
layout: article.html.ejs
title: Automatic runtime data validaton decorators for TypeScript
hidebreadcrumb: true
hidepubldate: true
---



<p style="font-size: 200%;" align="center"><em>Automatically validate data in fields or methods in JavaScript/TypeScript objects, at runtime.</em></p>

<p style="font-size: 200%;" align="center"><em>Succinctly describe data validation requirements using TypeScript decorators.</em></p>

<p style="font-size: 200%;" align="center"><em>Have certainty that protected properties and methods always have correctly validated values.</em></p>

This project - [`runtime-data-validation`](https://www.npmjs.com/package/runtime-data-validation) - is easily installed in a TypeScript/Node.js project, and provides seamless automatic data validation for the following:

* Properties handled by accessor functions in instances of TypeScript classes
* Parameters to member functions in instances of TypeScript classes

Values assigned through an accessor, or passed as a method parameter, are checked against validation requirements, and an exception is thrown if the data does not validate.  Throwing the exception prevents the assignment from being made, or prevents the method from being invoked.

For an example of what this means:

```js
import {
    ValidateAccessor, ValidateParams,
    IsIntRange, IsInt, IsFloatRange,
    conversions
} from 'runtime-data-validation';

const { ToFloat, ToInt } = conversions;

class ValidateExample {

    #year: number;

    @ValidateAccessor<number>()
    @IsIntRange(1990, 2050)
    @IsInt()
    set year(ny: number | string) { this.#year = ToInt(ny); }
    get year() { return this.#year; }

    @ValidateParams
    area(
        @IsFloatRange(0, 1000) width: number | string,
        @IsFloatRange(0, 1000) height: number | string
    ) {
        return ToFloat(width) * ToFloat(height);
    }

}
```

We have here:

* _Validation decorators_ attached to accessor `set` methods
* _Validation decorators_ attached to method parameters
* _Decorators_ attached to accessor `set` methods and regular methods
* _Conversion functions_ to aid with converting possibly `string` values to either integer or floating `number` values

If we attach to the script something like this:

```ts
const ve = new ValidateExample();

ve.year = 1999;
console.log({
    year: ve.year,
    area: ve.area(10, 200)
})

// Error: Value 2150 not an integer between 1990 and 2050
ve.year = 2150;

console.log(ve.area('10', '200'));

// Error: Value 'two hundred' not a float between 0 and 1000
console.log(ve.area('10', 'two hundred'));
```

Two of those statements throw the errors shown here.  The first assigns a value to the `year` property that's outside the declared range.  The next statement invokes the `area` method, with numerical values represented as strings.  The `IsFloatRange` decorator validates the strings correctly, and the `ToFloat` function converts that to a `number` so that the result of the multiplication is a number.  However, the invocation after that has a string that is decidedly not numerical.

In short, assignments to object properties, or method calls, can be automatically validated and protected from incorrect data.


The validation decorators describe specific validation algorithms to execute. The implementation for most uses [validator.js](https://www.npmjs.com/package/validator), in the background.  As a result, the current validation decorators focus on validating strings.

By themselves the validation decorators do not perform validation.  The rationale has to do with how decorator functions operate (to learn why, read [about using and developing TypeScript decorators](https://javascript.plainenglish.io/deep-introduction-to-using-and-implementing-typescript-decorators-a9e876ad0d43)).  To cause the validation decorators to execute, use `@ValidateAccessor` and `@ValidateParams` as shown here.  These decorators instrument function execution for both accessors and methods.  This causes any validation decorators to execute.  Those decorator functions indicate invalid data by throwing an exception, preventing function execution.

Put another way, the validation functions guarantee that when protected functions execute, their parameters are validated to be correct.

This mechanism protects two kinds of things:

* Properties in TypeScript object instances - via protecting accessor functions
  * It is recommended to use the new private field syntax, as shown in the example.  This will ensure that access to property data storage is channeled through the accessor functions, which are protected by validation decorators.
* Methods in TypeScript object instances - via protecting method parameters

The developer is not required to explicitly call validation functions.  Instead, validation is performed on every access to the protected property or methods this pattern of decorators ensures that validation is performed on every assignment to the accessor, or to every method call.

This is different from the data validation packages which require validation to be explicitly invoked.  Doesn't this raise the risk that the developer will forget to do validation on a critical code path?

To install this in your package, see: [](getting-started.html)



The package includes a very long list of validation decorators.  The implementation uses [validator.js](https://www.npmjs.com/package/validator) in the background.  Development was inspired by [class-validator](https://www.npmjs.com/package/class-validator).
