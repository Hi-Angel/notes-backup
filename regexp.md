# Misc

* Often during parsing happens that both left and right sides of the regexp match. In this case left side is being preferred unless there's a lazyfying `?`, which makes the right one being preferred.
