# TOML Parser for JavaScript
Version 0.9
Created by [Jon Abrams](https://twitter.com/JonathanAbrams)

## Introduction

This file provides functions for parsing input that is formatted using [TOML](https://github.com/mojombo/toml).

It is meant to be used in browsers. If you're looking for a TOML parser for node.js, [try this]().

I wrote this because I wanted to try out CoffeeScript's new "literate" mode, and I wanted to take a crack at writing a parser; luckily TOML just happened to appear.

I hope it's useful and/or educational. It should be understandable for beginner to intermediate JavaScripters. It will help you to know the following first:

- [CoffeeScript](http://coffeescript.org/)
- [regex](http://www.diveintojavascript.com/articles/javascript-regular-expressions)
- [recursion]().

## How to use the library

Include toml.js in your html file, then invoke TOML.parse(toml_input).

See the test.html file in this repo.

It's also available in a [live demo](http://tomljs.s3-website-us-east-1.amazonaws.com/).

## Some regex info

If you're new to regex is might help to know the following patterns that I make heavy use of:

- /aregex/gm - The /gm part enabled the _global_ flag and the _multi-line_ flag. The global flag tells the regex to make multiple matches, i.e. don't just quite the first time a match is found. The multi-line flag just changes the behaviour of _^_ and _$_ so that they act as _beginning of line_ and _end of line_ respectively.
- \s* - Look for 0 to many occurrences of whitespace

## FAQ

### Why does it allow arrays with mixed types?

Either because I think parsers should be forgiving in what they accept (and encoders strict in what they produce), or because I was too lazy to implement a type check.

### What's with all the words? I just care about the code.

If you don't like all the text, and just care about the code, look at the compiled JS or use js2coffee to convert it back to vanilla CoffeeScript. Maybe someone will make a litcoffee to coffee converter.

## Todo

- Tests
- Encoder
- Find more boundary conditions and weird errors

## License

MIT

## The code

GitHub does a pretty decent job of rendering literate CoffeeScript, so feel free to just [view it right here](https://github.com/JonAbrams/tomljs/blob/master/toml.litcoffee).
