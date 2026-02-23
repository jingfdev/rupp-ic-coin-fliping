# Coin Flip Cryptography

## Fair Remote Coin-Tossing Using Quadratic Residues & the Chinese Remainder Theorem

**Course:** Information & Coding Theory — Year 4, Semester 2  
**Demo Parameters:** `p = 31`, `q = 43`, `n = 1333`

---

## Slide 1 — The Problem

### Why can't we just flip a coin over the internet?

When Alice and Bob are not in the same room, a naive "I call heads!" protocol breaks immediately:

| Threat | Description |
|--------|-------------|
| **Alice cheats** | She waits to see Bob's call, then claims "heads" or "tails" to win. |
| **Bob cheats** | He changes his call after seeing Alice's message. |
| **Network tampering** | Messages can be edited, delayed, or replayed by a third party. |

> **Goal:** Design a protocol where **neither party can force the outcome** — each has an honest 50/50 chance of winning, enforced by mathematics alone.

---

## Slide 2 — Mathematical Building Blocks

Four concepts power the entire protocol:

1. **Prime Numbers**  
   Alice selects two secret primes `p` and `q`. Their product `n = p * q` is public, but factoring `n` back into `p` and `q` is computationally hard.

2. **Modular Arithmetic**  
   All operations are performed "mod n" — values wrap around like hours on a clock.

3. **Quadratic Residues (Squaring mod n)**  
   Computing `x^2 mod n` is easy. Reversing it (finding `x` from `x^2 mod n`) is hard **unless** you know `p` and `q`.

4. **Chinese Remainder Theorem (CRT)**  
   Lets Alice combine partial square roots mod `p` and mod `q` into full square roots mod `n`. This gives her exactly **4** roots — her secret advantage.

---

## Slide 3 — Protocol Overview

```
  Alice (knows p, q)                         Bob
  ──────────────────                         ───
  1. Picks primes p, q
     Computes n = p * q
                            ── n ──>
                                             2. Picks random alpha
                                                Computes A = alpha^2 mod n
                            <── A ──
  3. Computes 4 square roots
     of A mod n using CRT
  4. Picks one root beta
                            ── beta ──>
                                             5. Computes:
                                                gcd(alpha + beta, n)
                                                gcd(alpha - beta, n)
                                             6. Checks results:
                                                - Non-trivial factor -> Bob wins
                                                - Trivial (1 or n)  -> Alice wins
```

**Fairness:** Of Alice's 4 roots, exactly 2 are "safe" (beta = +/-alpha) and 2 are "dangerous" (beta != +/-alpha). If Alice picks honestly at random -> **50/50 chance**.

---

## Slide 4 — Why Choose p = q = 3 (mod 4)?

These are called **Blum primes**. They give us a clean formula for computing square roots:

```
x_p = A^((p+1)/4) mod p
x_q = A^((q+1)/4) mod q
```

**Verification with our primes:**
- `31 mod 4 = 3` (check)
- `43 mod 4 = 3` (check)

No need for a general-purpose root-finding algorithm — just one modular exponentiation per prime.

---

## Slide 5 — Demo Step 1: Alice Sends n

Alice picks `p = 31` and `q = 43` (kept secret), then publishes:

```
Alice sends n = p * q = 1333 to Bob.
```

> **Key point:** Bob receives `n = 1333` but has no efficient way to factor it back into 31 and 43 (in real cryptography, `n` would be thousands of bits long).

---

## Slide 6 — Demo Step 2: Bob Commits with A = alpha^2 mod n

Bob picks a random secret `alpha = 100` and computes:

```
A = 100^2 mod 1333 = 10000 mod 1333 = 669
```

```
Bob sends A = 669 to Alice.
```

> **Why this is a commitment:** Bob cannot change `alpha` later. Alice cannot recover `alpha` from `A` alone — squaring mod `n` is a one-way function without the factorization.

---

## Slide 7 — Demo Step 3: Alice Computes Roots mod p and q

Using her secret primes, Alice computes partial square roots:

```
x_p = 669^((31+1)/4) mod 31 = 669^8 mod 31 = 7
x_q = 669^((43+1)/4) mod 43 = 669^11 mod 43 = 14
```

```
x_p = +/- 7  (mod 31)
x_q = +/- 14 (mod 43)
```

> Each root has a positive and negative version, giving **2 x 2 = 4** combinations total.

---

## Slide 8 — Demo Step 4: CRT Produces 4 Roots mod n

Alice applies the Chinese Remainder Theorem to combine the four sign combinations:

| Signs (mod p, mod q) | CRT Result beta | Relation to alpha = 100 |
|:---------------------:|:---------------:|:-----------------------:|
| (+7, +14)            | **100**         | = alpha ← *safe*       |
| (+7, -14)            | **1061**        | != +/-alpha ← *dangerous* |
| (-7, +14)            | **272**         | != +/-alpha ← *dangerous* |
| (-7, -14)            | **1233**        | = n - alpha ← *safe*   |

