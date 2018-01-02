% Lecture 18: Multiparty secure computation: Construction using Fully Homomorphic Encryption
% Boaz Barak


\newcommand{\zo}{\{0,1\}}
\newcommand{\E}{\mathbb{E}}
\newcommand{\Z}{\mathbb{Z}}
\newcommand{\R}{\mathbb{R}}
\newcommand{\getsr}{\leftarrow_R\;}
\newcommand{\Gp}{\mathbb{G}}
\newcommand{\Hp}{\mathbb{H}}

\newcommand{\floor}[1]{\lfloor #1 \rfloor}
\newcommand{\ceil}[1]{\lceil #1 \rceil}

\newcommand{\iprod}[1]{\langle #1 \rangle}

\newcommand{\cF}{\mathcal{F}}

\newcommand{\onand}{\overline{\wedge}}

In the last lecture we saw the definition of secure multiparty computation, as well as the compiler reducing the task of achieving security in the general (malicious) setting to the passive (honest-but-curious) setting.
In this lecture we will see how using fully homomorphic encryption we can achieve security in the honest-but-curious setting.
We focus on the two party case, and so prove the following theorem:

__Theorem:__ Under LWE, for every two party functionality $F$ there is a protocol computing $F$ in the honest but curious model.

Before proving the theorem it might be worthwhile to recall what is actually the definition of secure multipary computation, when specialized for the $k=2$ and honest but curious case.
The definition significantly simplifies here since we don't have to deal with the possibility of aborts.

__Definition (2-party honest but curious computation):__ Let $F$ be (possibly probabilistic) map of $\zo^n\times \zo^n$ to $\zo^n\times\zo^n$.    A _secure protocol for $F$_ is a two party protocol such for every party $t\in \{1,2\}$, there exists an efficient "ideal adversary" (i.e., efficient interactive algorithm)  $S$ such that  for every pair of inputs $(x_1,x_2)$ the following two distributions are computationally indistinguishable:

* The tuple $(y_1,y_2,v)$ obtained by running the protocol on inputs $x_1,x_2$, and letting $y_1,y_2$ be the outputs of the two parties and $v$ be the _view_ (all internal randomness, inputs, and messages received) of party $t$.

* The tuple $(y_1,y_2,v)$ that is computed by letting $(y_1,y_2)=F(x_1,x_2)$ and $v=S(x_t,y_t)$.


That is, $S$, which only gets the input $x_t$ and output $y_t$, can simulate all the information that an  honest-but-curious adversary controlling party $t$ will view.


## Constructing 2 party honest but curious computation from fully homomorphic encryption

Let $F$ be a two party functionality.  Lets start with the case that $F$ is _deterministic_ and that only Alice receives an output.
We'll later show an easy reduction from the general case to this one.
Here is a suggested protocol for Alice and Bob to run on inputs $x,y$ respectively so that Alice will learn $F(x,y)$ but nothing more about $y$, and Bob will learn nothing about $x$ that he didn't know before.

__Protocol 2MPC:__

* __Assumptions:__ $(G,E,D,EVAL)$ is a fully homomorphic encryption scheme.

* __Inputs:__ Alice's input is $x\in\zo^n$ and Bob's input is $y\in\zo^n$. The goal is for Alice to learn only $F(x,y)$ and Bob learn nothing.

* __Alice->Bob:__ Alice generates $(e,d)\getsr G(1^n)$ and sends $e$ and $c=E_e(x)$.

* __Bob->Alice:__ Bob computes define $f$ to be the function $f(x)=F(x,y)$ and sends $c'=EVAL(f,c)$ to Alice.

* __Alice's output:__ Alice computes $z=D_d(c')$.

First,  note that if Alice and Bob both follow the protocol, then indeed at the end of the protocol Alice will compute $F(x,y)$.
We now claim that Bob does not learn anything about Alice's input:

__Claim B:__ For every $x,y$, there exists a standalone algorithm $S$ such that $S(y)$ is indistinguishable from Bob's view when interacting with Alice and their corresponding inputs are $(x,y)$.

__Proof:__ Bob only receives a single message in this protocol of the form $(e,c)$ where $e$ is a public key and $c=E_e(x)$.
The simulator $S$ will generate $(e,d) \getsr G(1^n)$ and compute $(e,c)$ where $c=E_e(0^n)$. (As usual $0^n$ denotes the length $n$ string consisting of all zeroes.)
No matter what $x$ is, the output of $S$ is indistinguishable from the message Bob receives by the security of the encryption scheme. QED

(In fact, this protocol is secure even against a _malicious_ strategy of Bob- can you see why?)

We would now hope that we can prove the same regarding Alice's security. That is prove the following:

__Claim A:__ For every $x,y$, there exists a standalone algorithm $S$ such that $S(y)$ is indistinguishable from Alice's view when interacting with Bob and their corresponding inputs are $(x,y)$.

At this point, you might want to try to see if you can prove Claim A on your own.
If you're having difficulties proving it, try to think whether it's even true.

.

.

.

.

.

.

.

.

.

.


.


So, it turns out that Claim A is _not_ generically true. The reason is the following: the definition of fully homomorphic encryption only requires that $EVAL(f,E(x))$ decrypts to $f(x)$ but it does _not_ require that it hides the contents of $f$. For example, for every FHE, if we modify $EVAL(f,c)$ to output the first $100$ bits of the description of $f$ then this would still be a secure FHE.[^length]
Now we didn't exactly specify how we describe the function $f(x)$ defined as $x \mapsto F(x,y)$ but there are clearly representations in which the first $100$ bits of the description would reveal the first few bits of the hardwired constant $y$, hence meaning that Alice will learn those bits from Bob's message.

[^length]: It's true that strictly speaking, we allowed $EVAL$'s output to have length at most $n$, while this would make the output be $n+100$, but this is just a technicality that can be easily bypassed, for example by having a new scheme that on security parameter $n$ runs the original scheme with parameter $n/2$ (and hence will have a lot of "room" to pad the output of $EVAL$ with extra bits).

Thus we need to get a stronger property, known as _circuit privacy_: this is a property that's useful elsewhere too for FHE. Let us now define it:

__Definition:__ Let $\mathcal{E}=(G,E,D,EVAL)$ be an FHE. We say that $\mathcal{E}$ satisfies _perfect circuit privacy_[^identical] if for every $(e,d)$ output by $G(1^n)$ and every  function $f:\zo^\ell\rightarrow\zo$ of $poly(n)$ description size,
and every ciphertexts $c_1,\ldots,c_\ell$ and $x_1,\ldots,x_\ell \in \zo$ such that $c_i$ is output by $E_e(x_i)$, the distribution of $EVAL_e(f,c_1,\ldots,c_\ell)$ is identical to the distribution of $E_e(f(x))$.
That is, for every $z\in\zo^*$, the probability that $EVAL_e(f,c_1,\ldots,c_\ell)=z$ is the same as the probability that $E_e(f(x))=z$. We stress that these probabilities are taken only over the coins of the algorithms $EVAL$ and $E$.

[^identical]: This is the same as what we called "the identical ciphertexts property" in the homework assignment.

Perfect circuit privacy is a strong property, that also automatically implies that $D_d(EVAL(f,E_e(x_1),\ldots,E_e(x_\ell)))=f(x)$ (can you see why?).
In particular, once you understand the definition, the following claim  is an easy exercise (and so one that is good for you to do to make sure you understood it):

__Claim (circuit privacy claim):__ If $(G,E,D,EVAL)$ satisfies perfect circuit privacy then if $(e,d) = G(1^n)$ then for every two functions $f,f':\zo^\ell\rightarrow\zo$ of $poly(n)$ description size and every $x\in\zo^\ell$ such that $f(x)=f'(x)$, and every algorithm $A$,
$| \Pr[ A(d,EVAL(f,E_e(x_1),\ldots,E_e(x_\ell)))=1] -  \Pr[ A(d,EVAL(f',E_e(x_1),\ldots,E_e(x_\ell)))=1] | < negl(n)$.

(Note that the algorithm $A$ above gets the _secret key_ as input, but still cannot distinguish whether the $EVAL$ algorithm used $f$ or $f'$.)
In fact, the expression above is even equal to zero, though for our applications bounding it by a negligible function is enough.
Indeed, we are fine with using the relaxed notion of "imperfect" circuit privacy, defined as follows:

__Definition:__ Let $\mathcal{E}=(G,E,D,EVAL)$ be an FHE. We say that $\mathcal{E}$ satisfies _statistical circuit privacy_ if for every $(e,d)$ output by $G(1^n)$ and every  function $f:\zo^\ell\rightarrow\zo$ of $poly(n)$ description size,
and every ciphertexts $c_1,\ldots,c_\ell$ and $x_1,\ldots,x_\ell \in \zo$ such that $c_i$ is output by $E_e(x_i)$, the distribution of $EVAL_e(f,c_1,\ldots,c_\ell)$ is equal up to $negl(n)$ total variation distance to the distribution of $E_e(f(x))$.
This means that

$\sum_{z\in\zo^*} \left| \Pr[ EVAL_e(f,c_1,\ldots,c_\ell)=z] - \Pr[ E_e(f(x))=z ] \right| < negl(n)$

where once again, these probabilities are taken only over the coins of the algorithms $EVAL$ and $E$.

If you find  the above definition  hard to parse, the most important points you need to remember about it are the  following:

* Statistical circuit privacy is as good as perfect circuit privacy for all applications, and so you can imagine the latter notion when using it.

* Statistical circuit privacy can easier to achieve in constructions.

(The third point, which goes without saying, is that you can always ask clarifying questions in class, Piazza, sections, or office hours...)

Intuitively, circuit privacy corresponds to what we need in the above protocol to protect Bob's security and ensure that Alice doesn't get any information about his input that she shouldn't have from the output of $EVAL$, but before working this out, let us see how we can construct fully homomorphic encryption schemes satisfying this property.

## Achieving circuit privacy in a fully homomorphic encryption

We now discuss how we can modify our fully homomorphic encryption schemes to achieve the notion of circuit privacy.
In the scheme we saw, the encryption of a bit $b$, whether obtained through the encryption algorithm or $EVAL$, always had the form of a matrix $C$ over $\Z_q$ (for $q=2^{\sqrt{n}}$) where $Cv = bv + e$ for some vector $e$ that is "small" (e.g., for every $i$, $|e_i| < n^{polylog(n)}\ll q=2^{\sqrt{n}}$).
However, the $EVAL$ algorithm was _deterministic_ and hence this vector $e$ is a function of whatever function $f$ we are evaluating and someone that knows the secret key $v$ could recover $e$ and then obtain from it some information about $f$.
We want to make $EVAL$ probabilistic and lose that information, and we use the following approach

>_To kill a signal, drown it in lots of noise_

That is, if we manage to add some additional random noise $e'$ that has magnitude  much larger than $e$, then it would essentially "erase" any structure $e$ had. More formally, we will use the following claim:

__Claim:__ Let $a\in \Z_q$ and $T\in\mathbb{N}$ be such that $aT<q/2$. If we let $X$ be the distribution obtained by taking $x (\mod q)$ for an integer $x$ chosen at random in $[-T,+T]$ and
let $X'$ be the distribution obtained by taking  $a+x (\mod q)$ for $x$ chosen in the same way, then

$\sum_{y \in \Z_q} \left| \Pr[X=y] - \Pr[X'=y] \right| <|a|/T$

__Proof:__  This has a simple "proof by picture": consider the intervals $[-T,+T]$ and $[-T+a,+T+a]$ on the number line

```
------*---*----------------------*--------------------------*-----*----
     -T -T+a                     0                         +T    +T+a
```

Note that the symmetric difference of these two intervals is only about a $1/T$ fraction of their union.
More formally, $X$ is the uniform distribution over the $2T+1$ numbers in the interval $[-T,+T]$ while $X'$ is the uniform distribution over the shifted version of this interval $[-T+a,+T+a]$.
There are exactly $2|a|$ numbers which get probability zero under one of those distributions and probability $(2T+1)^{-1}<(2T)^{-1}$ under the other. QED

We will also use the following claim, which we leave verifying as an exercise:

__Claim:__ If two distributions over numbers $X$ and $X'$ satisfy $\Delta(X,X')=\sum_{y\in\Z}|\Pr[X=x]-\Pr[Y=y]|<\delta$ then the distributions $X^m$ and $X'^m$ over $m$ dimensional vectors where every entry is sampled independently from $X$ or $X'$ respectively satisfy $\Delta(X^m,X'^m) \leq m\delta$.

(We will actually only use this claim for the distributions above; you can obtain intuition for it by considering the $m=2$ case where we compare the rectangles of the forms $[-T,+T]\times [-T,+T]$ and $[-T+a,+T+a]\times[-T+b,+T+b]$. You can see that their union has size roughly $4T^2$ while their symmetric difference has size roughly $2T\cdot 2a + 2T\cdot 2b$, and so if $|a|,|b| \leq \delta T$ then the symmetric difference is roughly a $2\delta$ fraction of the union.)

We will not provide the full details, but together these claims show that  $EVAL$ can use bootstrapping to reduce the magnitude of the noise to roughly $2^{n^{0.1}}$ and then add an additional random noise of roughly, say, $2^{n^{0.2}}$ which would make it statistically indistinguishable from the actual encryption.
(Here are some hints on the full details: the idea is that in order to "re-randomize" a ciphertext $C$ we need a very noisy encryption of zero and add it to $C$. The normal encryption will use noise of magnitude $2^{n^{0.2}}$ but we will provide an encryption of the secret key with smaller magnitude $2^{n^{0.1}/polylog(n)}$ so we can use bootstrapping to reduce the noise. The main idea that allows to add noise is that at the end of the day, our scheme boils down to LWE instances that have the form $(c,\sigma)$ where $c$ is a random vector in $\Z_q^{n-1}$ and $\sigma = \iprod{c,s}+a$ where $a \in [-\eta,+\eta]$ is a small noise addition. If we take any such input and add to $\sigma$ some $a' \in [-\eta',+\eta']$ then we create the effect of completely re-randomizing the noise. However, completely analyzing this requires non-trivial amount of care and work.)

<!--

__Claim:__ Let $q=2^{\sqrt{m}}$ and suppose that $\eta< 2^{n^{0.1}}$ and $e\in\Z_q^m$ is some vector with $|e_i| \leq \eta $ for all $i$, and let $\eta' = 2^{n^{0.1}}\eta$.
Let $Z'$ be the distribution over vectors in $\Z_q^m$ where each coordinate is chosen at random in $\{ - \eta' , \eta'+1,\ldots, \eta'-1,\eta' \}$ and let
$Z''$ be the distribution over $\Z_q^m$ where we choose $e' \getsr Z'$ and output $e + e' (\mod \; q)$.
Then,

$\sum_{z\in\Z_q^m} \left| \Pr[ Z' = z] - \Pr[ Z'' = z ] \right| < negl(n)$

__Proof:__ For every $z\in\Z_q^m$, if $|z_i| \leg \eta'$ for all $i$ then $\Pr[ Z'=z] = \tfrac{1}{(2\eta'+1)^m}$ and otherwise $\Pr[Z'=z]=0$.
On the other hand, if $|z_i - e_i| \leq \eta'$ for all $i$ then we get $\Pr[ Z''=z] = \tfrac{1}{(2\eta'+1)^m}$ and otherwise $\Pr[Z''=z]=0$.
We see that for a vector $z$ to have a different probability under $Z'$ than it has under $Z''$ there must be at least one coordinate $i$ where $|z_i - e_i| \leq \eta'$ and $|z_i|>\eta'$ or vice versa, while all other coordinates must be of magnitude  at most $(\eta+\eta')$.
Since $|e_i| \leq \eta$ we get $2\eta$ choices for this coordinate and at most $2(\eta+\eta')$ choices for the others.
So we get that the number of vectors $z'$ where the probability under $Z'$ differs from the probability under $Z''$ is at most  $m$ (for the choice of coordinate) times

$2\eta \cdot (2(\eta+\eta')+1)^m =  2^{m+1} \eta^m (T+1)^m\eta$

where $T = \eta'/\eta = 2^{n^{0.1}}$.
Each one of those vectors gets probability $(2\eta'+1)^{-m} = (2T\eta+1)^{-m} \geq (2T\eta)^m$ and so the total probability difference is at most

$\frac{2^{m+1}\eta^m \cdot (T+1)^m}{\eta^m\cdot 2^m T^m }

-->
