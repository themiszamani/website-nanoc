---
title: Integrating OmniGraffle and Latexmk
created_at: 2010-01-24
image: /notes/images/letterpress/
tags: [latex]
---
<% excerpt :summary do %>
Compiling LaTeX documents can be quite a hassle, because it's an iterative process, and the dependancies are not really clear.
There is one gem of a tool, however, that any regular LaTeXer should know: [Latexmk][].

Latexmk handles all the common TeX pipelines (DVI, PS, PDF…), detects most if not all the dependancies if your document is split into several source files, and will (re-)run LaTeX, BibTeX, etc. as needed, effectively reducing building most documents to this invocation:

    $ latexmk

Very nice indeed, but what about figures created in an external editor like [OmniGraffle][]? Going through OmniGraffle's menus to export documents to PDF after each change quickly gets boring.

[latexmk]: http://www.phys.psu.edu/~collins/software/latexmk-jcc/ "Latexmk's home page"
[omnigraffle]: http://www.omnigroup.com/applications/OmniGraffle/ "OmniGroup's diagramming application"
<% end %>


## Exporting from OmniGraffle automatically

First of all, we need a way to [convert a `.graffle` file to PDF automatically][graffle2pdf][^beware].
My solution was a short Ruby wrapper around an AppleScript invocation, but I guess Automator would work as well.
Just drop the script somewhere in your `$PATH` and make sure it's executable.
I didn't pay much attention to making the invocation flexible, because I only use it in an automatic way, so both input and output file names have to be specified explicitely:

    $ graffle2pdf figure.graffle figure.pdf

By the way, my script also includes the rule to use it with [Make][], just run `graffle2pdf >> Makefile` to append it:

    .SUFFIXES: .graffle .pdf
    .graffle.pdf:
            @echo Converting $< to PDF...
            @graffle2pdf $< $@


## Adding a custom rule to Latexmk

Latexmk already has many rules to handle the many steps in compiling a LaTeX document, and it's possible to specify more. Here's the one to call `graffle2pdf`:

    add_cus_dep( 'graffle', 'pdf', 0, 'graffle2pdf' );
    sub graffle2pdf {
       system("graffle2pdf $_[0].graffle $_[0].pdf");
    }

Append these lines to your `~/.latexmkrc` file, and you're done. The first line indicates to Latexmk that it can obtain a `.pdf` file from a `.graffle` one by calling the Perl procedure below. In this case it invokes the `graffle2pdf` script, but since this is Perl, it could do everything in place. Also, since this is Perl, the least I had to deal with it, the better :-)

For the curious, there are many more additional rules in the [example config file on CTAN][moreRules].


## Basic Latexmk configuration

My LaTeX pipeline of choice is PDFLaTeX; here are a few more options in `~/.latexmkrc` that I found useful:

    $pdf_mode = 1;
    $pdflatex = 'pdflatex -8bit -etex -file-line-error -halt-on-error -synctex=1 %O %S';
    $pdf_previewer = 'open %S';
    $pdf_update_method = 0;
    $clean_ext = "synctex.gz";
    
    @default_files = ('main.tex');

From top to bottom, these tell Latexmk:

- to generate PDF via `pdflatex`, with some options to get better command-line behavior and to activate [SyncTeX][] for jumping between editor and previewer (PDFsync has some strange interactions with style files we use);
- to open the generated PDF file the Mac way (I'm using [Skim][], for SyncTeX support);
- to include the SyncTeX file in the garbage when cleaning (`latexmk -C`).

The last line specifies that the document to compile is often named `main.tex`, so I can just type `latexmk` to build without thinking[^lmk]. This convention can be overridden on a per-document basis by setting the correct value for `@default_files` in a `.latexmkrc` or `latexmkrc` file besides the LaTeX sources.

_Happy LaTeXing!_


[graffle2pdf]: https://github.com/cdlm/infrastructure/blob/master/tools/graffle2pdf "graffle2pdf"
[make]: http://www.gnu.org/software/make/ "GNU Make"
[moreRules]: http://ctan.tug.org/tex-archive/support/latexmk/example_rcfiles/
[synctex]: http://en.foursenses.net/usingsynctex
[skim]: http://skim-app.sourceforge.net/
[^lmk]: In fact I have a shell alias that shortens that to just `lmk` :-)
[^beware]: Beware, the script contains a couple Unicode chevrons (`«class ppth»`) that will prevent execution if the encoding gets garbled.
