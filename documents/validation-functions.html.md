---
layout: article.html.ejs
title: Using and defining data validation functions
---

The `runtime-data-validation` package revolves around data validation functions.  There is a long list of validation decorators available, but each of those decorators in turn relies on validation functions.

The TypeScript documentation talks a lot about _type guard_ functions.  These functions inspect an object, and provide a `boolean` determination whether it is of a given _Type_.  In discussing [how to develop custom validation decorators](custom-validator.html) we showed a type, _CustomType_, and a function, _isCustomType_, which determines whether an object has the CustomType shape.

Type guard functions like that can be useful in data validation.  But, there is a difference between determining which _type_ an object is, versus whether the object contains valid data.  The object could be of the correct type, but have insane data values which will cause the application to crash.

To take an example, lets look at one of the decorator functions:

```ts
export function IsIntRange(min: number, max: number) {
    // console.log(`params.IsIntRange ${min} ${max}`);
    return generateValidationDecorator(
        (value) => validators.isIntRange(value, min, max),
        `Value :value: not an integer between ${min} and ${max}`);
}
```

This implements the `@IsIntRange` decorator.  This takes two parameters, `min` and `max`, and determines whether the number is within that range.  The `generateValidationDecorator` function is discussed in [elsewhere](custom-validator.html), and its function is to as its name implies, which is to create the actual decorator function.  It does this by using the validation function which is passed as the first parameter.

```ts
export const isIntRange = (value: string | number, min: number, max: number) => {
    if (typeof value === 'number') {
        if (Number.isInteger(value)
         && value >= min && value <= max) {
            return true;
        } else return false;
    }
    else if (validator.isInt(value, { min: min, max: max })) {
        return true;
    } else return false;
};
```

This is the validation function for `@IsIntRange`.  This particular validation function has to accommodate whether its value is a `number` or a `string`.  If it is a `number`, the validation is directly handled in this function, otherwise it is handled in a function provided by the `validator.js` library.

# Defining validation functions

The point of bringing this up is to demonstrate that data validation is a step beyond type validation.  We can easily determine between `number` and `string`.  But, the question of whether a `string` is a numerical string is a deeper question having to do with the contents of the string.  Likewise, if you want to limit the number to a given range, that's an even deeper question.

In other words, _data validation_ is _type validation_ plus determining whether the values are legitimate.

A variable that is supposed to represent _speed_ is going to be valid for a given range of numerical values.  It's not enough to determine that the variable is a `number` or a `string` which is numerical.  One must also determine whether the value is within the acceptable range.

Therefore the validation functions we pass to `generateValidationDecorator` should not stop at a determination of the data type.  They need to determine validity as well.  The `@IsURL` decorator determines whether a `string` is a valid URL, for example.

# Using validation functions

Inside the `runtime-data-validation` package, the validation functions are accessed this way:

```ts
import {
    ...
    ValidateParams, ValidateAccessor, generateValidationDecorator,
    validators
} from 'runtime-data-validation';
```

The `validators` object contains the validation functions provided by this package.

To use one is simple.  For example, the validation function for `@IsAscii` is called like so: `validators.isAscii(value.title)`

To be used in a validation decorator, the first argument of the validation function must be the value to be validated.  If the validation function takes any options, those will be passed in subsequent arguments.

Validation functions must return a `boolean` value where `true` says that `value` is valid, and `false` says it is not.
