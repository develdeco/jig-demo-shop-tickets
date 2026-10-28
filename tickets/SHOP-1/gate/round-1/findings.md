# Gate round 1

verdict: fix-slices

Read cart.go and cart_test.go at head c37e7f5 in the gate worktree, against the brief. The fix itself is right: the subtotal is accumulated first and the discount is taken once on it, (subtotal*(100-discountPct)+50)/100 is round-half-up for a non-negative subtotal, the clamp switch is correct in both directions, the empty-items and pct=0 and pct=100 edges hold, and the doc comment matches the behavior. Two findings. One fix: no test pins the nearest-cent rounding - all three cases are rounding-indifferent, and a mutation check in the worktree (reverted) showed that removing the +50, turning rounding into truncation, leaves go test ./cart/... green, so the oracle proves 'once on the order' but not 'to the nearest cent'; Total([]Item{{101,1}},1)==100 and the confirmed half-cent tie-break Total([]Item{{50,1}},3)==49 close it. One note: the +50 misrounds a negative subtotal generally, not only at ties (subtotal -101 at pct 0 returns -100), so the brief's user-confirmed non-negative-line-total premise is now a real precondition of Total; no change asked for under this intent.

## r1-f1 (medium, open): No test pins nearest-cent rounding: dropping the +50 term passes the whole suite

- file: cart/cart_test.go:9
- action: fix
- oracle: test
- workspace: cart
- triage: auto
- risk rationale: Order totals are money and the rounding term is a single unguarded '+ 50'; with no test pinning it, any later rewrite of the expression silently shifts every discounted order by up to a cent, and the oracle stays green. The current head is correct, so the harm is regression exposure on the ticket's own user-confirmed decision rather than a live defect.

The brief's headline decision is that the discount is rounded to the nearest cent, round-half-up on the exact half-cent. Nothing in cart_test.go distinguishes that from plain truncation. All three cases are rounding-indifferent: 999*90=89910 -> 899.1 (truncates to 899 either way), pct clamped to 0 is exact, pct clamped to 100 is 0. Mutation check, run in the round worktree and reverted: replacing cart.go:26 with 'return subtotal * (100 - discountPct) / 100' (the +50 removed, i.e. truncate instead of round) leaves 'go test ./cart/...' green - ok example.com/shop/cart. The surviving mutant means the oracle proves the 'discount once on the order' half of the slice goal but not the 'round to the nearest cent' half. Two cases close it: a plain nearest-cent case where truncation differs - Total([]Item{{PriceCents: 101, Qty: 1}}, 1) is 99.99 -> want 100, truncation gives 99 - and the exact half-cent tie-break the user confirmed, where half-even would differ - Total([]Item{{PriceCents: 50, Qty: 1}}, 3) is 48.50 -> want 49 under round-half-up, 48 under half-even.

## r1-f2 (low, noted): The +50 rounding term misrounds a negative subtotal, breaking the 0%-discount identity

- file: cart/cart.go:26
- action: note
- oracle: test
- workspace: cart
- risk rationale: Unreachable while every line total is non-negative, which the person confirmed holds today, so there is no live defect. If that premise ever lapses the error is a silent one cent per order rather than a crash, and it would land in money.

The brief's tie-break decision rests on 'line totals are always non-negative, so there is no sign ambiguity to resolve'. Nothing enforces that: Item.PriceCents is an exported int with no constructor or validation, so a refund or coupon line would carry it in. Worth knowing is that the formula does not merely pick the other tie-break for negatives, it is off by a cent away from ties too, because Go integer division truncates toward zero and the +50 then pushes the quotient the wrong way. Checked with the same expression on scratch values: subtotal=-101 pct=0 returns -100 where the exact value is -101, so 'a 0% discount returns the subtotal unchanged' no longer holds; subtotal=-1 pct=0 returns 0; subtotal=-333 pct=50 returns -166 where round-half-away-from-zero is -167. No change is needed for the intent as written - the non-negative premise is user-confirmed and clamping the price domain is not in this ticket's scope. Flagging it so the premise is a known precondition of cart.Total rather than an implicit one, should a later ticket admit negative lines.