```
Four square roots of A modulo n:
  beta = 100, 1061, 272, 1233
```

> All four satisfy beta^2 = 669 (mod 1333), but only two match Bob's secret +/-alpha.

---

## Slide 9 — Demo Step 5: Alice Sends beta, Bob Checks GCD

Alice chooses `beta = 272` and sends it to Bob.

Bob computes:

```
gcd(alpha + beta, n) = gcd(100 + 272, 1333) = gcd(372, 1333) = 31
gcd(alpha - beta, n) = gcd(100 - 272, 1333) = gcd(172, 1333) = 43
```

Both GCDs are **non-trivial factors** of `n`!

> Bob has successfully factored `n = 31 x 43`, breaking Alice's secret.

---

## Slide 10 — Result Summary

```
=== Summary ===
p = 31, q = 43, n = 1333
Bob chose alpha = 100
A = alpha^2 mod n = 669
Roots mod p: +/-7 (mod 31)
Roots mod q: +/-14 (mod 43)
Four roots beta (mod n): [100, 1233, 272, 1061]
Alice sent beta = 272
gcd(alpha + beta, n) = 31  <-- non-trivial factor!
gcd(alpha - beta, n) = 43  <-- non-trivial factor!

>>> RESULT: BOB WINS — he recovered p = 31 and q = 43.
```

---

## Slide 11 — Why Does the GCD Trick Work?

Since both `alpha` and `beta` are square roots of the same value `A`:

```
alpha^2 = beta^2 (mod n)
=> (alpha + beta)(alpha - beta) = 0 (mod n)
```

So `n = p * q` divides the product `(alpha + beta)(alpha - beta)`. Two cases arise:

| Scenario | What happens | Winner |
|----------|-------------|--------|
| **beta = +/-alpha (mod n)** | One of the factors is 0 mod n; GCD returns 1 or n (trivial) | **Alice** — Bob learns nothing |
| **beta != +/-alpha (mod n)** | Neither factor is 0 mod n, but their product is; `p` divides one factor and `q` divides the other | **Bob** — GCD reveals p or q |

In our demo, `beta = 272 != +/-100`, so Bob wins.

---

## Slide 12 — The "Coin Flip" Fairness Argument

Alice has **4 roots** to choose from. Exactly:
- **2 roots** are safe for Alice: `beta = alpha (100)` and `beta = n - alpha (1233)`
- **2 roots** are dangerous for Alice: `beta = 272` and `beta = 1061`

If Alice picks a root **uniformly at random** (the honest strategy):

```
P(Alice wins) = 2/4 = 50%
P(Bob wins)   = 2/4 = 50%
```

> This is a **fair coin flip** — neither party can gain an advantage without breaking the protocol.

**Can Alice cheat?** She doesn't know which roots are "safe" because she doesn't know Bob's `alpha`. She only knows the four roots. However, she *can* identify the +/-pairs — so in practice the protocol is usually extended with a commitment scheme to prevent Alice from always picking a safe root. For our class demo, we assume Alice picks honestly.

---

## Slide 13 — Security in Practice

| Aspect | Demo | Real-world |
|--------|------|------------|
| Prime size | 5 bits (`p = 31`, `q = 43`) | 1024-2048+ bits each |
| `n` size | 11 bits (`n = 1333`) | 2048-4096+ bits |
| Factoring difficulty | Trivial | Computationally infeasible |

- The protocol's security rests on the **hardness of integer factorization**.
- Same mathematical foundation as **RSA encryption**.
- Small primes are used here only so we can trace every step by hand.

---

## Slide 14 — Conclusion

1. **Trust is replaced by math** — no need for a trusted third party to flip the coin.
2. **Squaring is easy, un-squaring is hard** — this asymmetry is the core of the protocol.
3. **CRT gives Alice exactly 4 roots** — 2 safe, 2 dangerous — creating a fair 50/50 game.
4. **Bob's GCD check** is the verification step that detects whether Alice sent a "dangerous" root.
5. In our demo, Alice sent `beta = 272 != +/-alpha`, so **Bob won by factoring n**.

> *"We don't need to trust each other. We just need to trust the math."*

---

## Speaker Notes

1. "Imagine Alice and Bob need to settle a bet over the internet — but neither trusts the other to flip fairly."
2. "Alice commits to a hard problem: she publishes `n` but keeps its factors secret."
3. "Bob commits to a random number by squaring it — this locks in his choice."
4. "Alice uses her secret factors to compute all 4 square roots via CRT — this is something only she can do."
5. "She sends one root back. If it happens to match +/-alpha, Bob gets nothing. If not, Bob uses GCD to crack `n`."
6. "In our demo, Alice picked beta = 272, which is NOT +/-100, so Bob successfully factored 1333 = 31 * 43. Bob wins."
7. "The beauty: if Alice picks randomly, it's a perfect 50/50 — a fair coin flip, enforced by number theory."
