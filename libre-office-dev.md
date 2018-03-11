# Building

After `autogen`, use `make` to build and run test suit. `make build-nocheck` — no test-suit. `make sw.build` builds `sw` module, supposedly without tests. `make sw` builds the module, supposedly with tests.

# Links

* Docs: https://docs.libreoffice.org/
* Advices of what to look at for writing Writer: https://wiki.documentfoundation.org/Development/Writer
* LO Writer API docs *(SW for ancient StarWriter)*: https://docs.libreoffice.org/sw.html
* overview of basic Writer architecture: http://wiki.openoffice.org/wiki/Writer/Core_And_Layout http://wiki.openoffice.org/wiki/Writer/Text_Formatting
* diagram of code for opening a file in Writer http://kohei.us/wp-content/uploads/2012/05/file-load-process-diagram.png

# vmiklos

The blog http://vmiklos.hu/blog/ doesn't enlighten on filter writing*(11.09.2017)*, except of an article about RTF filter. However RTF filter is at `writerfilter/source/`, though most of them *(html one including)* is at `sw/source/filter`.

This `https://speakerdeck.com/vmiklos/libreoffice-rtf-import` don't have a video.

# Misc

`How to add a new Writer feature?`

Properties are at css1kywd.cxx.

File loading digaram: http://kohei.us/wp-content/uploads/2012/05/file-load-process-diagram.png

`librevenge::RVNGInputStream` data type is an input data. The definition is at `librevenge-stream` lib, RVNGStream.h file. Supports trivial operations, like `read()`, `seek()`. No write though, it's a constant stream.

Filters implement a `css::lang::XServiceInfo`, so that `cppu::supportsService()` could call the `getSupportedServiceNames()` to compare against an `rtl::OUString` arg.

`OdtGenerator &rGenerator` that `parse()` of AbiWordImportFilter accepts is declared in `parse()` declaration as `librevenge::RVNGTextInterface *documentInterface`

# glossary

`Filter` — the code to import/export a file format.

export filter — saves `SwDoc` to a stream.

import filter — opens the stream, and populates `SwDoc`.

css selector — object for the property to be applied.

# document model

## overview

Model-View-Controller pattern. One open document is `SwDoc`. Basic building block: paragraph (`SwNode`). Code is under `sw/source/core`. Page styles is `SwPageDesc`, and a document may have lots of other collections.

"First page header": `SwPageDesc::aFirst`

Literal quote, not sure of meaning: *No separate node for text runs, `SwpHints` instead. `SwTxtAttr` inside for each property: start/end/what*

## debugging

* Basic building block: paragraphs
* XML dump:
  * SW_DEBUG=1 ./soffice.bin --writer
  * Shift+F12 creates nodes.xml
* GDB:
  * `print pDoc->GetNodes()`
* UNO
  * Iterating over ThisComponent→paragraphs
  * Iterating over paragraphs→runs

Example in UNO:

    enum = ThisComponent.Txt.CreateEnumeration
    para = enum.NextElement
    para = enum.NextElement
    xray para

# Existing filters highlights

## ODF

Acc. to 2013y slides is a good example. Some highlights:

* ODF semantics are very close to Writer document model, e.g. UNO properties ≅ XML attributes.
* most of the implementation is an UNO filter.
* Code under `xmloff/`, `writerfilter`, and `sw/source/filter/xml/`

## RTF

Export filter is internal, core is shared with DOC/DOCX. Import filter is UNO, domain mapper is shared with DOCX.

## DOC

Tokenizer and domain mapper is not separated.

Import and export are both internal filters and somewhat shared.

## HTML

Most of the code is under `sw/source/filter/html/`.

