---
title: "Digital Goop"
date: 2019-02-05T13:46:34Z
draft: false
---
[Conflated Automatons](https://conflatedautomatons.wordpress.com/) discusses [digital reification](https://conflatedautomatons.wordpress.com/2019/01/08/heaps-of-slime/), with particular reference to the [sorites paradox](https://plato.stanford.edu/entries/sorites-paradox/):

> Most of the users of our dairy software will not be on quaint farms in the English countryside owning one cow named Britney, so it will be necessary to represent a herd. How many cows do you need to qualify as a herd? Well, in practice, a programmer will pick a useful bucket data structure, like a set or a list, and name that variable “herd”. Nowadays it would probably be a collection in a standard library, like java.util.HashSet. The concept of an empty collection is a familiar one to programmers, furthermore there is a specific object to point to called “herd” (the new variable), so a herd is defined to be a data structure with zero or more (whole) cows. Sorites paradox solved _<dusts hands>_. And unwittingly too.

In actual fact, it's not unheard-of (sorry...) to have a data structure that looks a bit like this:

{{< highlight kotlin >}}
sealed class Herd {

    companion object {
        fun of(cows: List<Cow>) = when {
            cows.isEmpty() -> Empty
            cows.distinct().size == 1 -> Singleton(cows.first())
            else -> Proper(cows.toSet())
        }
    }

    abstract val members: Set<Cow>
    abstract operator fun plus(other: Herd): Herd

    private object Empty : Herd() {
        override val members: Set<Cow> get() = emptySet()
        override operator fun plus(other: Herd): Herd = other
    }

    private data class Singleton(val member: Cow) : Herd() {
        override val members: Set<Cow> get() = setOf(member)
        override operator fun plus(other: Herd): Herd = when(other) {
            is Empty -> this
            is Singleton -> Proper(setOf(member, other.member))
            is Proper -> Proper(other.members + member)
        }
    }

    private data class Proper(override val members: Set<Cow>) : Herd() {
        override operator fun plus(other: Herd): Herd = when(other) {
            is Empty -> this
            else -> Proper(this.members + other.members)
        }
    }
}
{{< /highlight >}}

What this ([Kotlin](https://kotlinlang.org/), in case you're wondering) code says is that there are three ways of being a herd of cows: you can be _the empty herd_, a _singleton_ herd containing precisely one cow, or a _proper herd_ containing two or more cows. This code also defines a simple algebra (a monoid over herds) for merging herds together: the sum of an empty herd and any other herd is just the other herd, the sum of a singleton herd and any non-empty herd is a proper herd, and so on.

Why might someone do this? In cases where herds are often empty, or contain only a single cow, it may be more memory-efficient to have a simpler representation for those cases --- there are overheads involved in managing a `Set` that we can do without. (In reality, the internal representation of `Set` is already organised in a similar way, so there's probably not much in it). But notice that we have re-introduced the sorites paradox by the back-door: the distinction between a _proper_ herd and the _degenerate cases_ represented by the empty and singleton herds is based on a seemingly-arbitrary numeric threshold.

Why not have a _pair_ representation, for when there are just two cows? One reason might be that the algebra for merging herds starts to become more complex, the more cases we have to take into consideration: we want to avoid unnecessarily proliferating ramifications. The trade-offs we choose to make will likely be influenced by the distribution of herd-sizes in the practical domain we are trying to model: if herds of fewer than five cows are extremely rare, then it's probably not worth modelling singleton and empty herds at all.

The point I'm trying to make here is that _reification_ in such cases is not a matter of spiriting away the fuzzily empirical particularity of the domain into a crisp, frictionlessly powerful, abstract notion. It is rather the construction of a _machine_ that mediates between domains: a "thingifier" which adapts one sort of thingliness for the purposes of another. This adaptation is what CA, following Michael Weisberg, calls a "construal". It is simultaneously a theory about the world, and a theory about the means at hand for dealing with the world. A programmer who implements `Herd` as an abstract data type like the above is essaying a proposal about the way the machine will manage its own representations, in the same breath as a proposal about how herds themselves are numbered. They are also creating affordances and constraints for other programmers, suggestions or directions to understand the matter in the specified terms, and approach it with the provided tools and techniques.

---

I picture a certain sort of reader taking issue with CA's Wimsattian characterisation of the _un-reified_ regions of our ontologies as "slimy". Sliminess is the raw-and-wriggly thingliness of things in its undetermined, unramified, non-relational form. Per Irigaray (notoriously, in "The Mechanics of Fluids"), it's gendered female. Doesn't this account reinforce a certain metaphorics of code as masculine fixity and object-orientation, individuating its way out of the maternal goop through a series of violently deproblematising gestures?

Well, I guess. But let's imagine that it's 1960-something, and programming is still generally understood as predominantly women's work. A major player in the invention of the _abstract data type_, let's not forget, was a woman, [Barbara Liskov](https://franklinchen.com/blog/2011/11/10/seeing-the-inventor-of-the-abstract-data-type). Does knowing that shift our metaphorics in any way? Is there a feminine-coded relationship to _slime_ that isn't one of wallowing, absorption, boundarilessness and so on?

(I mean, other than _cleaning and tidying up_, although there is a lot to be said for seeing programming in terms of the characteristic operations of household organisation and maintenance. Does this subroutine spark joy? Programmers don't talk so much about "routines" and "subroutines" any more, as it happens --- a metaphoric shift that itself bears reflecting on).

The way forward may be to see slime itself as already code-bearing, rather as one imagines fragments of RNA floating and combining in a _primordial soup_. Suppose we think of programming as refining slime, making code out of its codes, sifting and synthesizing. Like making bread from sticky dough, or throwing a pot out of wet clay. I'm not suggesting that these are _better_ metaphors per se, but it might be interesting to add them to our repertoire.
