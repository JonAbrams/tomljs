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

## Some regex info

If you're new to regex is might help to know the following patterns that I make heavy use of:

- /aregex/gm - The /gm part enabled the _global_ flag and the _multi-line_ flag. The global flag tells the regex to make multiple matches, i.e. don't just quite the first time a match is found. The multi-line flag just changes the behaviour of _^_ and _$_ so that they act as _beginning of line_ and _end of line_ respectively.
- \s* - Look for 0 to many occurrences of whitespace

## FAQ

### Why does it allow arrays with mixed types?

Either because I think parsers should be forgiving in what they accept (and encoders strict in what they produce), or because I was too lazy to implement a type check.

### What's with all the words? I just care about the code.

If you don't like all the text, and just care about the code, look at the compiled JS or use js2coffee to convert it back to vanilla CoffeeScript. Maybe someone will make a litcoffee to coffee converter.

## License

MIT

## The code

To make the library available to JS code outside the browser we need to attach an appropriately named object to the global object, which in broswers in _window_.

    window.TOML = {}

Then we declare/attach the parse function that take a TOML multi-line string as input. It should be called just like JSON.parse() would. _obj_ is the object that will be returned. I couldn't come up with a better name that would be concise.

    TOML.parse = (text) ->
      obj = {}

The first thing to do is to trim leading & trailing whitespace, since it is never significant, and just gets in the way.

Think you can do better? [Read this first](http://blog.stevenlevithan.com/archives/faster-trim-javascript)

      text = text.replace(/^\s\s*/gm, "").replace(/\s\s*$/gm, "")

Now before moving on, we need to 'tokenize' strings. That means finding them and replacing them with variables (i.e. tokens). We need to do this or else we'll have trouble down the road. For example, when looking for comments we'll be looking for the _#_ character, and remove everything after it. But what happens if it's inside a string?

      strTokens = []

This big regex is looking for strings (i.e. strings of characters bounded by double-quotes) but allowing for escaped double-quotes. It's also not allowing newline characters.

Once a string is found, it is recorded and replaced with a token that will be swapped for the string later.

      text = text.replace /"(?:\\[^\n]|[^"\n])*"/g, (str) ->

Removed the surrounding double-quotes

        str = str.slice(0, str.length - 1).slice(1)

Replace any escaped characters with their 'actuals' counterparts.

        str = str
              .replace(/\\n/g, "\n")
              .replace(/\\t/g, "\t")
              .replace(/\\0/g, "\0")
              .replace(/\\r/g, "\r")
              .replace(/\\\\/g, "\\")
              .replace(/\\"/g, "\"")
        strTokens.push str
        "string#{strTokens.length - 1}"

Let's remove comments, which is the _#_ character plus the rest of the line.

      text = text.replace(/\s*#.*$/gm, "")

Now we can go through, line-by-line. Note how we keep track of the current _keygroup_

      keygroup = obj

      for line, lineNum in text.split("\n")
        if line is "" then continue

If a keygroup is found (recognized by the opening and closing square brackets). Add the key to the object by recursively examing the keys separated by _._'s.

        if line.match /^\[[_a-zA-Z](\w|\.)*]$/
          keygroup = obj

          addKey = (keys, parent) ->
            if keys.length is 0
              keygroup = parent
              return
            parent = obj unless parent
            key = keys.shift()
            parent[key] = {} unless parent[key]
            addKey(keys, parent[key])

          addKey line.replace(/^\[/, "").replace(/]$/, "").split("."), keygroup

We're now looking for assignments (i.e. key = value). First we test to make sure that it has a a key and a value on either side of a _=_. Since it's a big regex, we're using CoffeeScript's block regex feature.

        else if not line.match ///

Make sure the first character is an alpha or _.

          ^[_a-zA-z]

The rest of key can be a alphanumeric or _.

          \w*
Look for the _=_ sign, allowing for whitespace in front or behind it.

          \s*=\s*

Then a quick sanity check to make sure the value being assigned has at least a valid first character. In other words, the beginning of a string *"*, the beginning of an array *[*, a number, or a boolean.
          [string\d|\[\d]|true|false
        ///
          throw "Invalid statement on line #{lineNum}"

Split the line into a key and a value.We can't just use _split()_ because there may be an additional _=_ in the value. Also, do a trim on both the key and value.

        else
          equalsIndex = line.indexOf("=")
          key = line.slice(0, equalsIndex).replace(/\s*$/, "")
          value = line.slice(equalsIndex + 1).replace(/^\s*/, "")

Now that we have a valid key and a potentially valid value, let's see if it's an array, i.e. does it have an opening and closing square bracket?

          if value.match(isArray = /^\[.*]$/)

Since there can be arrays within arrays, let's do recursion again!

            processArray = (arrayStr, parent = null) ->

For each level, add values to the current array. Give up when only whitespace remains.

              while not arrayStr.match /^\s*$/

Remove leading & trailing whitespaces.

                arrayStr = arrayStr.replace(/^\s*/, "").replace(/\s*$/, "")

While looping through the array string, there are 3 possibilities:

- _[_ - Found a new array, and need to [recurse](http://en.wiktionary.org/wiki/recurse).
- _]_ - Reached the end of an array, return the current working array.
- A primitive value. Add it to the current working array.

                if arrayStr[0] is "["
                  if parent
                    newArray = []
                    arrayStr = arrayStr.slice(1)
                    [arrayStr, newArray] = processArray(arrayStr, newArray)
                    parent.push newArray
                  else # This is the first array encountered, so it will be the last to return
                    [arrayStr, topArray] = processArray(arrayStr.slice(1), [])
                    return topArray
                else if arrayStr[0] is "]"
                  throw "Could not parse array on line #{lineNum}. Check for extra ] characters." unless parent
                  return [arrayStr.replace(/]\s*,?\s*/, ""), parent]
                else

At this point we're probably looking at a primitive, get its value and add it to the array.

                  valueMatch = arrayStr.match /[^,\]]+/
                  throw "Could not parse array on line #{lineNum}. Check the number of opening and closing brackets." unless valueMatch
                  val = getVal(valueMatch[0].replace(/\s*$/, ""), strTokens)
                  unless val?
                    debugger
                    throw "Could not parse array on line #{lineNum}. Check for extra commas."
                  parent.push val

Remove the value from the array string, including any trailing comma.

                  arrayStr = arrayStr.replace(/[^,\]]+/, "")
                  arrayStr = arrayStr.slice(1) if arrayStr[0] is ","

              throw "Could not parse array on line #{lineNum}. Check for missing closing brackets."

At this point we should have a fully formed array, add it to the object.

            val = processArray(value)

          else

This assignment is not an array, so just convert.

            val = getVal value, strTokens # Look below for getVal implementation
            if val is null
              throw "Invalid value with assignment on line #{lineNum}"

The value is now ready to be assigned to its associated key.

          keygroup[key] = val
      obj



This function converts value strings into JavaScript primitives.

    getVal = (valStr, strTokens) ->

Is it a boolean?

      if valStr.match /^true|false$/
        return /true/.test valStr

Is it a number?

      else if valStr.match /^-?\d*\.?\d+$/
        return parseFloat(valStr.match(/^\d*\.?\d+$/)[0])

Is it a string? If so, look up its value in the string token array.

      else if valStr.match /string\d+/
        return strTokens[parseInt(valStr.match(/\d+/)[0], 10)]

Is it a Datetime? The easiest way to check is to create a new Date object, and see if it works.

      else if (date = new Date(valStr)).toString() isnt "Invalid Date"
        return date

At this point, we have an invalid value. We'll just return _null_ and let the caller throw an exception since it should know what line it's on.

      return null
