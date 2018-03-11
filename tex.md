# engine

The actual program that does the parsing. The common are pdfTex, XeTeX, LuaTex. They make use of a number of primitive instructions, like `\def`, `\outer`, `\expandafter`, `\noexpand`, `\futurelet`, `\relax`, `\catcode`, `\vbox`, `\hbox`,  `\accent`, `\kern`.

# format

`Latex2e` is a set of macros in use by over 20 years, most commmon. Compatible with `Plain Tex`. Code example:

    \documentclass{article}
    \begin{document}
    Hello World.
    \end{document}


`Plain Tex` is a set of macros used by D.Knuth to typeset his books. Compatible with `Latex2e`. Code Example:

    Hello
    \bye


`ConTeXt` is a specific format of LuaTex, incompatible with latex. Code Example:

    \starttext
    Hello World.
    \stoptext

`XeTeX` is like latex2e, but with macroses for unicode. Basically, it requires `\setromanfont{Ubuntu Mono}` to show cyrillic.

# tex syntax

Keywords are prepended by a backslash. According to `Tex by topic`, chapter `36.2 Keywords` the list is: at, bp, by, cc, cm, dd, depth, em, ex, fil, height, in, l, minus, mm, mu, pc, plus, pt, scaled, sp, spread, to, true, width.

Chapter 37 is a glossary of primitives. Some of them might look like macros, but they're the primitives implemented in TeX engine.

# misc

Box numbers are limited, probably due to restrictions on PCs' power of the time of the specification. I think I could make them the size of native uint, and later put a silly question on tex.se about the limitations, mostly to promote kickstarter project.

`\makebox` (latex) means making a virtual box around the text. The idea is to arrange the text inside the box. For example: `\makebox[30ex][s]{Censored text}\hspace{-30ex}\makebox[30ex][s]{X X X X X}` here "Censored text" is arranged (`s`pread) inside a box 30ex in width. Then the virtual caret moves back -30ex spaces, and the process repeated with text `X X X X X`. `framebox` does the same, but the box is visible *(thickness is configurable though)*.

`&` at least in latex (?) is a mark to align a text to *(it's easier to look up examples)*.

`\usepackage` is a latex macro. An analog in plain tex is `\input`, e.g.:

    \input amstex %
    \input harvmac %
    Some text here
    $$ y = mx + c $$
    \bye

# styles

see https://tex.stackexchange.com/questions/71028/displaystyle-dfrac-dcases As I understand there're 4 styles 1. math block(D), 2. inline math(T), 3. subscript or superscript(S), 4. sub-subscript or super-superscript and further(SS).

# tex category codes:

States of the TeX parser automata, to differ `{`, `$`, usual characters, etc. More here: https://tex.stackexchange.com/questions/16410/what-are-category-codes
