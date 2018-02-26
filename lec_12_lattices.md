# Lattice based crypto



Lattice based public key encryption (and its cousins known as knapsack and coding based encryption) have almost as long a history as discrete logarithm
and factoring based schemes.
Already in 1976, right after the Diffie-Hellman key exchange was discovered (and before RSA), Ralph Merkle was working on building public key encryption from the
NP hard _knapsack_ problem (see [Diffie's recollection](http://cr.yp.to/bib/1988/diffie.pdf)). This can be thought of as the task of solving a linear equation of the
form $Ax = y$ (where $A$ is a given matrix, $y$ is a given vector, and the unknown are $x$) over the real numbers but with the additional constraint that $x$ must be either $0$ or $1$.
His proposal evolved into the Merkle-Hellman system proposed in 1978 (which was broken in 1984).


McEliece proposed in 1978 a system based on the difficulty of the decoding problem for general linear codes. This is the task of solving _noisy linear equations_ where one is given $A$ and $y$ such that $y=Ax+e$ for a "small" error vector $e$, and needs to recover $x$.
Crucially, here we work in a finite field, such as working modulo $q$ for some prime $q$ (that can even be $2$) rather than over the reals or rationals.
There are special matrices $A^*$ for which we know how to solve this problem efficiently: these are known as efficiently decodable [error correcting codes](https://goo.gl/vM7Pvv).
McEliece suggested a scheme where the key generator lets $A$ be a "scrambled" version of a special $A^*$ (based on the [Goppa algebraic geometric code](https://goo.gl/Vd4yye)).
So, someone that knows the scrambling could solve the problem, but (hopefully) someone that doesn't know it wouldn't.
McEliece's system has so far not been broken.


In a 1996 breakthrough, Ajtai showed a _private key_ scheme based on integer lattices that had a very curious property- its security could be based on the assumption that certain problems were only hard in the _worst case_, and moreover variants of these problems were known to be NP hard.
This re-ignited the  hope that we could perhaps realize the old dream of basing crypto on the mere assumption that $P\neq NP$.
Alas, we now understand that there are fundamental barriers to this approach.

Nevertheless, Ajtai's work attracted significant interest, and within a year both Ajtai and Dwork, as well as Goldreich, Goldwasser and Halevi came up with lattice based constructions for _public key_ encryption (the former based also on _worst case_ assumptions).
At about the same time,  Hoffstein, Pipher, and Silverman came up with their NTRU public key system which is based on stronger assumptions but offers better performance, and they started a company around it together with Daniel Lieman.

You may note that I haven't yet said what _lattices_ are; we will do so later, but for now if you simply think of questions involving linear equations modulo some prime $q$, you will get enough of the intuition that you need.
(The lattice viewpoint is more geometric, and we'll discuss it more below; it was first used to _attack_ cryptosystems and in particular break the Merkle-Hellman knapsack scheme and many of its variants.)

Lattice based cryptography has captured a lot of attention recently from both theory and practice.
In the theory side, many cool new constructions are now based on lattice based cryptography, and chief among them fully homomorphic encryption,
as well as indistinguishability obfuscation (though the latter's security's foundations are still far less solid).
On the applied side, the steady advances in the technology of quantum computers have finally gotten practitioners worried about RSA, Diffie Hellman and Elliptic Curves.
While current constructions for quantum computers are nowhere near being able to, say, factor larger numbers that can be done classically (or even than can be done by hand), given that it takes many years to develop new standards and get them deployed, many believe the effort to transition away from these factoring/dlog based schemes should start today (or perhaps should have started several years ago).
The NSA has [suggested](https://www.nsa.gov/ia/programs/suiteb_cryptography/index.shtml) that it plans to initiate the process to "transition to quantum resistant algorithms in the not too distant future"; see also this [very interesting FAQ](https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf) on this topic.

Cryptography has the peculiar/unfortunate feature that if a machine is built that can factor large integers in 20 years, it can still be used to break the communication we transmit _today_, provided this communication was recorded.
So, if you have some data that you expect you'd want still kept secret in 20 years (as many government and commercial entities do), you might have reasons to worry.
Currently lattice based cryptography is the only real "game in town" for potentially quantum-resistant public key encryption schemes.


Lattice based cryptography is a huge area, and in this lecture and this course we only touch on few aspects of it.
I highly recommend [Chris Peikert's Survey](https://web.eecs.umich.edu/~cpeikert/pubs/lattice-survey.pdf) for a much more in depth treatment of this area.


## A world without Gaussian elimination

The general approach people used to get a public key encryption is to obtain a hard computational problem with some mathematical _structure_.
We've seen this in the _discrete logarithm_ problem, where the task is to invert the map $a \mapsto g^a \pmod{p}$, and the integer factoring problem,
where the task is to invert the map $a,b \mapsto a\cdot b$.
Perhaps the simplest structure to consider is the task of solving linear equations.


Pretend that we didn't know of Gaussian elimination,^[Despite the name, [Gaussian elimination](https://goo.gl/3HNb5U)  has been known to Chinese mathematicians since 150BC or so, and was popularized in the west through the 1670 notes of Isaac Newton.] and that if we picked a "generic" matrix $A$ then the map $x \mapsto Ax$ would be hard to invert.
(Here and elsewhere, our default interpretation of a vector $x$ is as a _column_ vector, and hence if $x$ is $n$ dimensional and $A$ is $m\times n$ then $Ax$ is $m$ dimensional. We use $x^\top$ to denote the row vector obtained by _transposing_ $x$.)
Could we use that to get a public key encryption scheme?

Here is a concrete approach.
Let us fix some prime $q$ (think of it as polynomial size, e.g., $q$ is smaller than $1024$ or so, though people can and sometimes do consider $q$ of exponential size),
and all computation below will be done modulo $q$.
The secret key is a vector $x\in\Z_q^n$, and the public key is $(A,y)$ where $A$ is a random $m\times n$ matrix with entries in $\Z_q$ and $y=Ax$.
Under our assumption, it is hard to recover the secret key from the public key, but how do we use the public key to encrypt?

The crucial observation is that even if we don't know how to solve linear equations, we can still combine several equations to get new ones.
To keep things simple, let's consider the case of encrypting a single bit.

> # { .pause }
If you have a CPA secure public key encryption scheme for single bit messages then you can extend it to a CPA secure encryption scheme for messages of any length.
Can you see why?


We think of the public key as the set  of equations $\iprod{a_1,x}=y_1,\ldots, \iprod{a_m,x}=y_m$ in the unknown variables $x$.
The idea is that to encrypt the value $0$ we will generate a new _correct_ equation on $x$, while to encrypt the value $1$ we will generate an _incorrect_ equation.
To decrypt a ciphertext $(a,\sigma)\in \Z_q^{n+1}$, we think of it as an equation of the form $\iprod{a,x}=\sigma$ and output $1$ if and only if the equation is correct.

How does the encrypting algorithm, that does not know $x$, get a correct or incorrect equation on demand?
One way would be to simply take two equations $\iprod{a_i,x}=y_i$ and $\iprod{a_j,x}=y_j$ and add them together to get the equation $\iprod{a_i+a_j,x}=y_i+y_j$.
This equation is correct and so one can use it to encrypt $0$, while to encrypt $1$ we simply add some fixed nonzero number $\alpha\in\Z_q$ to the right hand side to get the incorrect equation $\iprod{a_i+a_j,x}= y_i+y_j + \alpha$.
However, even if it's hard to solve for $x$ given the equations, an attacker (who also knows the public key $(A,y)$) can try itself all pairs of equations and do the same thing.

Our solution for this is simple- just add more equations! If the encryptor adds a random subset of equations then there are $2^m$ possibilities for that, and an attacker can't guess them all.
Thus, at least intuitively, the following encryption scheme would be "secure" in the Gaussian-elimination free world of attackers that haven't taken freshman linear algebra:

>__Scheme LwoE-ENC:__  Public key encryption under the hardness of "learning linear equations without errors".
>
* _Key generation_:  Pick random $m\times n$ matrix $A$ over $\Z_q$, and $x\leftarrow_R\Z_q^n$, the secret key is $x$ and the public key is $(A,y)$ where $y=Ax$.
>
* _Encryption_: To encrypt a message $b\in\{0,1\}$, pick $w\in\{0,1\}^m$ and output $w^\top A,\iprod{w,y}+\alpha b$ for some fixed nonzero $\alpha\in\Z_q$.
>
* _Decryption:_ To decrypt a ciphertext $(a,\sigma)$, output $0$ iff $\iprod{a,x}=\sigma$.




> # { .pause }
Please stop here and make sure that you see why this is a valid encryption, and this description corresponds to the previous one; as usual all calculations are done modulo $q$.



## Security in the real world.

Like it or not (and cryptographers typically don't) Gaussian elimination _is_ possible in the real world and the scheme above is completely insecure.
However, the Gaussian elimination algorithm is extremely _brittle_.  
Errors tend to be amplified when you combine equations.
This is usually thought of as a bad thing, and numerical analysis is much about dealing with issue.
However, from the cryptographic point of view, these errors can be our saving grace and enable us to salvage the security of the ridiculous scheme above.


To see why Gaussian elimination is brittle, let us recall how it works.
Think of $m=n$ for simplicity.
Given equations $Ax=y$ in the unknown variables $x$, the goal of Gaussian elimination is to transform them into the equations $Ix = y'$ where $I$ is the identity matrix (and hence the solution is simply $x=y'$).
Recall how we do it: by rearranging and scaling, we can assume that the top left corner of $A$ is equal to $1$, and then we add the first equation to the other equations (scaled appropriately) to zero out the first entry in all the other rows of $A$ (i.e., make the first column of $A$ equal to $(1,0,\ldots,0)$) and continue onwards to the second column and so on and so forth.

Now, suppose that the equations were _noisy_, in the sense that we added to $y$ a vector $e\in\Z_q^m$ such that $|e_i|<\delta q$ for every $i$.^[Over $\Z_q$, we can think of $q-1$ also as the number $-1$, and so on. Thus if $a\in\Z_q$, we define $|a|$ to be the minimum of $a$ and $q-a$. This ensures the absolute value satisfies the natural property of  $|a|=|-a|$.]
Even ignoring the effect of the scaling step, simply adding the first equation to the rest of the equations would typically tend to increase the  relative error of equations $2,\ldots,m$  from $\approx \delta$ to $\approx 2\delta$.
Now, when we repeat the process, we increase the error of equations $3,\ldots,m$ from $\approx 2\delta$ to $\approx 4\delta$, and we see that by the time we're done dealing with about $n/2$ variables, the remaining equations have error level roughly $2^{n/2}\delta$.
So, unless $\delta$ was truly tiny (and $q$ truly big, in which case the difference between working in $\Z_q$ and simply working with integers or rationals disappears), the resulting equations have the form $Ix = y' + e'$ where $e'$ is so big that we get no information on $x$.

The _Learning With Errors (LWE)_ conjecture is that this is _inherent_:

>__Conjecture (Learning with Errors, Regev 2005):__ Let $q=q(n)$ and $\delta=\delta(n)$ be some functions.
The _Learning with Error (LWE) conjecture with respect to $q,\delta$_, is that for every polynomial-time adversary $E$
and $m=poly(n)$, the probability that $E(A,Ax+e)=x$ is negligible, where $A$ is a random $m\times n$ matrix in $\Z_q$,
$x$ is random in $\Z_q^n,$ and $e \in \Z_q^m$ is a random noise vector with magnitude $\delta q$.[^noise]
>
The _LWE conjecture_ is that for every polynomial $p(n)$ there is some polynomial $q(n)$ such that LWE holds with respect to $q(n)$ and $\delta(n)=1/p(n)$.[^superpoly]


[^noise]: One can think of $e$ as chosen by simply letting every coordinate be chosen at random in $\{ -\delta q, -\delta q + 1 , \ldots, +\delta q \}$. For technical reasons, we sometimes consider other distributions and in particular the _discrete Gaussian_ distribution which is obtained by letting every coordinate of $e$ be an independent Gaussian random variable with standard deviation $\delta q$, conditioned on it being an integer. (A closely related distribution is obtained by picking such a Gaussian random variable and then rounding it to the nearest integer.)

[^superpoly]: People sometimes also consider variants where both $p(n)$ and $q(n)$ can  be as large as exponential.


## Search to decision

It turns out that if the LWE is hard, then it is even hard to distinguish between random equations and nearly correct ones:


> # {.theorem title="Search to decision reduction for LWE" #LWEsearchtodecthm}
If the LWE conjecture is true then for every $q=poly(n)$ and $\delta=1/poly(n)$ and $m=poly(n)$, the following two distributions are computationally
indistinguishable:
>
* $\{ (A,Ax+e) \}$ where $A$ is random $m\times n$ matrix in $\Z_q$, $x$ is random in $\Z_q^n$ and $e\in \Z_q^m$ is random noise vector of magnitude $\delta$.
>
* $\{ (A,y) \}$ where $A$ is random $m\times n$ matrix in $\Z_q$ and $y$ is random in $\Z_q^m$

> # {.proof data-ref="LWEsearchtodecthm"}
Suppose that we had a decisional adversary $D$ that succeeds in distinguishing the two distributions above with bias $\epsilon$.
For example, suppose that $D$ outputs $1$ with probability $p+\epsilon$ on inputs from the first distribution, and outputs  $1$ with probability $p$
on inputs from the second distribution.
>
We will show how we can use this to obtain a polynomial-time algorithm $S$ that on input $m$ noisy equations on $x$ and a value $a\in\ Z_q$, will learn with high probability whether or not the first coordinate of $x$ equals $a$.
Clearly, we can repeat this for all the possible $q$ values of $a$ to learn the first coordinate exactly, and then continue in this way to learn all coordinates.
>
Our algorithm $S$ gets as input the pair $(A,y)$ where $y=Ax+e$ and we need to decide whether $x_1 = a$.
Now consider the instance $A+(r\|0^m\|\cdots \|0^m),y+ar$, where $r$ is a random vector in $\Z_q^m$ and the matrix $(r\|0^m\|\cdots \|0^m)$ is simply the matrix with first column equal to $r$ and all other columns equal to $0$. Note that if $A$ is random then $A+r\|0^m\|\cdots \|0^m)$ is random as well.
Now note that $Ax + (r|0^m\cdots \|0^m)x = Ax + x_1 r$ and hence if $x_1 = a$ then we still have an input of the same form.
However, if $x_1 \neq a$ then this amounts to adding a non-zero multiple of the random vector $r$ to the noise vector, and hence we get an instance of the form $(A',y')$
with random $A',y'$.  
Hence if we send this input to our the decision algorithm $D$, then we would get $1$ with probability $p+\epsilon$  if $x_1=a$ and an output of $1$ with probability $p$ otherwise.
>
Now the crucial observation is that if our decision algorithm $D$ requires $m$ equations to succeed with bias $\epsilon$, we can use $100mn/\epsilon^2$ equations (which is still polynomial) to invoke it $100n/\epsilon^2$ times.
This allows us to distinguish with probability $1-2^{-n}$ between the case that $D$ outputs $1$ with probability $p+\epsilon$ and the case that it outputs $1$ with probability $p$ (this follows from the Chernoff bound we discussed in the mathematical background handout; can you see why?).
Hence by using polynomially more samples than the decision algorithm $D$, we get a search algorithm $S$ that can actually recover $x$.

## An LWE based encryption scheme

We can now show the secure variant of our original encryption scheme:


>__LWE-based encryption LWEENC:__
>
>
* _Parameters:_ Let $\delta(n)=1/n^4$ and let $q=poly(n)$ be a prime such that LWE holds w.r.t. $q,\delta$. We let $m = n^2\log q$.
>
* _Key generation:_ Pick $x\in\Z_q^n$. The private key is $x$ and the public key is $(A,y)$ with $y=Ax+e$ with $e$ a $\delta$-noise vector and $A$ a random $m\times n$ matrix.
>
* _Encrypt:_ To encrypt $b\in\{0,1\}$ given the key $(A,y)$, pick $w\in\{0,1\}^m$ and output $w^\top A, \iprod{w,y}+b\floor{q/2}$ (all computations are done in $\Z_q$).
>
* _Decrypt:_ To decrypt $(a,\sigma)$, output $0$ iff $|\iprod{a,x}-\sigma|<q/10$.

 \



Unlike our typical schemes,
here it is not immediately clear that this encryption is valid,
in the sense that the decrypting an encryption of $b$ returns the value $b$. But this is the case:

> # {.lemma #LWEcorrectlem}
With high probability, the decryption of the encryption of $b$ equals $b$.

> # {.proof data-ref="LWEcorrectlem"}
$\iprod{w^\top A,x} = \iprod{w,Ax}$. Hence, if $y=Ax+e$ then $\iprod{w,y} = \iprod{w^\top A,x} + \iprod{w,e}$.
But since every coordinate of $w$ is either $0$ or $1$, $|\iprod{w,e}|<\delta m q < q/10$ for our choice of parameters.[^cancellations]
So, we get that if $a= w^\top A$ and $\sigma = \iprod{w,y}+b\floor{q/2}$ then $\sigma - \iprod{a,x} = \iprod{w,e} + b\floor{q/2}$ which will be smaller than
$q/10$ iff $b=0$.

[^cancellations]: In fact, due to the fact that the _signs_ of the error vector's entries are different, we expect the errors to have significant cancellations and hence we would expect $|\iprod{w,e}|$ to only be roughly of magnitude $\sqrt{m}\delta q$, but this is not crucial for our discussions.

We now prove security of the LWE based encryption:

> # {.theorem title="CPA security of LWEENC" #LWEENCthm}
If the LWE conjecture is true then LWEENC is CPA secure.


For a public key encryption scheme with messages that are just bits, CPA security  means that an encryption of $0$ is indistinguishable from an encryption of $1$, even given the public key.
Thus [LWEENCthm](){.ref}  will follow from the following lemma:


> # {.lemma #LWEENClem}
Assuming the LWE, the distribution $(A,y=Ax+e),(w^\top A,\iprod{w^\top,y})$ (where all values are chosen as above) is indistinguishable from the distribution
$(A,y),(a,\sigma)$ where $y$ is completely random in $\Z_q^m$, $a$ is random and independent in $\Z_q^n$ and $\sigma$ is random and independent in $\Z_q$.


> # { .pause }
You should stop here and verify that [LWEENClem](){.ref} implies [LWEENCthm](){.ref}.
The idea is that it shows that the concatenation of the public key and encryption of $0$ is indistinguishable from something that is completely random, and you can use it to show that the concatenation of the public key and encryption of $1$ is indistinguishable from the same thing, and then finish using the hybrid argument.

We now prove [LWEENClem](){.ref}, which will complete the proof of [LWEENCthm](){.ref}.

> # {.proof data-ref="LWEENClem"}
By the search to decision reduction, the distribution above is indistinguishable from the distribution where $y$ is completely random. However, in this case I claim that if we choose $w$ at random in $\{0,1\}^m$ and let $(a,\sigma) = w^\top(A\|y)$ then $(a,\sigma)$ would be a (close to) completely uniform and independent
vector in $\Z_q^{n+1}$.
We will not do the whole proof (which uses the mod $q$ version of the [leftover hash lemma](https://goo.gl/KXpccP) which  we mentioned before and is also "Wikipedia-able") but the idea is simple.
Let $A' = (A\|y)$ which in our case is a completely random matrix.
This mans that the map $w \mapsto w^\top A'$ is essentially a _pairwise independent_ hash function mapping $\Z_q^m$ to $\Z_q^{n+1}$.
Now when we choose $w$ at random in $\{0,1\}^m$, it is coming from a distribution with $m$ bits of entropy.
If $m \gg (n+1)\log q$, then because the output of this function is so much smaller than $m$, we expect it to be completely uniform.




> # { .pause }
The proof of [LWEENCthm](){.ref} is quite subtle and requires some re-reading and  thought.
To read more about this, you can look at the survey of Oded Regev, ["On the Learning with Error Problem"](http://www.cims.nyu.edu/~regev/papers/lwesurvey.pdf) Sections 3 and 4.

## But what are lattices?

You can think of a lattice as a discrete version of a subspace.
A lattice $L$ is simply a discrete subset of $\mathbb{R}^n$ such that if $u,v\in L$ and $a,b$ are integers then $au+bv\in L$.^[By discrete we mean that points in $L$ are isolated. One formal way to define it is that there is some $\epsilon>0$ such that every distinct $u,v \in L$ are of distance at least $\epsilon$ from one another.]
A lattice is given by a basis  which simply a matrix $B$ such that every vector $u\in L$ is obtained as $u=Bx$ for some vector of integers $x$.
It can be shown that we can  assume without loss of generality that $B$ is full dimensional and hence it's an $n$ by $n$ invertible matrix.
Note that given a basis $B$ we can generate vectors in $L$, as well as test whether a vector $v$ is in $L$ by testing if $B^{-1}v$ is an integer vector.
There can be many different bases for the same lattice, and some of them are easier to work with than others (see [latticebasesfig](){.ref}).

![A _lattice_ is a discrete subspace $L \subseteq \R^n$ that is closed under _integer_ combinations. A _basis_ for the lattice is a minimal set $b_1,\ldots,b_m$ (typically $m=n$) such that every $u \in L$ is an integer combination of $b_1,\ldots,b_m$. The same lattice can have different bases. In this figure the lattice is a set of points in $\R^2$, and the black vectors $v_1,v_2$ and the ref vectors $u_1,u_2$ are two alternative bases for it. Generally we consider the basis $u_1,u_2$ "better" since the vectors are shorter and it is less "skewed".](../figure/Lattice-reduction.png){#latticebasesfig .class width=300px height=300px}

Some classical computational questions on lattices are:

* _Shortest vector problem:_ Given a basis $B$ for $L$, find the nonzero vector $v$ with smallest norm in $L$.

* _Closest vector problem:_ Given a basis $B$ for $L$ and a vector $u$ that is _not_ in $L$, find the closest vector to $u$ in $L$.

* _Bounded distance decoding:_ Given a basis $B$ for $L$ and a vector $u$ of the form $u=v+e$ where $v$ is in $L$, and $e$ is a particularly short "error" vector (so in particular no other vector in the lattice is within distance $\|e\|$ to $u$), recover $v$. Note that this is a special case of the closest vector problem.

In particular, if $V$ is a linear subspace of $\Z_q^n$, we can think of it also as a lattice $\hat{V}$ of $\mathbb{R}^n$ where we simply say that
that a vector $\hat{u}$ is in $\hat{V}$ if all of $\hat{u}$'s coordinates are integers and if we let $u_i = \hat{u}_i \pmod{q}$ then $u\in V$.
The learning with error task of recovering $x$ from $Ax+e$ can then be thought of as an instance of the  bounded distance decoding problem for $\hat{V}$.

A natural algorithm to try to solve the _closest vector_ and _bounded distance decoding_ problems is that to take the vector $u$, express it in the basis $B$ by computing $w = B^{-1}u$, and then round all the coordinates of $w$ to obtain an integer vector $\tilde{w}$ and let $v=B\tilde{w}$ be a vector in the lattice.
If we have an extremely good basis $L$ for the lattice then $v$ may indeed be the closest vector in the lattice,  but in other more "skewed" bases it can be extremely far from it.


## Ring based lattices

One of the biggest issues with lattice based cryptosystem is the key size. In particular, the scheme above uses an $m\times n$ matrix where each entry takes $\log q$ bits to describe.
(It also encrypts a single bit using a whole vector, but more efficient "multi-bit" variants are known.)
Schemes using _ideal lattices_ are an attempt to get more practical variants. These have very similar structure except that the matrix $A$ chosen is not completely random but rather can be described by a single vector.
One common variant is the following: we fix some polynomial $p$ over $\Z_q$ with degree $n$ and then treat vectors in $\Z_q^n$ as the coefficients of $n-1$ degree polynomials and always work modulo this polynomial $p()$.
(By this I mean that for every polynomial $t$ of degree at least $n$ we write $t$ as $ps+r$ where $p$ is the polynomial above, $s$ is some polynomial and $r$ is the "remainder" polynomial of degree $<n$;
then $t \pmod{p} = r$.)
Now for every fixed polynomial $t$, the operation $A_t$ which is defined as  $s \mapsto ts \pmod{p}$ is a linear operation mapping polynomials of degree at most $n-1$ to polynomials of degree at most $n-1$, or put another way, it is a linear map over $\Z_q^n$.
However, the map $A_d$ can be described using the $n$ coefficients of $t$ as opposed to the $n^2$ description of a matrix.
It also turns out that by using the Fast Fourier Transform we can evaluate this operation in roughly $n$ steps as opposed to $n^2$.
The ideal lattice based cryptosystem use matrices of this form to save on key size and computation time.
It is still unclear if this structure can be used for attacks; recent papers attacking principal ideal lattices have shown that one needs to be careful about this.

One ideal-lattice based system is the ["New Hope" cryptosystem](https://newhopecrypto.org/) (see also [paper](https://eprint.iacr.org/2015/1092.pdf))  that has been experimented with by Google.
People have also made highly optimized general (non ideal) lattice based constructions, see in particular the ["Frodo" system](https://frodokem.org/)   (paper [here](https://eprint.iacr.org/2016/659), can you guess what's behind the name?).
Both New Hope and Frodo have been submitted to the [NIST competition](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography) to select a "post quantum" public key encryption standard.
