# @edgefirst-dev/email

The `Email` class offers methods to validate, parse, and manipulate email addresses. It supports operations like retrieving the username, domain, and alias, and email validation using an [external API](https://verifier.meetchopra.com) via the `EmailVerifier` class.

## Installation

```sh
bun add @edgefirst-dev/email
```

## Usage

Import the `Email` class and create an instance using a valid email string or an `Email` object.

```ts
import { Email } from "@edgefirst-dev/email";

let email = Email.from("user@example.com");
console.log(email.toString()); // "user@example.com"
```

### Email Parsing

You can retrieve and modify the username, domain (hostname), and alias from an email object.

```ts
let email = Email.from("user+alias@example.com");

console.log(email.username); // "user"
console.log(email.hostname); // "example.com"
console.log(email.alias); // "alias"

email.username = "newuser";
email.hostname = "newdomain.com";
email.alias = "newalias";

console.log(email.toString()); // "newuser+newalias@newdomain.com"
```

> [!TIP]
> If you want to disallow users from using aliases in their email addresses, you can set the `alias` property to `undefined`.
>
> ```ts
> let email = Email.from("user+alias@example.com)
> email.alias = undefined;
> console.log(email.toString()); // "user@example.com"
> ```
>
> This can be useful to prevent users from creating multiple accounts using the same email address.

### Email Verification

The `EmailVerifier` class validates an email through an external API.

```ts
import { Email } from "@edgefirst-dev/email";

let email = Email.from("user@example.com");

try {
  await email.verify();
  console.log("Email is valid");
} catch (error) {
  console.error("Invalid email:", error);
}
```

You can override this by extending the `Email` class and implementing your own verification logic.

```ts
class MyEmail extends Email {
  override verifier = {
    async verify(email: Email): Promise<void> {
      // Custom verification logic that throws if it's invalid
    },
  };
}

let myEmail = MyEmail.from("user@example.com");
await myEmail.verify(); // This will call your own verify function
```

### Email Hashing

You can retrieve the SHA-256 hash of an email address.

```ts
console.log(email.hash); // SHA-256 hash of the email
```

This can be used along with Gravatar to get the user's avatar.

```ts
let email = Email.from("user@example.com");
let avatar = `https://www.gravatar.com/avatar/${email.hash}`;
```

### Static Methods

The `Email` class provides static methods for validation and parsing:

```ts
Email.canParse("user@example.com"); // true
Email.canParse("invalid-email"); // false
```

This is similar to `URL.canParse` in the [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL) class.

## Testing

You can test the `EmailVerifier` using [msw](https://mswjs.io/) to mock API responses.

```ts
import { http } from "msw";
import { setupServer } from "msw/node";
import { Email } from "@edgefirst-dev/email";

let server = setupServer(
  http.get(
    "https://verifier.meetchopra.com/verify/user@example.com",
    (req, res, ctx) => {
      return res(ctx.json({ status: true }));
    }
  )
);

server.listen();

let email = Email.from("user@example.com");
await email.verify(); // Mocked valid response
```

## License

See [LICENSE](./LICENSE)

## Author

- [Sergio Xalambrí](https://sergiodxa.com)
