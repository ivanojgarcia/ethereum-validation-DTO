# Custom Header Validation for Ethereum Addresses

This README explains how to set up custom header validation for Ethereum addresses in a NestJS application.

## Description

This package provides a custom validation solution for NestJS applications that need to verify Ethereum addresses in HTTP headers. It offers a streamlined way to ensure that incoming requests contain valid Ethereum addresses in specified headers, enhancing security and data integrity in blockchain-related applications.

## Problem Solved

In blockchain applications, particularly those interacting with Ethereum, it's crucial to validate user addresses before processing requests. Incorrect or malformed addresses can lead to transaction errors, security vulnerabilities, or data inconsistencies. This package solves several key issues:

1. **Input Validation**: It ensures that Ethereum addresses in headers conform to the correct format (0x followed by 40 hexadecimal characters), preventing processing of invalid data.

2. **Type Safety**: By integrating with NestJS's dependency injection system and TypeScript, it provides type-safe access to validated header values.

3. **Reusability**: The custom decorator and validators can be easily reused across different controllers and routes, promoting code consistency and reducing duplication.

4. **Separation of Concerns**: It separates the validation logic from the business logic in controllers, leading to cleaner, more maintainable code.

5. **Early Error Detection**: Invalid addresses are caught early in the request lifecycle, allowing for quick rejection of improper requests before they reach application logic.

This solution streamlines the development process for NestJS applications dealing with Ethereum addresses, enhancing reliability and reducing the potential for errors in blockchain interactions.

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
