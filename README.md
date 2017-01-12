sp
==

Public domain ([CC0]) string formatting micro-library for C++, based on
python-style format strings.

Features:

* Safe. It's not possible to use the wrong format specifier for a given type.
  Data types are detected, not specified.
* Light-weight. The entire implementation is in one header, with no third party
  dependencies.
* More flexible formatting than printf, less verbose than iostreams.

**NOTE:** Please note that the contents of the `tests/catch` folder is *NOT*
released under CC0, but is a third party test framework. If you make copies of
these files, make sure you first refer to the license information in that same
folder.

Examples
--------

The header file has been split in two sections; one for public API
declarations, and one for the implementation. As such you may use the top of
the header file itself for a brief of the available API.

```cpp
// Print `Hello, World!\n` to stdout
sp::print("Hello, {}!\n", "World");
```

```cpp
char buffer[16];

// Format `+00000512` into a char buffer
sp::format(buffer, sizeof(buffer), "{:+08}", 512);

// Alternate way, since this is a fixed-size array
sp::format(buffer, "{:+08}", 512);
```

```cpp
// Format `name=John,height=1.80,employed=true` into a file
FILE* file = fopen("output.txt", "w");
sp::format(file, "name={2},height={0:.2f},employed={1}", 1.8019f, true, "John");
fclose(file);
```

Format string
-------------

As the format string is based on python's format, please refer to [python's
documentation on the string format][pyformat]. The differences between python's
format and this library's format are as such:

* Invalid replacement fields are written to the resulting string as-is, rather
  than having an exception be raised.

  * The format `{:` results in `{:` as output, as it is non-terminated.
  * The format `{:.}` results in `{:.}` as output, as it is missing the numbers
    specifying the precision.

* Certain flags may silently be ignored if they are used for a type that does
  not support it, rather than causing the format to be considered invalid.

  * `{:=}` when called with `foo` as the first argument results in `foo`,
    regardless of the fact that `=` is an unsupported alignment option for
    strings.

* `!conversions` are not supported.

  * `{0!s}` is an invalid format. As is `{0!a}`.

* Only numerical or omitted `field_name`s are supported.

  * `{} {2}` is a valid replacement field.
  * `{foo.bar}` is an invalid replacement field.
  * `{0[0]}` is an invalid replacement field.

* Mixing of indexed and non-indexed replacement fields is supported. When a
  non-indexed replacement field is encountered, its index is determined by
  incrementing the previous replacement field's index by `1`. If there is no
  previous replacement field, its index is instead `0`.

  * `{} {} {}` is the same as `{0} {1} {2}`.
  * `{} {} {1} {} {1}` is the same as `{0} {1} {1} {2} {1}`.

* Replacement fields may not be used within another replacement field.

  * `{0:{1}}` is an invalid replacement field.

* `grouping_option`s are not supported.

  * `{:_}` is an invalid replacement field. As is `{:,}`.

* Specifying `c` or `n` as `type` is unsupported.

  * `{:c}` is an invalid replacement field. As is `{:n}`.

* Omitting the `type` for floating point types causes it to fall back to `g`
  with a different default precision. The special case with getting at least
  one decimal for when the value ends up as fixed-point does not apply.

  * `{}` when called with `314159265.0` as the first argument results in
    `314159265`, rather than `314159265.0`.

Custom formatter
----------------

Support for formatting your own types may be achieved by overloading the
`format_value` function. The signature should be:

    bool format_value(sp::Output& output, const sp::StringView& format, T value);

Where `T` is your type. This function must be placed either in the `sp`
namespace, or the same namespace as the type it is formatting (`T` above).

### Arguments

<dl>
    <dt>output</dt>
    <dd>Output to write the resulting formatted string into.</dd>
    <dt>format</dt>
    <dd>The raw format specifier flags, as they appeared in the replacement
        field. If `{:foo}` was used to format your type, `"foo"` will be passed
        as this `format` argument. Please note that these format specifiers are
        not null-terminated.</dd>
    <dt>value</dt>
    <dd>The value to format. May be passed as `const T&` to avoid copying.</dd>
</dl>

[CC0]:      https://creativecommons.org/publicdomain/zero/1.0/              "CC0"
[pyformat]: https://docs.python.org/3/library/string.html#formatstrings     "Python 3 format string"