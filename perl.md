# `grep`, `sed`, and `awk` replacement

* Grep lines:      `perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' filename` *(or use `unless`)*
* Recursive grep:  `find cache-write/ -type f -exec perl -ne 'print $ARGV . ":" . $. . $_ if /pattern/' {} \;`
* Recursive grep with ack *(perl compatible regexps)*: `ack pattern`
* Modify lines:    `perl -i -pe 's/pattern/replacement/g' filename`
* Delete lines:    `perl -i -ne 'print if not /bar/' filename`
* Recursively modify files: `ack -l --print0 pattern | xargs -r0 perl -i -pe 's/pattern/replacement/g'`
* Print nth *(8th in example)* column, count starts from `0`: `perl -lane 'if (/pattern/) { print $F[8] }' ./file`

# hints

Use `ack` instead of `grep` or `find` + `perl`, since it has better syntax than find+perl, and perl-compatible regexps, so you can test them.
