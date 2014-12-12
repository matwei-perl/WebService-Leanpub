# Makefile for leanpub / dropbox

BOOK = Book.txt
PREVIEW = Subset.txt
SAMPLE = Sample.txt

DROPBOXDIR = manuscript

DROPBOXFILES = $(DROPBOXDIR)/$(BOOK) \
               $(DROPBOXDIR)/$(PREVIEW) \
               $(DROPBOXDIR)/$(SAMPLE) \
               $(DROPBOXDIR)/preface.md \
               $(DROPBOXDIR)/chapter01-empty.md \
               $(DROPBOXDIR)/chapter01.md \
               $(DROPBOXDIR)/chapter02-empty.md \
               $(DROPBOXDIR)/chapter02.md \
               $(DROPBOXDIR)/backmatter.md \
               $(DROPBOXDIR)/appendix01.md \
               $(DROPBOXDIR)/appendix02.md \
               $(DROPBOXDIR)/module.md \
               $(DROPBOXDIR)/cli.md \
               $(DROPBOXDIR)/images/title_page.png \
#
IMAGES = \
#
$(DROPBOXDIR)/%.md: %.md
	cp $< $@

$(DROPBOXDIR)/%-empty.md: %.md
	grep '^#' $< | grep -v '## Tip' | sed -e 's/^/\n/' > $@

$(DROPBOXDIR)/%.txt: %.txt
	cp $< $@

$(DROPBOXDIR)/code/%: code/% $(DROPBOXDIR)/code
	cp $< $@

$(DROPBOXDIR)/code: $(DROPBOXDIR)
	mkdir $(DROPBOXDIR)/code

$(DROPBOXDIR)/images/%.jpg: images/%.jpg
	cp $< $@

$(DROPBOXDIR)/images/%.png: images/%.png
	cp $< $@

$(DROPBOXDIR)/module.md: ../lib/WebService/Leanpub.pm
	pod2markdown < $< | perl -p -e 's/^#/##/' > $@

$(DROPBOXDIR)/cli.md: ../bin/leanpub
	pod2markdown < $< | perl -p -e 's/^#/##/' > $@

all:

dropbox: $(DROPBOXFILES)

preview: dropbox
	leanpub preview

status:
	leanpub job_status

# end of Makefile
