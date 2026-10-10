## HITCON CTF – Crypto – \so easy rsa

### Provided Files

* `chal.py`: RSA key-generation logic. The primes $p, q$ are not generated with secure randomness, but from a **Linear Congruential Generator** (LCG).
    ```python!
    from gmpy2 import next_prime, is_prime
    from random import randint
    from Crypto.Util.number import bytes_to_long

    class Rand:
        def __init__(self):
            self.seed = randint(2, 2**512)
            self.A = next_prime(randint(2, 2**512))
            self.B = next_prime(randint(2, 2**512))
            self.M = next_prime(randint(2, 2**512))
            for _ in range(10000):
                self.next()

        def next(self):
            self.seed = self.seed * self.A + self.B
            self.seed = self.seed % self.M
            return self.seed

        def __str__(self):
            return f"{self.A}, {self.B}, {self.M}"


    def gen_prime(r):
        while True:
            v = r.next()
            if is_prime(v):
                return v

    r = Rand()
    p,q = gen_prime(r), gen_prime(r)
    n = p*q
    e = 65537
    flag = bytes_to_long(open('flag','rb').read())
    val = pow(flag, e, n)

    print(n)
    print(r)
    print(val)

    ```

* `output`: contains $n, A, B, M, c$:

  ```
  n
  A, B, M
  c
  ```
  
  ```
  258720332534100287234458110010185155261130153254107442235854414506170536384060781366385258399243052901972558998237319622949803003628352468038779889447749958031079924389696834908608708311694932037259947294352067926932155092194117105684832481620765614657816427294107055109317830777750924756091556427290450319
    5005408141313345292407945953835071149806179641852075007515684286050215929954837846280174872639627250238013460230160536843037257042746543278346526830149401, 10298048151006208752433869042311201313715296070490185742808091797981718158025069645386345886679150152180335525547545682404445248373464275107700557029336907, 604303033052239158158537830898604399794738462357731223894068010903732221760427899769617718944018972013962181770068766282586698417988133109292194814390223
    254133282227653630180821066744315140149066310218246354286254540471151103799946109571434944707352386885808429309340377597941420587423387023258637198593238599440369492997225370381912614338035497842921842405804013255379659887004174563901115006873037426232846670437444057875737001884961148116954898910964947274

  ```

### Observations

From `chal.py`, we can see:

```python
x_{t+1} = (A * x_t + B) % M
```

Then:

* Take one LCG output as $p$ (a prime)
* Take the output $k$ steps later as $q$ (a prime)
* $n = p \times q$
* RSA public key $(n, e)$, where $e = 65537$

Since LCG parameters $A, B, M$ and $n, c$ are all known, the problem becomes:
**How to recover $p, q$ from the LCG relation and decrypt the flag.**

---

### Analysis

### LCG Recurrence Expansion

Define:

$$
A_k \equiv A^k \pmod M
$$

$$
C_k \equiv B \cdot (1 + A + A^2 + \dots + A^{k-1}) \pmod M
$$

This can be computed recursively:

$$
C_1 = B,\quad C_{t+1} \equiv A\cdot C_t + B \pmod M
$$

From LCG properties:

$$
q \equiv A_k \cdot p + C_k \pmod M
$$

### Deriving the Quadratic Congruence

Given $n = p q$, substitute:

$$
n \equiv p \cdot q \equiv p (A_k p + C_k) \pmod M
$$

$$
\Rightarrow A_k p^2 + C_k p - n \equiv 0 \pmod M
$$

This is a quadratic equation modulo $M$:

$$
a p^2 + b p - n \equiv 0 \ (\bmod\ M)
$$

where $a = A_k \bmod M,\ b = C_k \bmod M$.

---

## Solution

1. **Enumerate $k$**
   Since the challenge does not tell how many steps apart $p, q$ are, directly try $k = 1 \dots K_{\max}$ (for this challenge, $K_{\max} = 50000$ is enough).

