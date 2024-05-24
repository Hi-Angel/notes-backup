# `grep`, `sed`, and `awk` replacement

* Grep lines:      `perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' filename` *(or use `unless`)*
* Recursive grep with ack *(perl compatible regexps)*: `ack pattern`
* Recursive grep:  `find cache-write/ -type f -exec perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' {} +`
* Modify lines:    `perl -i -pe 's/pattern/replacement/g' filename`
* Delete lines:    `perl -i -ne 'print if not /bar/' filename`
* Recursively modify files: `ack -l --print0 pattern | xargs -r0 perl -i -pe 's/pattern/replacement/g'`
* Print nth *(9th in example)* column, count starts from `0`: `perl -lane 'if (/pattern/) { print $F[8] }' ./file`

## hints

* Use `ack` instead of `grep` or `find` + `perl`, since it has better syntax than find+perl, and perl-compatible regexps, so you can test them.
* Few problems are reported by default, use `strict` and `warnings` modules while writing/debugging Perl code.

# Misc

* booleans: "false" is anything of `undef`, `0`, `"0"`, `""`, and "true" is everything else. Note that this can lead to subtle error if you do `while($line)` where line is `"0"`.
* processing stdin: use `<>`. This operator is a call to `readline()` wrapped. You can do e.g. `my $line = <>` Or `while(<>) {…}` is a "for-each-input-line" loop. And here `map m{Directory.+trunk/(.+?)/}, <>` we do replacement similarly for each input line.
* `@F` is the current line as array of columns, `$_` is the line as string.
* `qw(foo bar)` makes an array of strings `('foo', 'bar')`.
* String replacement: inside a `while(<>)` loop use `if (/pattern/)`
* String replacement delimiter changed: given a `if (/pattern/)`, add `m` to change delimiter, e.g. `if (m{pattern})`
* Set-builder notation: `my @arr = map m{Directory.+trunk/(.+?)/}, <>` here we map the function over stdin, and store result in `arr`
* Printing array without assigning to variable: you may need to explicitly order the code `print ((map m{Directory.+trunk/(.+?)/}, <>)[0])`.
* Note that some ways of splitting a string on newlines leave the newline inside the split string. Use `split /\n/, $mystr` or if you want a list of files in a dir, then `glob()`.
* `print /.*/` returns just `1`: read `Matching in list context` in `perldoc perlop`. Basically, it would by default print matched groups in regexp or just `1` for success if none groups were used. So you can make it work with either `print /(.*)/` or `print /.*/g`.
* `string to number conversion`: implicit. Just use a number-related op and conversion will happen automatically. E.g.: `perl -e 'print "5.45" + 0.1'`
* An example of summing up human-readable sizes from a file, represented by lines like `1 KiB`, `3M`, `4.7T`, etc: `perl -lane 'BEGIN{use Number::Bytes::Human qw(parse_bytes); my $sz = 0}; $sz += (parse_bytes $_); END{print $sz}' ./1.txt`
