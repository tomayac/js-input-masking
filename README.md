# Input Masking

**Author:** Thomas Steiner ([tomac@google.com](mailto:tomac@google.com),
[@tomayac](https://twitter.com/tomayac))

**Last updated:** 2021-06-29

## The problem

By _"restricting input using preformatted input masks (telephone number,
birthday, Social Security number), or even auto-correcting input by appending or
removing unnecessary characters"_ [_cf._
[Klimczak](https://books.google.es/books?id=xMJOm4ppCkYC&printsec=frontcover&dq=editions:OTpjPvHq_7MC&hl=en&sa=X&redir_esc=y#v=onepage&q=input%20masks&f=false)],
**input masking** helps users enter information in forms more correctly, and
existing data can be formatted adequately. Many implementations of input masking
[exist in user land](https://bashooka.com/coding/javascript-input-mask-libraries/),
proving that there is a true need for this feature.

## A beginning of a proposal

This proposal is to gauge interest in making input masking part of the language,
maybe as a new interface of `Intl`. Here're some code snippets that show what
this could look like in practice:

- **Globally agreed-on** input mask:

  ```js
  new Intl.InputMask("credit-card-number").format("4012888888881881");
  // "4012 8888 8888 1881"
  ```

- **Locale-aware input mask** with customization options:
  ```js
  new Intl.InputMask("phone-number", {
    locale: "de-DE",
    countryCode: "leadingPlus",
    areaCode: "leadingZero",
    groupSize: 2,
  }).format("00494012345678");
  // "+49 (0)40 12 34 56 78"
  ```
- Fully **custom input mask** with a mask function:

  ```js
  new Intl.InputMask("custom", {
    maskFunction: (input) => input.toLowerCase().replaceAll(" ", ""),
  }).format("No Spaces No Uppercase");
  // "nospacesnouppercase"
  ```

## Isomporhic JS™

Making this part of the language would allow people to use this on
the **client** and the **server**.

### On the client

For example, a **client-side implementation** could use this as follows (note
that the `type="tel"` of the `<input>` does not mean the input value is
_"automatically validated to a particular format before the
form can be submitted, because formats for telephone numbers vary so much around
the world"_ [_cf._
[MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/tel#:~:text=the%20input%20value%20is%20not%20automatically%20validated%20to%20a%20particular%20format%20before%20the%20form%20can%20be%20submitted%2C%20because%20formats%20for%20telephone%20numbers%20vary%20so%20much%20around%20the%20world.)]):

```html
<label for="phone">Enter your phone number:</label>
<input type="tel" id="phone" name="phone" />
```

```js
const formatPhoneNumber = (value) => {
  return new Intl.InputMask("phone-number", {
    locale: "de-DE",
    countryCode: "leadingPlus",
    areaCode: "leadingZero",
    groupSize: 2,
  }).format(value);
};

const input = document
  .querySelector("#phone")
  .addEventListener("input", (e) => {
    e.target.value = formatPhoneNumber(e.target.value);
  });
```

### On the server

For a **server-side implementation**, this could look as follows:

```bash
SELECT phone FROM users_legacy;
# Returns a mix of formats from a legacy dataset:
# 00494012345678\n+494012345678\n04012345678
```

```js
const formatPhoneNumber = (value) => {
  return new Intl.InputMask("phone-number", {
    locale: "de-DE",
    countryCode: "leadingPlus",
    areaCode: "leadingZero",
    groupSize: 2,
  }).format(value);
};

// Express.js YOLO example.
app.get("/phones", (req, res) => {
  const rawPhones = getPhoneNumbersFromLegacyDB();
  const formattedPhonesHTML = rawPhones
    .map((rawPhone) => {
      return formatPhoneNumber(rawPhone);
    })
    .join("<br>");
  res.send(formattedPhonesHTML);
});
```

## Alternatives

Just leaving this to the user land is an obvious alternative. The ecosystem of
input masking libraries is alive, and great implementations exist. Pulling any
of those in comes at a cost for each locale that's needed, and the weight of the
library itself.

Another alternative would be to add new (and smarter) input types. A recent
example is the
[`input[type=currency]`](https://discourse.wicg.io/t/proposal-input-type-currency/5398)
proposal. The declarative burden for advanced formatting needs is not to be
underestimated, though (see the above `countryCode` and `areaCode` examples).
This also does not solve the issue on the server side.

## Feedback

Feedback on this early-stage idea is welcome. Please
[open a new Issue](https://github.com/tomayac/js-input-masking/issues).
