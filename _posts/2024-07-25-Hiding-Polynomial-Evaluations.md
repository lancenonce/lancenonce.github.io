# Hiding Polynomial Evaluations
![thunderbird, who inspired this article](/media/thunderbird.png)
### What is this blog about
The world of mathematics requires an abstract imagination that doesn't directly map to the physical world. Ever since growing an interest in programmable cryptography (zero knowledge, fully homomorphic encryption, multi party computation, etc.), I've been fascinated at the strange and wonderful things you can do with math.

For example, in the real world, if I wanted to ask you which countries you've been to, you'd show me your passport stamps or some photos from your camera reel to prove your experience. But what if I showed you nothing, and though you don't trust me, you subsequently know _for a fact_ that I actually travelled to said country?

This is what's possible with math, in particular **polynomials over finite fields**. This is why they are used everywhere in crypto. Often times data is encoded as a polynomial, and that polynomial is used to prove certain things about the data without revealing the entire polynomial (or in this case, any of it).

In this article, we'll explore an example of polynomials being used in cryptography (important to show the youngins asking "why is this math stuff important?") and explain how to hide a polynomial evaluation.

### Polynomial Extension Fields
In cryptography, polynomials are used frequently. An example is the finite extension field, where a field (typically a prime field) is extended to include polynomials whose coefficients are in the field and are reduced by a certain irreducible polynomial. An example I like from the Moon Math Manual is the field $$\mathbb{F}_{3^2}$$. We use the irreducible polynomial $$P(t) = t2 + 1$$ This field contains all polynomials of degree less than 2 with coefficients in $$\mathbb{F}_3$$. The elements included in this field are:

$$
\mathbb{F}_{3^2} = [{0,1,2,t,t+1,t+2,2t,2t+1,2t+2}]
$$

This is quite the group. It maintains all the properties of a group, so we can add and multiply these elements together to yield other elements of the group. 

The addition table looks like:

| +   | 0   | 1   | 2   | t   | t+1 | t+2 | 2t  | 2t+1 | 2t+2 |
|-----|-----|-----|-----|-----|-----|-----|-----|------|------|
| 0   | 0   | 1   | 2   | t   | t+1 | t+2 | 2t  | 2t+1 | 2t+2 |
| 1   | 1   | 2   | 0   | t+1 | t+2 | t   | 2t+1| 2t+2 | 2t   |
| 2   | 2   | 0   | 1   | t+2 | t   | t+1 | 2t+2| 2t   | 2t+1 |
| t   | t   | t+1 | t+2 | 2t  | 2t+1| 2t+2| 0   | 1    | 2    |
| t+1 | t+1 | t+2 | t   | 2t+1| 2t+2| 2t  | 1   | 2    | 0    |
| t+2 | t+2 | t   | t+1 | 2t+2| 2t  | 2t+1| 2   | 0    | 1    |
| 2t  | 2t  | 2t+1| 2t+2| 0   | 1   | 2   | t   | t+1  | t+2  |
| 2t+1| 2t+1| 2t+2| 2t  | 1   | 2   | 0   | t+1 | t+2  | t    |
| 2t+2| 2t+2| 2t  | 2t+1| 2   | 0   | 1   | t+2 | t    | t+1  |


The multiplication table looks like:

| ·   | 0   | 1   | 2   | t   | t+1 | t+2 | 2t  | 2t+1 | 2t+2 |
|-----|-----|-----|-----|-----|-----|-----|-----|------|------|
| 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0   | 0    | 0    |
| 1   | 0   | 1   | 2   | t   | t+1 | t+2 | 2t  | 2t+1 | 2t+2 |
| 2   | 0   | 2   | 1   | 2t  | 2t+2| 2t+1| t   | t+2  | t+1  |
| t   | 0   | t   | 2t  | 2   | 2t+1| 1   | t+2 | 2    | t+1  |
| t+1 | 0   | t+1 | 2t+2| 2t+1| 2   | 2t  | 1   | t+2  | 2    |
| t+2 | 0   | t+2 | 2t+1| 1   | 2t  | 2   | t+1 | 2t+2 | 2t   |
| 2t  | 0   | 2t  | t   | t+2 | 1   | 2t+2| 2   | t+1  | 2t+1 |
| 2t+1| 0   | 2t+1| t+2 | t+1 | 2t+2| 2   | 2t  | 1    | 2t   |
| 2t+2| 0   | 2t+2| t+1 | 2   | 2t  | t   | 2t+1| 2    | 1    |

