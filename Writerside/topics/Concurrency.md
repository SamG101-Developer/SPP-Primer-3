# Concurrency

<primary-label ref="header-label"/>

<secondary-label ref="doc-complete"/>

S++ uses coroutines as the primary concurrency mechanism. They can be suspended and resumed, and support bidirectional
data flow between the caller and the coroutine. They are defined in the same way as a function or method, but they
require the `cor` keyword rather than `fun`.

Coroutine return types are constrained to generators, as explored below. Typically, there will be `gen` expressions
inside the coroutine, to generate or yield values to the caller. Assigning a value from the `gen` expression allows data
back into the function on resuming.

## Coroutine Return Types

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

There are 3 generator types in S++:

- `GenMov[Gen, Send=Void]`: Moves the value out of the generator, and into the caller.
- `GenMut[Gen, Send=Void]`: Mutably borrows the value from the generator, and into the caller.
- `GenRef[Gen, Send=Void]`: Immutably borrows the value from the generator, and into the caller.

{columns="3"}

Each generator type corresponds to the convention on the `gen` expression. For example, if values are generated with
`gen &mut value`, then the `GenMut` type is used as the return value. All values being generated must use the same
convention, and generate the same type.

The `Gen` generic parameter represents the type being generated. The return type `GenMov[Gen=BigInt]` means that big
integers are being generated and moved: `gen 123`. This means that both the expression type and expression convention
are represented in the return type.

The `Send` generic parameter represents the type being sent back to the coroutine. This defaults to `Void`, disallowing
data to be sent back (cannot have a `Void` variable), but can be set to any type. Only owned objects can be sent back
into a coroutine.

## Advancing a Coroutine

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-not-impl-yet"/>

A coroutine in advanced by using the `.step()` method on the coroutine. As this requires compiler specific code to
invalidate yielded borrows, `step` is a callable postfix keyword, rather than a method.

In iterative loops, a `condition.step()` call is inserted into the end of the loop, so the generator automatically
advances to the step value. Manual `.step()` calls can be used in normal expression contexts.

```
cor coroutine(a: BigInt, b: BigInt, c: BigInt) -> GenRef[Gen=BigInt] {
    gen &a
    gen &b
    gen &c
}

fun main() -> Void {
    let generator = coroutine(1, 2, 3)
    let a = generator.step()
    let b = generator.step()  # invalidates "a"
    let c = generator.step()  # invalidates "b"
}
```

Invalidating is done by assignment or variable definition statements marking their left-hand-side symbolic value as
borrow-bound to the generator rhs (check the postfix expression type for the `.step` postfix operation), and the
generator is subsequently advanced.

## Passing Data Out of a Coroutine

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

As seen above, data is passed out of a coroutine using the `gen` expression, including an optional convention. Allowing
borrows to be yielded from coroutines is the basis for iteration, as it allows elements of a vector, for example, to be
borrowed and used in the caller.

If the `Gen` argument is `Void`, then the generator type must be `GenMov[Gen=Void]`, because borrows cannot be taken
from nothing, ie `gen` is valid on its own, but `gen &mut` doesn't make sense without a value to borrow.

```
cor coroutine(a: BigInt, b: BigInt, c: BigInt) -> GenRef[Gen=BigInt] {
    gen &a
    gen &b
    gen &c
}
```

### Invalidating Borrows

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

When borrowed values are yielded from a coroutine, they are only valid until the next `step()` call is made to the
coroutine. This is to ensure they remain valid for the coroutine to use in steps up to the next yield.

```
cor coroutine(a: BigInt, b: BigInt, c: BigInt) -> GenRef[Gen=BigInt] {
    gen &a
    
    let s = a + b
    gen &s
}

fun main() -> Void {
    let generator = coroutine(1, 2, 3)
    let a = generator.step()
    let b = generator.step()  # invalidates "a"
    let c = generator.step()  # invalidates "b"
}
```

In this example it can be seen that `a` is consumed in the coroutine. As there is no guarantee whether a variable is
consumed or not in the coroutine after being yielded as a borrow, it must be assumed that a worst-cast scenario occurs,
and that every variable yielded as a borrow might subsequently be consumed. As such, every time another borrowed value
is yielded, the previous borrow will be invalidated in the caller.

## Passing Data Into a Coroutine

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

The `.step()` method takes a single argument, whose type matches the `Send` generic parameter of the generator (
coroutine return type). Because substituting `Void` as a generic parameter causes function parameters of that type to be
removed from the signature, `.step()` can be used for th default `Send=Void` generic parameter.

Values are received by placing the `gen` expression on the right-hand-side of a variable definition statement. Variables
are always moved into a coroutine, not borrowed. This means that receiving a second value into the coroutine doesn't
invalidate the first one.

```
cor coroutine() -> GenMov[Gen=BigInt, Send=BigInt] {
    let a = gen 1
    let b = gen 2
    let c = gen 3
    gen a + b + c
}

fun main() -> Void {
    let generator = coroutine()
    let a = generator.step(1)
    let b = generator.step(2)
    let c = generator.step(3)
    let t = generator.step(0)
}
```

## Borrowing Data Into a Coroutine

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-impl"/>

A problem is presented when trying to use borrows with coroutines; a borrow could be passed into a coroutine, the
coroutine suspends, the owned object in the caller context is consumed, and the borrow is subsequently used in the
coroutine. In order to prevent this, memory pinning is used.

All borrows into a coroutine must be pinned. This prevents them from being consumed in the caller context, until they
are manually released. Releasing a pin will invalidate any object relying on the pin. Therefore, if a coroutine received
a borrow, and the memory of that borrow is released, the coroutine is marked as consumed and is non-usable.

## Chaining Coroutines

<secondary-label ref="doc-sect-complete"/>

<secondary-label ref="feature-wip"/>

Coroutine chaining allows a coroutine to yield the entirety of another coroutine in a one-line expression; without a
loop. This is identical to Pythons `yield from` statement. In S++, `gen with` is used, to generate values with another
coroutine.

The following example shows equivalent code using a loop and coroutine chaining:

```
cor coroutine() -> GenMov[Gen=BigInt] {
    gen 1
    gen 2
    gen 3
}

cor coroutine_chain() -> GenMov[Gen=BigInt] {
    gen 0
    for i in coroutine() {
        gen i
    }
}
```

```
cor coroutine() -> GenMov[Gen=BigInt] {
    gen 1
    gen 2
    gen 3
}

cor coroutine_chain() -> GenMov[Gen=BigInt] {
    gen 0
    gen with coroutine()
}
```
