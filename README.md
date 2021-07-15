👍 Sum Types
===

### Installation

Imagine working with a client, and you need to take online payment for a service. There are two options to pay - either in part or in full. When paying in full, the payment should include a total amount and a refundable amount. When paying in partial, there should be a minimum amount required to be paid at the time of purchase, the remaining amount with a surcharge (optional based on the card used for payment) amount, and a refundable amount.

```typescript
export interface PaymentOption {
  type: "partial" | "full";
  totalRental: number;
  payNow: number;
  refundableBond: number;
  balance?: number;
  balanceSurcharge?: number;
  payLater?: number;
}
```

However; that is not expressive enough!

Instead, let's split the payment options into two different definitions. Sum Types (or Discriminated Union or Algebraic Data Types) are a great way to represent data when they can take multiple options. We can have a `PaymentOption` type which can either be a `FullPaymentOption` or a `PartPaymentOption`. We can now have the properties that apply to each scenario together.

We can combine singleton types, union types, type guards, and type aliases to build an advanced pattern called discriminated unions, also known as tagged unions or algebraic data types Or Sum Types.

```typescript
export type PaymentOption = FullPaymentOption | PartPaymentOption;

export interface FullPaymentOption {
  type: "full";
  totalRental: number;
  payNow: number;
  refundableBond: number;
}

export interface PartPaymentOption {
  type: "partial";
  totalRental: number;
  payNow: number;
  refundableBond: number;
  balance: number;
  balanceSurcharge?: number;
  payLater: number;
}
```

The data is now expressive and indicates what fields apply to the relevant payment option. Since `balanceSurcharge` is optional based on the card type used for payment, I have it as optional on `PartPaymentOption` type.

No longer do we need to keep track of when data will and will not be populated. Having conditional properties on an interface or a class creates confusion. It makes it harder to deal with the data and the various combinations it can take. Tend to avoid this as much as possible. I hope this gives you an idea to take away and implement for your problem.

```
npm install sum-types
```

### Example

In TypeScript:

```typescript
import SumType from 'sum-types';

class Maybe<T> extends SumType<{ Just: [T]; Nothing: [] }> {}

function Just<T>(value: T): Maybe<T> {
  return new Maybe("Just", value);
}

function Nothing<T>(): Maybe<T> {
  return new Maybe("Nothing");
}

const x = Just("foo");

const result = x.caseOf({
  Nothing: () => "nope",
  Just: (a) => a + "bar",
});
```

Or in JavaScript

```javascript
import SumType from 'sum-types';

class Maybe extends SumType {}

function Just(value) {
  return new Maybe("Just", value);
}

function Nothing() {
  return new Maybe("Nothing");
}

const x = Just("foo");

const result = x.caseOf({
  Nothing: () => "nope",
  Just: (a) => a + "bar",
});
```

### Wildcard Matching

If the kind of a sum type instance isn't present in the pattern given to `caseOf`, a default key called `_` will be used instead.

```ts
import SumType from 'sum-types';

class RequestState<T = never> extends SumType<{
  NotStarted: [];
  Connecting: [];
  Downloading: [number];
  Completed: [T];
  Failed: [string];
}> {}

const state = new RequestState('Failed', 'Connection reset.');
const progressPercentage = state.caseOf({
  Downloading: pct => pct,
  Completed: () => 100,
  _: () => 0
});
```