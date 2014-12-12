
# Making it easier with `make`

When I am writing a book I prefer as little distractions as possible.
Therefore I write most of the text with a pencil on paper.

Later I start the editing process with only two or three windows:

*   one window to edit the text,
*   one window to start the Leanpub jobs,
*   one window to look at the generated PDF output, because this is what needs
    the most tweaking.

Most of the time the first and the second window are combined to one terminal
window with two tabs so that I can switch between them with just a keystroke.

I'd like to share some insights about how I setup my environment so that I
only need a few keystrokes to create the latest formatted version of my book.

## Basic directory structure

I use a Dropbox folder to share my files with Leanpub and won't go into
details about setting this up.
The Leanpub documentation will lead you through the process.

Having a new Dropbox folder containing the books files, I start creating my
workspace with an empty directory.

In this directory I create a directory named images and two links referring to
the subdirectories *manuscript* and *preview* of the Dropbox folder.

    $ slug=using-the-leanpub-api-with-perl
    $ mkdir images
    $ ln -s ~/Dropbox/$slug/manuscript .
    $ ln -s ~/Dropbox/$slug/preview .

After that I copy all files from *manuscript* into the current directory and
all files from *manuscript/images* to the directory *images*

## Setup the Makefile

Now that I have everything in place it's time to setup the Makefile.

### Constants

First I define some constants which I will use later in different rules.

    DROPBOXDIR = manuscript
    DROPBOXFILES = $(DROPBOXDIR)/Book.txt \
                   $(DROPBOXDIR)/Sample.txt \
                   $(DROPBOXDIR)/Subset.txt \
                   $(DROPBOXDIR)/images/title_page.png \
                   $(DROPBOXDIR)/chapter01.md \
                   $(DROPBOXDIR)/chapter01-empty.md \
		   ...
    #

*DROPBOXDIR* contains the path to the Dropbox folder for this book.

*DROPBOXFILES* contains a list of all files that belong to that book and have
to be copied to the Dropbox folder before starting a job at Leanpub.

### Implicit rules

Those constants are followed by some implicit rules how to create certain files.
These are mostly copy commands:

    $(DROPBOXDIR)/%.md: %.md
	cp $< $@

    $(DROPBOXDIR)/%.txt: %.txt
	cp $< $@

    $(DROPBOXDIR)/images/%.png: images/%.png
	cp $< $@

    $(DROPBOXDIR)/images/%.jpg: images/%.jpg
	cp $< $@

These rules just say: if you need a file in `$(DROPBOXDIR)` and you have a
newer file with the same name in this directory, just copy it over to where
you need it.
The same holds for the files in directory *images* or *code* if I happen to
include code in my book.

Sometimes I like to have the full table of contents in the sample book but
without the text in most of the chapters.
To achieve this, I create files named *chapter$nr-empty.md* automatically from
the input:

    $(DROPBOXDIR)/%-empty.md: %.md
    	grep '^#' $< | grep -v '^###' | sed -e 's/^/\n/' > $@

So, whenever I don't want the text of a chapter but the 
titles, I replace `chapter$nr.md` with `chapter$nr-empty.md` in
*Sample.txt* and be done.

### Explicit targets

Besides the implicit rules I covered above, there are some explicit rules
`make` uses to perform its tasks.
These rules are also called *targets*.

The first explicit target is called `all` and does nothing.
So if I accidentally call `make` in this directory nothing unexpected occurs.
I could printout some help message about useful targets in this Makefile
instead.

There are four targets that I use regularly 

    dropbox: $(DROPBOXFILES)

This is a valid target although it contains no explicit action.
This rule makes sure that all necessary files are copied to the Dropbox
folder.

    partial: dropbox revision.md
            sleep 15 && leanpub partial_preview

This targets depends on the target `dropbox` so that it first makes sure that
all files are copied to the Dropbox folder and then starts its action, namely
sleep 15 seconds to allow Dropbox to copy those files to Leanpub and after
that starting the generation of a partial preview - using *Subset.txt* - for
the book.

    preview: dropbox revision.md
            sleep 15 && leanpub preview

This target does the same for a full preview.

    status:
            leanpub job_status

I prefer to use this target to call `make status` instead of
`leanpub job_status` just to save a few keystrokes.

