# A guide to UniMaths

## Global Notations
`UU`

  Type.
  There is type in type in UniMaths, though it is known to be inconsistent.
  This allows to defined the squash `||A|| := forall (P:hProp), (A -> P) -> P` 

## Categorical notations

Notations are usually redefined at the top of each file (as local)

`C ⟦ a , b ⟧`

  Morphism between a and b in the category C

`a ;; b`

  Composition of morphisms

`# F a`

  Functor F applied to the morphism a

`G □ F`

  Composition of functors
  
## Tactics

`etrans`

  Transitivity of equality
  replace a goal `a=b` with two goals `a = ?x` and `?x = b`
  
`use`

  enhanced `apply` (`use (f _ _)` means `refine (f _ _)`)
  
`symmetry`

  DO NOT USE (it inserts a `match` in the proof term that does not fit UniMaths standard) ! instead use `eapply pathsinv0`

## Rewrite vs apply
`rewrite assoc` usually works well, but other rewrite don't work properly because of hidden coercions that may not be displayed by Coq (Coq is dumb when it comes to recognizes a rewrite pattern : no unfold etc...).

One would use `apply` in order to replace `rewrite`. It means that one must first isolate the subterm that should be rewritten using auxiliary lemmas.

The advantage of this is that when the code breaks, it easier to fix it. Indeed, the `rewrite` syntax does not reveal where it rewrites.

Example : 
Goal is `a ;; # F (b ;; c) ;; d = a ;; # F b ;; (# F c ;; d)`

Typically, rewrite `functor_on_morphisms` will not work, so we must first isolate the term
`# F (b ;; c)` and apply `functor_on_morphisms` on it.

```Coq
etrans.
apply cancel_postcomposition.
apply cancel_precomposition.
apply functor_on_morphisms.
```
The previous script replace the previous goal with `a ;; # F b ;; # F c ;; d = a ;; # F b ;; (# F c ;; d)`. Then `repeat rewrite assoc` does the job.

### Isolation lemmas

I put an underscore to mark what is eliminated using this lemma, and the interrogation mark is what remains

`cancel_postcomposition` : `? ;; _`

   What it means is suppose you have a goal `a ;; b = ?x` (because of a previous `etrans`). Then
   `apply cancel_postcomposition` will replace it with `a = ?y`

`cancel_precomposition` : `_ ;; ?`

`maponpaths` : `_ ?` (function application), `# _ ?` (functor on morphisms)

`toforallpaths` : `? _` 

Sometimes, the previous lemmas failed to apply because Coq does not succeed in guessing the right category. It can then be given explicitely : `apply (lemma (C:=precat))`


## Fun extensionality
`funext`

`funextsec` for the dependent version

## Equivalence of types
The notation `X ≃ Y` says that type X is equivalent to type Y.
Such a lemma can be directly as a term `X -> Y`. The lemma `invmap` allows to get the inverse map.
The following pattern is thus common: `use (invmap (weqb _ _  ...))`

## Precategories
A precategory is like a usual category. However, one often needs (for technical univalent reasons) that the homsets of a precategory are Sets in the univalent meaning (only one proof of equalities between two given morphisms). This property is named `has_homsets`, and the couple `(precategory, has_homsets)` is called a Precategory.

## Transports
Transports allows to cast a term from one type A to another B given a proof of equality `e` between A and B. It is implemented in Coq as a pattern matching on the proof of equality.

In UniMaths, one should use the `transportf` function : `transportf P e t` is of type `P B` when `t: P A` (`e:A=B`).

There is also the converse `transportb P e t := transportf P !e t` where `!e: B = A`.
The notation `!` denotes the proof of the reverse equality.

This extends to dependent types.

The trivial case is when `P` does not depend explicitely on its variable. In this case, `induction e` eliminates the transport (or the lemma `transport_const`)

Coq is not very helpful when it comes to working with transports.
