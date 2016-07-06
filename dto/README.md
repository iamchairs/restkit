DTO
---

DTO (Data Transfer Object) Validation picks up where [Rules](/rule/README.md) left off. Data
can get complicated with multiple rules for each property. Writing rules for this
is exhaustive, so DTOs and the `Validate` decorator provide an alternative.
Additionally, `ScrubIn` and `ScrubOut` decorators conveniently eliminate properties
coming in and properties going out.

[Property Validation](#validation)

[Data Scrubbing](#scrubbing)

<a name="validation"></a>
## Property Validation

Currently there is no formal definition of a "DTO", but a collection of validation
metadata on class properties. The `Validate` decorator takes a validation object
with rules for validation. Keep in mind that **property validation only applies
to data coming in**.

```typescript
export class User {
  @Validate({
    ... validation rules
  })
  public someProperty: string;
}
```

The following rules can be used in the validation object.

**Options**

| Rule      | Description                                                                                     | Possible Values                       | Example                    |
|-----------|-------------------------------------------------------------------------------------------------|---------------------------------------|----------------------------|
| required  | Set to true if this property is required on this DTO. Fails if the property is undefined.       | true, false                           | true                       |
| type      | Ensures the value received is of this type. Fails the value is not the given type.              | 'string', 'number', 'object', 'array' | 'string'                   |
| minLength | Ensures the string value length is at least this.                                               | 0 to Infinity                         | 10                         |
| maxLength | Ensures the string value length is not greater than this.                                       | 0 to Infinity                         | 20                         |
| min       | Ensures the numerical value is at least this.                                                   | -Infinity to Infinity                 | 10                         |
| max       | Ensures the numerical value is not greater than this.                                           | -Infinity to Infinity                 | 20                         |
| pattern   | Validates if the string value given satisfies this pattern through the RegExp.test(str) method. | RegExp                                | /^[\w\d]+$/ (alphanumeric) |
| values    | Ensures the string value given is one of these values.                                          | CSV                                   | 'red,blue,green'           |


**Error Reporting**

When a validation rule is not met, a default message will be returned with a `400`
error code. If you are going to display this message then you might want to
define your own error messages. For example: Instead of showing the client "username did not 
satisfy the pattern /^[\w\d]+$/", you may instead want the error to read "Please
use alphanumerica characters for your Username". To do that, the value of the
validation rule becomes a tuple, where the first item in the array is the rule,
and the second is the failing message.

```typescript
import {Validate} from 'expresskit';

export class User {
  // With custom message
  @Validate({
    required: [true, 'Please enter a Username'],
    pattern: [/^[\w\d]+$/, 'Please use alphanumerica characters for your Username']
  })
  public username: string;

  // Without custom message
  @Validate({
    required: true,
    pattern: /^[\w\d-_]+@[\w\d-_]+\.[\w\d]+$/
  })
  public email: string;
}
```

**Body**

Validation is applied to the Body of a request. When using the `Body` decorator, pass
the DTO to validate against as the first parameter.

```typescript
export class UserRouter {
  @Route('POST', '/user')
  public static createUser(@Body(User) user: User) {}
}
```

When a DTO is provided to the Body decorator, it is validated against the validation
rules of that DTO. Additionally, when a DTO is used, the __request is required to have
a body__. If a body is not present, a `400` error will be sent.

<a name="scrubbing"></a>
## Data Scrubbing

You may want to scrub data coming in and going out. When data is scrubbed in, use the
`ScrubIn` decorator. When going out, use the `ScrubOut` decorator.

Use `ScrubIn` for **Read-Only** properties. 

```typescript
import {ScrubIn} from 'expresskit';

public class User {
  public id: string;

  public username: string;

  @ScrubIn()
  public created: string;

  @ScrubIn()
  public updated: string;
}
```

Use `ScrubOut` for sensitive data. In addition to using ScrubOut, you will need to
use the `ResponseType` decorator on your route. Without ResponseType, the route can't
determine you are responding with a DTO.

```typescript
import {ScrubOut, ResponseType} from 'expresskit';

public class User {
  public id: string;

  public username: string;

  @ScrubOut()
  public password: string;
}

export class UserRouter {
  @Route('GET', '/user/:id')
  @ResponseType(User)
  public static getUser(@Param('id') userId: string): User {}
}
```

## Keep Reading

[Routing](/route/README.md)

[Response](/response/README.md)

[Middleware](/middleware/README.md)

[Resource Resolutions](/resource/README.md)

[Rules](/rule/README.md)

[DTOs](/dto/README.md)

## More Links

[Restkit Seed Project](https://github.com/iamchairs/restkit-seed)

[Github](https://github.com/iamchairs/restkit)

[Issues](https://github.com/iamchairs/restkit/issues)

[NPM](https://www.npmjs.com/package/restkit)

[Twitter](https://twitter.com/micahwllmsn)