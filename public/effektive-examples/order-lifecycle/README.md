# Order lifecycle application

This application is a witness for type identity. It is not one of the three
gate applications that the accepted stopping rule names. It models an order and
its pricing with type declarations that have identity by name.

Files in this directory:

- [contracts.eff](contracts.eff): four order states, the InvalidTransition error, the pricing rule tree and the input record.
- [orders.eff](orders.eff): transitions, match by name, pricing and JSON decoding.
- [main.ts](main.ts): runnable entry point that prints a JSON report.
- [checks.ts](checks.ts): eleven compiled application checks.
- [effektive.json](effektive.json): compiler project configuration.
- [deno.json](deno.json): pinned Effect imports and strict checking options.

Download these files together. Use the effektive-lang CLI to build this
directory with --validate --tsc. Build writes contracts.ts and orders.ts
beside the source files. From the downloaded directory, run:

```sh
deno run --config deno.json main.ts
deno test --config deno.json checks.ts
```

The four states Pending, Paid, Shipped and Cancelled are record types with
identity. The union Order in the same module lists them, so each one is a tagged
record. Each has a runtime tag that includes the module name, for example
"contracts#Pending". A
function that accepts one state, such as pay(order: Pending, ...), rejects the
other states at compile time. A match over Order has one arm for each state and
no wildcard arm. A missing state is a compile error.

The pricing rules form a recursive tree of Fixed, Times, Percent, Capped and Sum.
The price of Times is the base price times the factor. The constructor Cents(t.factor)
turns the Count factor into Cents, because two different types with identity
never mix in an operator. The price of Percent is
round(base * Cents(p.percent) / 100). The division operator gives number, and the
builtin round gives an int, with a half rounded away from zero.

The function fromInput decodes the record OrderInput from unknown JSON. No union
lists OrderInput, so it has no tag. The function then builds a Pending value with
the constructor. JSON with a wrong or missing tag fails decodeOrder.

The eleven checks cover each transition, each stateOf and describe result, isOpen
and the InvalidTransition failure. They price the sample rule from source and
one rule that TypeScript builds with make. Two checks price a percent rule,
including the rounding of 1.5 to 2 and of 4.5 to 5. They decode valid and invalid input,
check that the encoded form keeps the tag, and check the rejected decode of a
wrong tag.