2. **Solve the quadratic modulo $M$**
   Discriminant:

   $$
   D \equiv b^2 + 4 a n \pmod M
   $$

   If $D$ has a square root modulo $M$ (via Tonelli–Shanks), then:

   $$
   p \equiv \frac{-b \pm \sqrt{D}}{2a} \pmod M
   $$

3. **Validate $p$**
   For each candidate $p$, compute $g = \gcd(n, p)$:

   * If $1 < g < n$, you found the real $p$.
   * The other prime is $q = n / p$.

4. **RSA Decryption**

   $$
   \varphi(n) = (p-1)(q-1),\quad d \equiv e^{-1} \pmod{\varphi(n)}
   $$

   Plaintext:

   $$
   m \equiv c^d \pmod n
   $$

   Convert to bytes to recover the flag.

---

## Exploit Code

```python
from math import gcd

# Tonelli–Shanks modular square root
def legendre(a, p): return pow(a, (p - 1) // 2, p)
def mod_sqrt(n, p):
    if n == 0: return 0
    if p % 4 == 3: return pow(n, (p + 1) // 4, p)
    if legendre(n, p) != 1: return None
    q, s = p - 1, 0
    while q % 2 == 0:
        q //= 2; s += 1
    z = 2
    while legendre(z, p) != p - 1:
        z += 1
    c = pow(z, q, p)
    x = pow(n, (q + 1) // 2, p)
    t = pow(n, q, p)
    m = s
    while t != 1:
        i, t2i = 1, pow(t, 2, p)
        while t2i != 1:
            t2i = pow(t2i, 2, p)
            i += 1
        b = pow(c, 1 << (m - i - 1), p)
        x = (x * b) % p
        t = (t * b * b) % p
        c = (b * b) % p
        m = i
    return x

def inv(a, m): return pow(a, -1, m)

# Provided challenge data
n = 258720332534100287234458110010185155261130153254107442235854414506170536384060781366385258399243052901972558998237319622949803003628352468038779889447749958031079924389696834908608708311694932037259947294352067926932155092194117105684832481620765614657816427294107055109317830777750924756091556427290450319
A = 5005408141313345292407945953835071149806179641852075007515684286050215929954837846280174872639627250238013460230160536843037257042746543278346526830149401
B = 10298048151006208752433869042311201313715296070490185742808091797981718158025069645386345886679150152180335525547545682404445248373464275107700557029336907
M = 604303033052239158158537830898604399794738462357731223894068010903732221760427899769617718944018972013962181770068766282586698417988133109292194814390223
c = 254133282227653630180821066744315140149066310218246354286254540471151103799946109571434944707352386885808429309340377597941420587423387023258637198593238599440369492997225370381912614338035497842921842405804013255379659887004174563901115006873037426232846670437444057875737001884961148116954898910964947274
e = 65537

Ak, Ck = A % M, B % M
for k in range(1, 50000):
    a, b = Ak, Ck
    D = (b*b + (4*a % M)*(n % M)) % M
    r = mod_sqrt(D, M)
    if r is not None:
        inv2a = inv((2*a) % M, M)
        for cand in (((-b + r) * inv2a) % M, ((-b - r) * inv2a) % M):
            g = gcd(n, cand)
            if 1 < g < n:
                p = g
                q = n // g
                phi = (p-1)*(q-1)
                d = pow(e, -1, phi)
                m = pow(c, d, n)
                print(int.to_bytes(m, (m.bit_length()+7)//8, 'big'))
                exit()
    Ak = (Ak * A) % M
    Ck = (Ck * A + B) % M
```

### Result

Final result:

* $k = 827$
* $p, q$ are each about 508 bits

**Flag: hitcon{too_weak_randomnesssssss}**

### Key Takeaways
* If LCG outputs are directly used as primes, they leave linear relations modulo $M$.
* With known $A, B, M$, unknown $p, q$ can be linked into a quadratic equation modulo $M$, then recovered through modular square roots.
* Enumerating step count $k$ is necessary, but in practice $k$ is usually not too large.
