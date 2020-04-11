# `grep`, `sed`, and `awk` replacement

* Grep lines:      `perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' filename` *(or use `unless`)*
* Recursive grep with ack *(perl compatible regexps)*: `ack pattern`
* Recursive grep:  `find cache-write/ -type f -exec perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' {} +`
* Modify lines:    `perl -i -pe 's/pattern/replacement/g' filename`
* Delete lines:    `perl -i -ne 'print if not /bar/' filename`
* Recursively modify files: `ack -l --print0 pattern | xargs -r0 perl -i -pe 's/pattern/replacement/g'`
* Print nth *(9th in example)* column, count starts from `0`: `perl -lane 'if (/pattern/) { print $F[8] }' ./file`

## hints

Use `ack` instead of `grep` or `find` + `perl`, since it has better syntax than find+perl, and perl-compatible regexps, so you can test them.

# Misc

* booleans: "false" is anything of `undef`, `0`, `"0"`, `""`, and "true" is everything else. Note that this can lead to subtle error if you do `while($line)` where line is `"0"`.
* processing stdin: use `<>`. This operator is a call to `readline()` wrapped. You can do e.g. `my $line = <>` Or `while(<>) {…}` is a "for-each-input-line" loop. And here `map m{Directory.+trunk/(.+?)/}, <>` we do replacement similarly for each input line.
* String replacement: inside a `while(<>)` loop use `if (/pattern/)`
* String replacement delimiter changed: given a `if (/pattern/)`, add `m` to change delimiter, e.g. `if (m{pattern})`
* Set-builder notation: `my @arr = map m{Directory.+trunk/(.+?)/}, <>` here we map the function over stdin, and store result in `arr`
* Printing array without assigning to variable: you may need to explicitly order the code `print ((map m{Directory.+trunk/(.+?)/}, <>)[0])`.
* Note that some ways of splitting a string on newlines leave the newline inside the split string. Use `split /\n/, $mystr` or if you want a list of files in a dir, then `glob()`.
