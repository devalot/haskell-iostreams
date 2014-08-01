---
title: Simple Streaming with io-streams
author: Peter J. Jones \<pjones@devalot.com\>
...

# Simple Introduction to io-streams

The Basics of [io-streams][]:

  * `InputStream a`: read values of type `a`
  * `OutputStream a`: write values of type `a`
  * `Maybe a`: `Nothing` means EOF

<div class="notes">

Values are read from an `InputStream`.  You usually begin by creating
an `InputStream ByteString` (an input stream which produces a
`ByteString` each time it's read from) and transform it with a
function which produces the values you are interested in.  We'll see
one such function (`decodeUtf8`) which transforms an `InputStream
ByteString` into an `InputStream Text`.

An `OutputStream` is similar, except that it represents a stream you
can write values to.  Input streams and output streams can be
connected to one another as long as they share the same value type
(the `a` type from above).

When reading values from an `InputStream` with the `read` function, a
`Maybe a` value is returned.  `Nothing` signals the end of the stream.
In a similar fashion, values written to an `OutputStream` are wrapped
in a `Maybe` to signal downstream when no more values will be written.

</div>

# io-streams is great because...

  * It's really easy to learn and use
  * Stream processing is very fast
  * It's easy to compose stream functions together

# io-streams is less desirable because...

  * It makes heavy use of the `IO` type
  * Limited functionality compared to Conduit and Pipes
  * Error handling is done via exceptions

# Writing `wc` with Haskell and io-streams

The POSIX `wc` utility:

  * Processes zero or more files
  * When no files are given, processes STDIN
  * Counts bytes, characters, words, and lines
  * I've decided to omit bytes in my implementation

# Keeping Track of Lines, Words, and Characters

<div class="notes">

Let's start with some types.  First up is a type for tracking the
counters we'll report after processing all files.

</div>

~~~ {.haskell include="vendor/wc-streams/src/Main.hs" token="counters"}
~~~

<div class="notes">

The `Counters` type is a `Monoid` so that an initial counter can be
created (`mempty`) and a list of counters can be totaled (`mconcat`).

</div>

# Maintaining a Bit of State

<div class="notes">

We also need to maintain a little bit of state as we process files in
chunks.  Besides the previously shown counters, we also need to know
if the last character processed was part of a word.  This makes it
easy to count words when encountering whitespace, even when there are
several consecutive whitespace characters.

</div>

~~~ {.haskell include="vendor/wc-streams/src/Main.hs" token="state"}
~~~

# Counting Unicode Characters

<div class="notes">

The `wc` function creates a new `State` value based on the previous
state and one character from the input stream.

</div>

~~~ {.haskell include="vendor/wc-streams/src/Main.hs" token="wc"}
~~~

<div class="notes">

A newline character modifies the state more than other characters
because it should increment the number of characters, lines, and
possibly the number of words (a newline might terminate a word).

Space characters also increase the character count.  If the previous
character was part of a word, space characters also increment the
number of words (and set `inWord` to `False`).

All other characters increment the character count and update the
state so that `inWord` is `True`.

</div>

# Processing a Stream of Bytes

<div class="notes">

The `stream` function takes an `InputStream ByteString`, converts it
into an `InputStream Text`, and then process all characters in the
stream by continually reading from it until a `Nothing` is returned.

</div>

~~~ {.haskell include="vendor/wc-streams/src/Main.hs" token="stream"}
~~~

<div class="notes">

The `eof` function isn't shown here but plays an important role.
After consuming all input from a stream we might need to increment the
number of words if the last character in the stream was part of a
word.

</div>


# Putting it All Together

<div class="notes">

The only (important) thing left is the `main` function.  There are
basically two branches depending on how many files were listed on the
command line.  When no files are given, the `stream` function will be
used with standard input.  Otherwise the `stream` function will be
used with each of the files listed on the command line.

When more than one file is given to the `wc` utility, it will print
the counters for each file and then a grand total.

</div>

~~~ {.haskell include="vendor/wc-streams/src/Main.hs" token="main"}
~~~

# Getting the Code

<div class="notes">

The source code for the Haskell implementation of `wc` can be found at
the following URL:

</div>

<https://github.com/boulder-haskell-programmers/wc-streams>

<!-- Links -->
[io-streams]: https://hackage.haskell.org/package/io-streams