A random snip from the middle of stracktrace to give general overview of its execution:

    #0  0x00007fffcbc49c1b in ParseCSS1_font_family(CSS1Expression const*, SfxItemSet&, SvxCSS1PropertyInfo&, SvxCSS1Parser const&)
    #1  0x00007fffcbc4befb in ParseCSS1_font(CSS1Expression const*, SfxItemSet&, SvxCSS1PropertyInfo&, SvxCSS1Parser const&)
    #2  0x00007fffcbc4f2ba in SvxCSS1Parser::ParseProperty(rtl::OUString const&, CSS1Expression const*)
    #3  0x00007fffcbc483f7 in SvxCSS1Parser::DeclarationParsed(rtl::OUString const&, CSS1Expression const*)
    #4  0x00007fffcbc456b4 in CSS1Parser::ParseRule()
    #5  0x00007fffcbc4535b in CSS1Parser::ParseStyleSheet()
    #6  0x00007fffcbc4629f in CSS1Parser::ParseStyleSheet(rtl::OUString const&)
    #7  0x00007fffcbc490db in SvxCSS1Parser::ParseStyleSheet(rtl::OUString const&)
    #8  0x00007fffcbbd65c5 in SwCSS1Parser::ParseStyleSheet(rtl::OUString const&)
    #9  0x00007fffcbbd64cb in SwHTMLParser::InsertLink()
    #10 0x00007fffcbc57feb in SwHTMLParser::NextToken(HtmlTokenId)
    #11 0x00007fffef7bdd51 in HTMLParser::Continue(HtmlTokenId)
    #12 0x00007fffcbc558f2 in SwHTMLParser::Continue(HtmlTokenId)
    #13 0x00007fffef7bdca7 in HTMLParser::CallParser()
    #14 0x00007fffcbc553ff in SwHTMLParser::CallParser()
    #15 0x00007fffcbc52e41 in HTMLReader::Read(SwDoc&, rtl::OUString const&, SwPaM&, rtl::OUString const&)
    #16 0x00007fffcbbaa354 in SwReader::Read(Reader const&)
    #17 0x00007fffcbce7311 in SwDocShell::ConvertFrom(SfxMedium&)
    #18 0x00007ffff23f5d9b in SfxObjectShell::DoLoad(SfxMedium*)
    #19 0x00007ffff242ff16 in SfxBaseModel::load(com::sun::star::uno::Sequence<com::sun::star::beans::PropertyValue> const&)
    #20 0x00007ffff24fcc1b in (anonymous namespace)::SfxFrameLoader_Impl::load(com::sun::star::uno::Sequence<com::sun::star::beans::PropertyValue> const&, com::sun::star::uno::Reference<com::sun::star::frame::XFrame> const&)
    #21 0x00007fffcea88baa in framework::LoadEnv::impl_loadContent()
    #22 0x00007fffcea84fb3 in framework::LoadEnv::startLoading()
    #23 0x00007fffcea180b7 in framework::LoadDispatcher::impl_dispatch(com::sun::star::util::URL const&, com::sun::star::uno::Sequence<com::sun::star::beans::PropertyValue> const&, com::sun::star::uno::Reference<com::sun::star::frame::XDispatchResultListener> const&)
    #24 0x00007fffcea17cb5 in framework::LoadDispatcher::dispatch(com::sun::star::util::URL const&, com::sun::star::uno::Sequence<com::sun::star::beans::PropertyValue> const&)
    #25 0x00007ffff23d3363 in (anonymous namespace)::IFrameObject::load(com::sun::star::uno::Sequence<com::sun::star::beans::PropertyValue> const&, com::sun::star::uno::Reference<com::sun::star::frame::XFrame> const&)
    #26 0x00007fffb11b523c in DocumentHolder::LoadDocToFrame(bool)
    #27 0x00007fffb11b2441 in DocumentHolder::ShowInplace()

`CSS1Parser` parses the content of a style element or a style option and preprocesses it. The example from the code:

    * H1, H2 { font-weight: bold; text-align: right }
    *  |  |                    |                  |
    *  |  |                    |                  DeclP( 'text-align', 'right' )
    *  |  |                    DeclP( 'font-weight', 'bold' )
    *  |  SelP( 'H2', false )
    *  SelP( 'H1', true )

`SvxCSS1Parser` is a child of `CSS1Parser`. Processes the `CSS1Parser` output by converting the CSS1 properties into SvxItem(Set).

`CSS1Parser::ParseDeclaration()` accepts CSS property *(at `ParseRule()`)*, and parses the value. Tho for some reason its loop doesn't stop on comma *(i.e. the start of next property)*. It saves the(??) property as 1-st arg, and returns its value. They're both acted upon in `CSS1Parser::DeclarationParsed()`.

`CSS1Parser::DeclarationParsed()` through `ParseProperty()` searches the function in the big table of property strings with functions to get called (insert the name), and calls one. E.g. for `font-family` it's `ParseCSS1_font_family()`.

Whole `CSS1Parser` extensively uses global variables. `GetNextToken()` saves next token to `aToken` and its type to `nToken`.

`CSS1Parser` have list of `CSS1Expression`s pointed by `pRoot` and `pLast`. But for some odd reason it doesn't have any information on what descriptor it is. I.e. it only saves the value and the css-type of value *(being `CSS1_STRING`)*.

Delegation of args to higher the stack happens through `SfxItemSet::Put()`. Example, given `rItemSet` is `SfxItemSet`:

    SvxFontItem aFont( FAMILY_DONTKNOW, aName, OUString(), PITCH_DONTKNOW,
                       eEnc, aItemIds.nFont );
    rItemSet.Put( aFont );

