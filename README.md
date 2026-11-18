# Trim Specific Characters

- Stage: **0**
- Champion: [Eemeli Aro](https://github.com/eemeli)

When processing String values from user input or otherwise,
needing to remove (i.e. trim) the String of its outer whitespace or other characters is a very common need.
JavaScript supports this with the
[String.prototype.trim](https://tc39.es/ecma262/#sec-string.prototype.trim),
[.trimStart](https://tc39.es/ecma262/#sec-string.prototype.trimstart), and
[.trimEnd](https://tc39.es/ecma262/#sec-string.prototype.trimend) functions.

However, currently String trimming is restricted to only trimming white space,
which is defined as:

> The definition of white space is the union of _WhiteSpace_ and _LineTerminator_.
> When determining whether a Unicode code point is in Unicode general category “Space_Separator” (“Zs”),
> code unit sequences are interpreted as UTF-16 encoded code point sequences as specified in
> [6.1.4](https://tc39.es/ecma262/#sec-ecmascript-language-types-string-type).

This is somewhat problematic for at least two reasons:

1. The set of trimmed characters may change when the Unicode standard is updated.
   This is rare, but does happen; the most recent such change was made in Unicode 6.3 (2013),
   when MONGOLIAN VOWEL SEPARATOR U+180E was re-classified as a formatting character instead of a space character.

2. The set of trimmed characters may not match user expectations,
   in particular when parsing syntax that has a different definition for "white space".
   For example, [JSONWhiteSpace](https://tc39.es/ecma404/#prod-JSONWhiteSpace) only allows for
   `\r`, `\n`, `\t`, and space characters as white space.
   Using String.prototype.trim when parsing JSON would lead to treating additional characters as white space.

We should allow for the set of trimmed characters to be user-customizable.

This idea was previously briefly discussed in [proposal-string-left-right-trim#22](https://github.com/tc39/proposal-string-left-right-trim/issues/22),
but dismissed there as out-of-scope for that proposal.

## Customizable Trimming Elsewhere

At least the following languages' standard libraries allow for string trimming to be customized:

| Language    | Function                                                                                 | Argument type                                                                             |
| ----------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Common Lisp | [string-trim](https://novaspec.org/cl/f_string-trim)                                     | sequence of characters                                                                    |
| C#          | [String.Trim](https://learn.microsoft.com/en-us/dotnet/api/system.string.trim)           | character or array of characters                                                          |
| Go          | [strings.Trim](https://pkg.go.dev/strings#Trim)                                          | string of characters                                                                      |
| PHP         | [trim](https://www.php.net/manual/en/function.trim.php)                                  | string of characters                                                                      |
| Python      | [str.strip](https://docs.python.org/3/builtins/stdtypes.html#str.strip)                  | string of characters                                                                      |
| Rust        | [str.trim_matches](https://doc.rust-lang.org/std/primitive.str.html#method.trim_matches) | a char, a slice of chars, or a function or closure that determines if a character matches |

## Proposed Solution

Add an optional _characters_ argument to the
[String.prototype.trim](https://tc39.es/ecma262/#sec-string.prototype.trim),
[.trimStart](https://tc39.es/ecma262/#sec-string.prototype.trimstart), and
[.trimEnd](https://tc39.es/ecma262/#sec-string.prototype.trimend) functions.
If present, this would be required to be a String of characters
that would be trimmed from the start and/or end of the string:

```js
"\n\n foo".trim("\r\n"); // ' foo'
"bar:: ".trimEnd(" :"); // 'bar'

"bar:: ".trimEnd([" ", ":"]); // TypeError
```

## Alternatives

It is already now possible to trim strings using regular expressions,
but this is not very ergonomic. For example, compare:

```js
jsonValue.replace(/^[\r\n\t ]+|[\r\n\t ]+$/g, "");
jsonValue.trim("\r\n\t ");
```

The `.trim()` argument could accept an iterable of characters instead of a String,
which would allow for e.g. `[" ", ":"]` to be used as a value.
When iterating, we would then need to check that each value has a length of 1,
which seems like an unnecessary complication.

Rather than adding an argument to the existing functions,
we could add new String functions like
`.trimMatches()`,  `.trimStartMatches()`, and `.trimEndMatches()`
which would require the argument.
However, this seems quite unnecessary unless adding the argument turns out to be web-incompatible.