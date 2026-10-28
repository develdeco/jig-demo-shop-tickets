# Review Notes

## Round 2

r1-f1 is fixed: the two new cases in cart_test.go pin nearest-cent rounding. Mutation check in the round worktree, reverted and re-verified green: replacing cart.go:26 with 'subtotal * (100 - discountPct) / 100' (truncation) now fails TestTotalRoundsUpToTheNearestCent (99 want 100) and TestTotalBreaksExactHalfCentTiesUp (48 want 49); the opposite mutant '+ 99' (ceiling) fails TestTotalRoundsOnceOnTheOrder (900 want 899), so the oracle pins rounding in both directions and pins the user-confirmed round-half-up tie-break against half-even. Both new cases match the brief's arithmetic (101 at 1% = 0.9999 -> 100; 50 at 3% = 0.485 -> 49) and the clamp seams at both bounds stay covered. No new problems in the delta, which touches cart_test.go only. r1-f2 confirmed still present: cart.go:26 is byte-identical in this delta, so the negative-subtotal misrounding stays a note and a known precondition of cart.Total, not something this round changed.

## Findings

- **No test pins nearest-cent rounding: dropping the +50 term passes the whole suite** (medium): Order totals are money and the rounding term is a single unguarded '+ 50'; with no test pinning it, any later rewrite of the expression silently shifts every discounted order by up to a cent, and the oracle stays green. The current head is correct, so the harm is regression exposure on the ticket's own user-confirmed decision rather than a live defect.
  - Status: fixed by slice fix-1-cart-test and cleared at round 2
  - Oracle: test

- **The +50 rounding term misrounds a negative subtotal, breaking the 0%-discount identity** (low): Unreachable while every line total is non-negative, which the person confirmed holds today, so there is no live defect. If that premise ever lapses the error is a silent one cent per order rather than a crash, and it would land in money.
  - Status: noted
  - Oracle: test

## Coverage

Touched files:

- cart/cart.go
- cart/cart_test.go

Reviewed paths:

- cart/cart.go
- cart/cart_test.go