`IDENT`, i.e. `CSS1_IDENT`, includes *(but not limited to)* stuff like element `IDENT`, id selector `:#IDENT`, class `:IDENT`.

    enum CSS1SelectorType
    {
        CSS1_SELTYPE_ELEMENT, // "elem {…"; aToken is "elem"
        CSS1_SELTYPE_ELEM_CLASS, // "elem.class {…"; aToken is "elem.class"
        CSS1_SELTYPE_CLASS,
        CSS1_SELTYPE_ID,
        CSS1_SELTYPE_PSEUDO,
        CSS1_SELTYPE_PAGE // Feature: PrintExt
    };

The only place where `color` property is used is at `OutCSS1_SvxColor()` of css1atr.cxx. But `OutCSS1_SvxColor()` is only used *(directly)* at struct of type `SwAttrFnTab` which is used in few places at `sw/source/filter`.

`ParseCSS1_font_family()` makes me insane. I am not very familiar with the code, but it looks like *(aside of unused variables and unnecessary comparisons)* it does a bunch of jugglery by searching and creating strings whereas a simple enum should be enough. Anyway, this is where "font-family" of CSS happens — I checked with gdb, aName is the font name.

Might be interesting: `swhtml.cxx` has a code starting with `// now remove the last useless paragraph`. If very shortly, it gets the last node, does some setup, then deletes the node.

At the end of `ParseCSS1_background_color()` of svxcss1.cxx there is seting of background color.

`SvxCSS1Parser` class converts CSS properties into `SvxItem(Set)`.

## html todo:
### content
Wire up `content` property support. Support for counters is pointless otherwise.

`ParseSelector()`does some parsing, but its only influence to outside world is through the returned value, which in turn, higher the stack, gets into `m_Selectors`. Interestingly, there are two `SelectorParsed()`'s, and gets called not the one that doesn't do anything with args. The real one can only be seen in debugger.

`SwCSS1Parser::StyleParsed()` is where selectors and their styles meet each other. They're stored through `InsertClass()` into a std::map `m_Classes` for further processing.

`SwCSS1Parser::ParseStyleSheet()` calls `SvxCSS1Parser::ParseStyleSheet()` which does the parsing. FTR: with a simple CSS test about yellow background for a class, after the `ParseStyleSheet()` all those "page" codes below getting skipped through `if`s.

1. add `css1_double_semicolon` *(for starters make it working with a single one)*
2. at `ParseSelector()` reuse the parsing of `CSS1_IDENT`+`CSS1_DOT_W_WS`. To `eType` assign a new element of CSS1SelectorType.

* Look at how `aColor` is assigned. Possibly `content` has to be added there.
  ✓ yes, it's done. Some properties set in `pItemSet`.
* How does `SfxItemSet` know what class/element it belongs to ?
  class is accessed at `SwCSS1Parser::GetTextFormatColl()` (htmlcss1.cxx), the last `else` block in the end. The function sets the current style and its attributes, and is called from `NewHeading()` at swhtml.cxx
* Consider asking #libreoffice-dev of better implementation: I could manually cycle throuh nodes and the text, or is there an established property I have to set?


`ParseSelector()` is bailing out on `CSS1_COLON == nToken` to `default:`

# UI

Shared code is under `svx/source/dialog`  and `cui`. Writer specific code is under `sw/source/ui`.

# filter writing

Multiple ways of writing a filter (from http://fridrich.blogspot.ru/2013/08/extending-swiss-army-knife-overview.html):
1. xslt: way only fits to such mapping that don't require high-level language (i.e. loop through the data, modify, etc)
2. XFilter framework: requires writing a xml with filter description, and then the actual C++ code implementing certain interfaces (depends on the functionality). Judging by final paragraph of the article, it is the UNO way.

## tex

Checklist before kickstarter:

1. *Write a "hello world" input filter printing the input to stdout.* — ✓ using `read()` function of the `RVNGInputStream` allows to print content of the file.
2. Check whether working with "boxes" is supported through librevenge interface.
3. *Find out where latex packages stored, and whether the format is simple tex, so I can parse it with primivies.* — it's `.def` and `.sty` files, which looks like, and acc. to a tex.se are, latex or tex.
4. Write support for most common primitives, so I could show off a demo on kickstarter.

### the hello world

I've choosen to modify `writerperfect/source/writer/AbiWordImportFilter.cxx`. For some reason `.abw` matches some other filter — you have to choose AbiWord manually in the opening menu.

`cout` at `doImportDocument()` does work.

# TODO

aCSS1PropFnTab should be constexpr-sorted.

Unused variable `bFound` at `ParseCSS1_font_family`. A run with gcc warn-set could be useful.
