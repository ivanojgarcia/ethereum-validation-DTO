# Custom Header Validation for Ethereum Addresses

This README explains how to set up custom header validation for Ethereum addresses in a NestJS application.

## 1. Custom Header Decorator

Create a file named `header-validation.decorator.ts`:

```typescript
import { createParamDecorator, ExecutionContext, ValidationPipe } from '@nestjs/common';
import { ClassConstructor } from 'class-transformer';

export const HeadersWithValidation = createParamDecorator(
  async (typeResolver: () => ClassConstructor<unknown>, ctx: ExecutionContext) => {
    const headers = ctx.switchToHttp().getRequest().headers;
    const validationPipe = new ValidationPipe({
      expectedType: typeResolver(),
      transform: true,
      whitelist: true,
      validateCustomDecorators: true,
    });
    return await validationPipe.transform(headers, { type: 'custom' });
  },
);
```

## 2. HeaderUserAddressDTO

Create a file named `header-user-address.dto.ts`:

```typescript
import { IsEthereumAddress } from './ethereum-address.validator';

export class HeaderUserAddressDTO {
  @IsEthereumAddress()
  'x-user-address': string;
}
```

## 3. Ethereum Address Validator

Create a file named `ethereum-address.validator.ts`:

```typescript
import { registerDecorator, ValidationOptions, ValidatorConstraint, ValidatorConstraintInterface, ValidationArguments } from 'class-validator';

@ValidatorConstraint({ async: false })
class IsEthereumAddressConstraint implements ValidatorConstraintInterface {
  validate(value: any, args: ValidationArguments) {
    if (typeof value !== 'string') return false;
    return /^0x[a-fA-F0-9]{40}$/i.test(value);
  }

  defaultMessage(args: ValidationArguments) {
    return 'The value must be a valid Ethereum address';
  }
}

export function IsEthereumAddress(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsEthereumAddressConstraint,
    });
  };
}
```

## Usage

In your controller, use the custom decorator and DTO like this:

```typescript
import { Controller, Get } from '@nestjs/common';
import { HeadersWithValidation } from './header-validation.decorator';
import { HeaderUserAddressDTO } from './header-user-address.dto';

@Controller('example')
export class ExampleController {
  @Get()
  exampleEndpoint(
    @HeadersWithValidation(() => HeaderUserAddressDTO) headers: HeaderUserAddressDTO
  ) {
    console.log(headers['x-user-address']);
    // Your logic here
  }
}
```

This setup will validate that the `x-user-address` header is present and is a valid Ethereum address.

## Notes

- Ensure you have `class-validator` and `class-transformer` installed in your project.
- The Ethereum address validation is case-insensitive and checks for the format `0x` followed by 40 hexadecimal characters.
- This validation checks the format only. For more robust validation, consider using libraries like `ethers.js` or `web3.js`.
- Handle Ethereum addresses securely in your application, especially when performing transactions or sensitive operations.
