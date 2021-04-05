A converter between many formats.

# Examples

## Converting a Markdown to PDF

Most of the stuff I took from [this post](https://jdhao.github.io/2019/05/30/markdown2pdf_pandoc/); and a bit from an answer on SO.

I use this command `pandoc -H preamble.tex -V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm" -V colorlinks -V urlcolor=NavyBlue --pdf-engine=lualatex -s ~/document.md -o document.pdf`

And preamble is:

```tex
\usepackage{fontspec}
\directlua{luaotfload.add_fallback
   ("emojifallback",
    {
      "NotoColorEmoji:mode=harf;"
    }
   )}

% set fallback for the main font.
\setmainfont{Ubuntu Mono}[
  RawFeature={fallback=emojifallback}
]

% change background color for inline code in
% markdown files. The following code does not work well for
% long text as the text will exceed the page boundary
\definecolor{bgcolor}{HTML}{E0E0E0}
\let\oldtexttt\texttt

\renewcommand{\texttt}[1]{
  \colorbox{bgcolor}{\oldtexttt{#1}}
  }
```

You can also use csv for styling, and then create an html with `pandoc -c /tmp/github-pandoc.css -V colorlinks -V urlcolor=NavyBlue -s ./document.md -o document.html`. `pdf` later then can be created with a browser by "printing to a file". NOTE: the css file has to be included with absolute path, otherwise browser won't find it.