___

For example, {$$t+1$$} $$+$$ $$t$$ is $$2t + 1$$ because we add $$t$$ to the first $$t$$.

And $$2t \cdot 2t$$ is $$2$$ because $$2t \cdot 2t = 4t^2$$, and if we reduce that by $$P(t) = t^2 + 1$$, we get:

$$
2t \cdot 2t = 4t^2
$$


Since we are working in $$\mathbb{F}_3$$, we reduce coefficients modulo 3:

$$
4 \equiv 1 \pmod{3}
$$


Therefore:

$$
4t^2 \equiv t^2
$$


Using the irreducible polynomial $$P(t) = t^2 + 1$$, we substitute $$t^2$$ with $$-1$$ (or equivalently $$2$$ in $$\mathbb{F}_3$$):

$$
t^2 \equiv -1 \equiv 2 \pmod{3}
$$


Substituting this back, we get:

$$
4t^2 \equiv t^2 \equiv 2
$$


Therefore:

$$
2t \cdot 2t = 2
$$


It's neat!

### Bilinear Pairings
So why use these extension fields? They have lots of uses, but arguably my favorite one is the usage in bilinear pairings.

I like to think of bilinear pairings as a way to make an equation $$A \times B \rightarrow C$$ where each element $$A, B,$$ and $$C$$ are in related [groups](https://en.wikipedia.org/wiki/Group_(mathematics)) (A and B can belong to the same group). The $$\times$$ here is some arbitrary combination (most commonly exponentiation of [generators](https://en.wikipedia.org/wiki/Generating_set_of_a_group)), not necessarily multiplication. Let's clarify:

A bilinear pairing is a map:
$$e: G_1 × G_2 → G_T $$ where $$G_1$$, $$G_2$$, and $$G_T$$ are groups of the same prime order $$p$$ (Prime order means that the size of the group is a prime number; this helps with making [trapdoor](https://eprint.iacr.org/2011/131.pdf) values). $$e$$ must satisfy these properties of Bilinearity, Non-degeneracy, and Computability.

**Bilinearity**: 
> ∀ $$u$$ ∈ $$G_1$$, $$v$$ ∈ $$G_2$$, {$${a, b}$$} ∈ $$\mathbb{Z}_p$$:
>
> $$e(u^a,v^b) = e(u, v)^{ab}$$
$$
∀ u ∈ G_1, v ∈ G_2, {a, b} ∈ \mathbb{Z}_p: e(u^a, v^b) = e(u,v)^{ab}
$$

**Non-degeneracy**: 
> ∃ $$u$$ ∈ $$G_1$$ & $$v$$ ∈ $$G_2$$ ∣ $$e(u, v)$$ ≠ 1

**Computability**: 
> $$e(u,v)$$ is computable ∀ $$u ∈ G_1$$ & $$v ∈ G_2$$ 

The intuition behind these properties is that with a bilinear pairing, you can take the pairing of two values and map their relationship to another value in an extension field (such as the polynomial extension field we observed above). 

It's convenient to do this because the extension field is inherently larger than the initial fields. So multiplying two of the initial field values together will simply get you something in the extension rather than a wrap-around value from the order modulus.

Notice the hiding feature in the bilinearity property. Let's say we both know $$u$$ and $$v$$ and I give you $$a$$ and take $$b$$ as my own secret. I can share with you $$v^b$$ and you can give me $$u^a$$ and we both can confirm the integrity of these secrets later. The equation works out to $$e(u, v)^{ab}$$ if and only if all the values are not corrupted.

This is critical in cryptography where people hold secret values they use for security. In a zero knowledge scheme, I can keep $$b$$ to myself, but prove to you that $$b$$ is the valid value that we agreed on. It gets extra fancy when $$b$$ is something like a linear combination of data that has been encoded. In the next section, let's think about how a protocol like this could be used to hide polynomial evaluations (polynomials are just linear combinations of variables with coefficients and exponents).

### Hiding Polynomial Evaluations
When hiding an evaluation, the first thing we need to do is define our bilinear pairing. We find a sufficiently large prime number $$p$$ and let $$G_1$$ and $$G_2$$ be cyclic groups of order $$p$$. We then let $$G_T$$ be the group of an extension field, such as $$\mathbb{F}_{p^k}$$, where $$k$$ is the maximum degree of the polynomials in the extension field (remember the example with $$\mathbb{F}_{3^2}$$ above).

Now, let's select a polynomial (in the crypto world, this polynomial will represent some data we want to hide such as a stack trace, regular expression, an L/R/O circuit in R1CS, etc.). Let's take the generic $$P(x)$$:

$$
P(x) = a_0 + a_1x + a_2x^2 + \dots + a_dx^d
$$
where the values of $$a$$ are in $$\mathbb{F}_{p}$$ and $$d$$ ≤ $$k$$. 

We will have another piece on generators, but refer to the last link about generators to learn more. In order to use the hiding property we mentioned earlier ($$u^a$$), we use a generator $$g_1$$ to be a generator of $$G_1$$ and encode each coefficient $$a_i$$ as $$g_1^{a_i}$$. 

The next step is to take this encoded polynomial and evaluate it at another encoded point. We effctively hide the polynomial from the evaluating party and hide the point from the polynomial owning party. This is like separating the waters of the Red Sea, impossible until it's done. 

We want to evaluate the polynomial at $$x$$. We let $$g_2$$ be the generator for $$G_2$$ and encode $$x$$ as $$g_2^x$$. 

Finally, we use the bilinear pairing to compute all the elements of the polynomial:

$$
e(g_1^{a_0},g_2^1) + e(g_1^{a_1}, g_2^x) + e(g_1^{a_2}, g_2^{x^2}) + \dots + e(g_1^{a_d}, g_2^{x^d})
$$

Following the bilinearity property we defined before, we can abstract this to:

$$
e(g1, g2)^{a_0 * 1} + e(g_1, g_2)^{a_1 * x} + e(g_1, g_2)^{a_2 * x^2} + \dots + e(g_1, g_2)^{a_d*x^d}
$$

Essentially, it's:

$$
e(g_1, g_2)^{P(x)}
$$

Ain't that some shit??

Now, if I were to commit to a polynomial, I can hide this polynomial while allowing you to evaluate certain points and constrain that the evaluations are equal to some value, say the equivalent of $$0$$ in $$G_T$$. Beautiful!
### A Rusty Example
Rust is a powerful language when it comes to cryptography due to its speed and security. The [Arkworks](https://github.com/arkworks-rs) library is a Rust crate that allows low level implementation of crypto-relevant math such as this notion of homomorphic hiding.

In this Github [repository](https://github.com/lancenonce/hiding), we implement a *very* simple implementation of what it would look like to hide a polynomial from a verifier and a point from a prover. 

We start with a prompt asking the prover for their polynomial:

```shell
    let poly = input_polynomial();
```
where `input_polynomial()` is a function that prompts for a shell input and returns a `DensePolynomial<ark_bn254::Fr>` polynomial of the coefficients given by the user. For example:

```shell
Enter the polynomial coefficients (u64), separated by spaces:
8 8 8
```
this will yield:
$$
8x^2 + 8x + 8
$$

After the polynomial is established, the verifier verifies the point at the hidden $$x$$ value of 1:

```shell
Verifier, enter the point you want to secretly evaluate at (u64):
1
The sample passed!
The polynomial was evaluated at point: Fp256 "(0000000000000000000000000000000000000000000000000000000000000001)"
The evaluation result is: Fp256 "(0000000000000000000000000000000000000000000000000000000000000018)"
```
It may not be intuitive (and I know, they should be in separate threads), but in this case, the verifier has no idea what the polynomial was. Nor does the prover know what point was used for the evaluation. 

Also, observe that we use the $$\mathbb{F}_{256}$$ field, which gives us 128 bits of security. 

And voila! We have an implementation of a core cryptographic protocol that can be built upon to create a zk SNARK.

Contributions are welcome! You'll see a few todos. If you're reading this close to the time it's posted, the repo barely has an implementation. Give it a go. Hopefully over time we'll implement the full protocol with robust security.

### Reflection
I hope I did a good job making the unknown known to you. I had lots of fun writing this. Expect more deep crypto content like this in the future. 

As always, thanks for reading!