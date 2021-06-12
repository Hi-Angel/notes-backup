A nice module that allows to search for quite complicated patterns in text, including nested structures *(balanced expressions)*.

Basically, pattern is created from token-classes *(such as Word, Char, etc)* in python code. There's Regex class too, so for example you could replace Word with `Regex('\w+')` if you want to. Then, once you got a pattern, you call a `pattern.searchString(mystring)` *(or `scanString`)* and you get matches in the text. There's a lot of helper functions too, like `parseFile` to parse a file.

Things worthy to remember before getting to work with it:

1. Match positions in text: by default `pyparsing` only prints them for the whole match *(or not at all if you used `searchString`)* which itself may consist of many small tokens. It is not very useful since most often a pattern comprises a non-interesting context *(which is only used to sort out false-positives)* and the actual core that one wants to extract.

   To make position of that "core" to appear, once you created it from tokens and before you meld it into the rest of the pattern, you call on it `myPatternObj = locatedExpr(myPatternObj)`.
2. Token positions in match results: they can be referred by index, but it is error-prone, since adding a new token will make token positions change. Instead you can name the token, and then refer to it by name in results. Example of how to create a named token: `token = Word(alphanums)('token_name')`
3. Extracting group values from regexps: [currently, getting them with index does not seem to be implemented](https://github.com/pyparsing/pyparsing/issues/252). Instead, you have to use named groups *(example of one: `(?P<name>content follows)`)*, which makes them behave as named tokens, see the prev. point.
    In my experience it's more useful to wrap the part of `Regex` you're interested into a `locatedExpr`. So you can create an expression as `locatedExpr(Regex(some pattern'))('pattern1') + locatedExpr(Regex('some other pattern'))('line_to_del')`.
