# Proposal for `a[^i]` syntax

A JavaScript proposal to add `a[^i]` syntax for `a[a.length - i]`

Stage: 0

Champions: HE Shi-Jun

Rationale
---------

For many years, programmers have asked for the ability to do "negative indexing" of JS Arrays, like you can do with Python. That is, asking for the ability to write `arr[-1]` instead of `arr[arr.length-1]`, where negative numbers count backwards from the last element.

Unfortunately, JS's language design makes this impossible. The `[]` syntax is not specific to Arrays and Strings; it applies to all objects. Referring to a value by index, like `arr[1]`, actually just refers to the property of the object with the key "1", which is something that any object can have. So `arr[-1]` already "works" in today's code, but it returns the value of the "-1" property of the object, rather than returning an index counting back from the end.

There have been many attempts to work around this; the most recent is a restricted proposal to make it easier to access just the last element of an array (<https://github.com/tc39/proposal-array-last>) via a `.last` property.

This proposal instead adopts a more common approach, and suggests adding a `arr[^N]` syntax (follow the prior art of C#), which just work as `arr[arr.length - N]`, with the semantics as described above.

### Existing Methods

Currently, to access a value from the end of an indexable object, the common practice is to write `arr[arr.length - N]`, where N is the Nth item from the end.  This requires naming the indexable twice, additionally adds 7 more characters for the `.length`, and is hostile to anonymous values; you can't use this technique to grab the last item of the return value of a function unless you first store it in a temp variable.

Another method that avoids some of those drawbacks, but has some performance drawbacks of its own, is `arr.slice(-N)[0]`. This avoids repeating the name, and thus is friendly to anonymous values as well. However, the spelling is a little weird, particularly the trailing `[0]` (since `.slice()` returns an Array). Also, a temporary array is created with all the contents of the source from the desired item to the end, only to be immediately thrown away after retrieving the first item.

Note, however, the fact that `.slice()` (and related methods like `.splice()`) already have the notion of negative indexes, and resolve them exactly as desired.


Examples
--------
```js
let a = [1, 2, 3, 4]
a[^1] // 4
++a[^1] // 5
a[^0] // a[4], undefined
a[^0] = 10
a // [1, 2, 3, 5, 10]
```

Possible Issues
---------------

`arr[^N]` is a pure syntax sugar (though we may extend `^N` as a first-class value in the future like C#), some TC39 delegates don't like syntax sugar, or setup a very high bar for any new syntax.

Transpiling
-----------

```js
// x = EXPR[^N]
// ->
x = ((a, n) => a[a.length - n])(EXPR, N)

// EXPR[^N] = VALUE
// ->
((a, n, v) => (a[a.length - n] = v))(EXPR, N, VALUE)
```

## Comparison of `arr[^N]` to `arr.at(-N)` proposal

### Web compatibility

`arr[^N]` do not have web compat issue.

`.item()` (the old version of `.at()`) has been proved suffer from web compat issue, `.at()` might also be web incompatible, for example, Sugar.js provide `Array.prototype.at()` from 2010, and old `String.prototype.at()` polyfill with incompatible semantic.

### Generality

`arr[^N]` work for both read and write.

`.at()` only applies to read.

`arr[^N]` work for everything which is Array Like.

`.at()` only applies to Array, TypedArray, String.

### Ergonomics

Similar, though `arr[^N]` is 3 chars shorter than `arr.at(-N)`

### Learning, understanding and memory cost

`arr[^N]` could be think as pure syntax sugar, so every JS programmers could learn it in 1 minute.

`.at()` looks also easy, but in real programming there are much things to consider, need to RTFM:
- Which objects have the `at()` method? Does strings have it? Does DOM collections have it?
- Does `.at()` throw or return `undefined` for out of range index?
- Is `string.at()` codepoint safe?
- What `.at(-0)` means?
- etc.

### Adoption cost

Similar, `arr[^N]` need transpiling but no runtime polyfill; `.at()` need no transpiling but runtime polyfill.

### Relation to other proposals

`arr[^N]` could be extended to slice notation: `arr[0:^N]` (drop last N items), `arr[^N:^0]` (take last N items), which solve the block issue (syntax/semantic inconsitency of `a[-1]` with `a[0:-1]`) of slice notation.

### Edge case

Note, `^N` always means `arr.length - N`, so if `N` is 0, it means `arr.length`, as programmer expect. On the other side, `a.at(-N)` and `a.slice(-N)` have the edge case of `-0` which behave same as `0`, it's very likely not programmers expect and error-prone. This edge case is very common in `slice` usage, but may also affect `at`, for example code `if (arr.at(-N) !== undefined) ...`.

See the discussions of `-0` edge case in various places:

- https://github.com/rust-lang/rfcs/issues/2249#issuecomment-352128826
- https://stackoverflow.com/questions/39460528/in-string-prototype-slice-should-slice0-0-and-slice0-0-output-the-sam/39461147
- https://stackoverflow.com/questions/31740252/when-use-negative-number-to-slice-a-string-in-python-0-is-disabled?noredirect=1&lq=1
- https://bytes.com/topic/python/answers/504522-string-negative-indices
- https://tesarek.me/articles/slicing-primer#indexing
- https://open.cs.uwaterloo.ca/python-from-scratch/2/9/transcript
- https://news.ycombinator.com/item?id=13186225


## Prior arts

### C# 8 `^fromEnd` (*index from end* operator)
- https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/ranges#systemindex
- https://stackoverflow.com/questions/54092458/why-doesnt-the-new-hat-operator-index-from-the-c-sharp-8-array-slicing-feature/54092520
- https://www.reddit.com/r/csharp/comments/arknnk/c_8_introducing_index_struct_and_a_brand_new/

### D `a[$-1]`

- https://dlang.org/spec/arrays.html#array-length
- https://dlang.org/spec/operatoroverloading.html#dollar

### Raku (formerly Perl 6) drop Perl's `@a[-i]` and introduce `@a[*-i]` 
- https://docs.raku.org/language/traps#Referencing_the_last_element_of_an_array
- https://docs.raku.org/language/subscripts#From_the_end
- https://design.raku.org/S09.html#Negative_and_differential_subscripts
