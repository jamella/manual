
Precomputation {#sec:precomputation}
============== 

In this section, we will explain some of the aspects of the
precomputation performed by Tamarin.  This is relevant for users that
model complex protocols since they may at some point run into so-called
[open chains](#sec:openchains), which can be problematic for verification.

To illustrate the concepts, consider the example of the Needham-Schroeder-Lowe
Public Key Protocol, given here in Alice&Bob notation:

~~~~ {.tamarin slice="code/NSLPK3.spthy" lower=24 upper=29}
~~~~

It is specified in Tamarin by the following rules:

~~~~ {.tamarin slice="code/NSLPK3.spthy" lower=32 upper=71}
~~~~

We now want to prove the following lemma:

~~~~ {.tamarin slice="code/NSLPK3.spthy" lower=105 upper=118}
~~~~

This proof attempt will not terminate due to there being `12 chains
left` when looking at this example in the GUI as described in detail below.

Open Chains {#sec:openchains}
-----------

In the precomputation phase, Tamarin goes through all rules and inspects their
premises. For each of these facts, Tamarin will precompute a set of possible
*sources*. Each such source is called a *chain* and represents
combinations of rules from which the fact could be obtained.  For each fact,
this leads to a set of possible sources and we refer to these sets as the *case
distinctions*.

<!--**FIX: above notions are confusing: chain versus sources versus case distinctions.**-->

However, for some rules Tamarin cannot resolve where a fact must have come from.
We call such a chain an *open chain*, and we will explain them in more detail
below.

The existence of open chains complicates automated proof generation and
often (but not always) means that no proof will be found automatically.  For
this reason, it is useful for users to be able to find open chains and examine
if it is possible to remove them.

In the interactive mode you can find open chains as follows.  On the top left,
under "Untyped case distinction", one can find the chains that were precomputed
by Tamarin.

![Tamarin GUI](../images/FindOpenChains1.png "Untyped case distinctions"){ width=50% }\

The unsolved chains can be identified by the light green arrows as in the
following example:

![Open chain visible in green](../images/FindOpenChains2.png "Open chain visible"){ width=100% }\

The green arrow indicates that Tamarin cannot exclude the possibility that the
adversary can derive any fresh term `~t.1` with this rule `I_2`.  As we are
using an untyped protocol model, the tool cannot determine that `nr.7` should be
a fresh nonce, but that it could be any message. For this reason Tamarin
concludes that it can derive any message with this rule.

<!--**FIX Cas: In the above, we mention untyped protocol model. Did we explain
this?**-->

### Why open chains complicate proofs

To get a better understanding of the problem, consider  what happens if
we try to prove the lemma `nonce_secrecy`.  If we manually always choose
the first case for the proof, we can see that Tamarin derives the secret key to
decrypt the output of rule `I_2` by repeatedly using this rule `I_2`.
More specifically, in `a)` the output of rule `I_2` is decrypted by the 
adversary. To get the relevant key for this, in part `b)` again the output
from rule `I_2` is decrypted by the adversary. This is done with a key coming
from part `c)` where the same will happen repeatedly.

![Secret derived by using `I_2`](../images/FindOpenChains3_RepetitionHilighted.jpg "`I_2` repeatedly"){ width=90% }\

As Tamarin is unable to conclude that the secret key could not have come from
the rule `I_2`, the algorithm derives the secret key that is needed. The proof
uses the same strategy recursively but will not terminate.



Using Typing Lemmas to Mitigate Open Chains
-------------------------------------

Once we identified the rules and cases in which open chains occur, we
can try to avoid them. A good mechanism to get rid of open chains is the use of
so-called *typing lemmas*.

Typing lemmas are a special case of lemmas, and are applied
during Tamarin's pre-computation. Roughly, verification in Tamarin involves
the following steps:

  1. Tamarin first determines the possible sources of all premises. We call these the
     untyped case distinctions.

  2. Next, automatic proof mode is used to discharge any typing lemmas using induction.

  3. The typing lemmas are applied to the untyped case distinctions, yielding a
     new set of sources, which we call the typed case distinctions.

  4. Depending on the mode, the other (non-typing) lemmas are now considered
     manually or automatically using the typed case distinctions.

For full technical details, we refer the reader to [@meierthesis].

In our example, we can add the following lemma:

~~~~ {.tamarin slice="code/NSLPK3.spthy" lower=86 upper=102}
~~~~

This typing lemma is applied to the untyped case distinctions to compute the
typed case distinctions. All non-typing lemmas are proven with the resulting
typed case distinctions, while typing lemmas must be proved with
the untyped case distinctions.

This lemma relates the point of instantiation to the point of sending by either
the adversary or the communicating partner. In other words, it says that
whenever the responder receives the first nonce, either the nonce was known to
the adversary or the initiator sent the first message prior to that moment.
Similarly, the second part states that whenever the initiator receives the
second message, either the adversary knew the corresponding nonce or the
responder has sent the second message before.
Generally, in a protocol with open chains it is advisable to try if the problem
can be solved by a typing lemma that considers where a term could be coming
from.
As in the above example, one idea to do so is by stating that a used term 
must either have occured in a one of a list of rules before or it must have 
come from the adversary.

The above typing lemma can be automatically proven by Tamarin. With the typing 
lemma, Tamarin can then automatically prove the lemma `nonce_secrecy`.


Another possibility is that the open chains only occur in an undesired
application of a rule that we do not wish to consider in our model.
In such a case, we can explicitly exclude this application of the rule
with an axiom. But, we should ensure that the resulting model is the
one we want; so use this with care.



