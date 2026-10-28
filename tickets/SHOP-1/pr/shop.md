## Intent

`cart.Total` applies the discount to each line item and truncates each line
to whole cents before summing. `cart/cart_test.go:TestTotalRoundsOnceOnTheOrder`
already encodes the repro: three items at 333 cents each, 10% off.

```
go test ./cart/...
--- FAIL: TestTotalRoundsOnceOnTheOrder (0.00s)
    cart_test.go:10: Total = 897 cents, want 899
```

## What changed

- Take the discount once on the order total, round to the nearest cent, and clamp discountPct to [0, 100]. (c37e7f5)

Fixes from review:

- Fix these gate findings (fbd8f46)

## Verification

Oracles:

- test

Reviewed head:

- shop: fbd8f46

Revalidation tier: none

Findings: 1 fixed, 0 dismissed, 1 noted, 0 asked. See the pull request's first comment for detail.
