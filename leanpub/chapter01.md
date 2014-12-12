# Using the Perl module

This book describes the Perl module *WebService::Leanpub* and how I use it to
produce books.
Using the module it is possible to access the Leanpub API from
within Perl or from the command line.
It is mainly useful to authors at Leanpub and to those who help them to
produce their books.

## What is the Leanpub API

The Leanpub API is a web API specified in <https://leanpub.com/help/api>.

I do need an API key for most of the functions of this API.
I can get this key in my dashboard at Leanpub, going to the *Account* tab and
scrolling to the end of the page.
Using this key I can work on all my books at Leanpub.

A further essential component is the slug, the part of the URL for your book
following <https://leanpub.com/>.
For instance the URL of this book *Using the Leanpub API with Perl* is
<https://leanpub.com/using-the-leanpub-api-with-perl> and the slug is
therefore **using-the-leanpub-api-with-perl**.

## What can I do with it?

Using the API I can

*   create previews for the book, parts of the book or individual files,
*   publish the book,
*   get the book summary information (this is the only function that works
    without an API key),
*   query for the status of the last job,
*   get the sales data,
*   and create coupons for the book as well as get a list of the coupons.

## How do I use the Perl module?

All actions related to one book take place as method calls to an object of
type `WebService::Leanpub`.
For this reason I provide the API key and the slug when using the function
`new()` to create such an object.

    use WebService::Leanpub;

    my $wl = WebService::Leanpub->new($api_key, $slug);

Using this object I call different methods, depending from what I want to do.
These methods return the text of the web API in JSON format which I can print
or interpret in my program.

### Preview

To create a complete or partial preview is really easy, there is a method for
each of these tasks.
In the files *Book.txt* and *Subset.txt* respectively in the Leanpub
manuscript directory are the lists of the files to be used for the preview.
Leanpub authors know how look like.
So I simply have to call

    $pv = $wl->preview();

or

    $pv = $wl->subset();

with my `WebService::Leanpub` object.

Creating a preview for an individual file is a little bit more elaborate
because I have to send the file to the web API using a POST request.

The following example shows how to open a file, read its content into a scalar
variable and send it to the web API.

    if (open(my $input, '<', $filename)) {
      local $/;
      undef $/;
      my $content = <$input>;
      close $input;
    
      $pv = $wl->single({ content => $content });
    }

### Publish

Publishing a book is easier.
I can advise whether the readers get an email and what should be the content
of that email.
You can find details in the man page of the module.

    $pv = $wl->publish( $opt );

### Getting the status of the running job

Because I only start the creation of a preview or the publishing process of my
book, it may happen that I want to know the state of the affairs.
This is as easy as it can be:

    $pv = $wl->get_job_status();

### Summary of the book

To get a summary of the book I just need to call the function `summary()`.

    $pv = $wl->summary();

The web API doesn't need an API key for this task but the function `new()`
that created my `WebService::Leanpub` object needed one.
If I want to know how the summary looks like without an API key, I have to
call `WebService::Leanpub->new()` using a false API key.

### Sales data

There are two methods to get the sales data.

I use one for a summary of the sales data and the other for information about
the individual purchases.

    $pv = $wl->get_sales_data();
    
    $pv = $wl->get_individual_purchases( { } );
    
    $pv = $wl->get_individual_purchases( { page => 2 } );

The function `get_individual_purchases()` delivers the data for the last 50
purchases.
If I want to get older data, I have to give the page number of the older data.
A page contains at most data for 50 purchases.
The first page (`page => 1`) contains the data of the most recent purchases,
the higher the page number, the older the sales data.

### Coupons

I can create, list and change coupons.

    $pv = $wl->create_coupon(\%lopt);

    $pv = $wl->get_coupon_list()

    $pv = $wl->update_coupon(\%lopt);

Details can be found in the documentation of the module in chapter 2.

## I don't want to program

I don't need to program - with Perl - anymore, because there is a command line
program called `leanpub` which is included in the distribution of the Perl
module.
Using this I can utilize the Leanpub API in my Makefiles.

The program uses `Getopt::Long` and `Pod::Usage`, so it's very easy to get a
short help text with the command line option `-h` or `--help` and the full man
page with option `-m` or `--man`.

    $ leanpub --help
    Usage:
         leanpub [options] command [command options]
    ...

When calling the program I tell the program what to do with the command
argument.
These command arguments are called like the methods of the Perl module.

I may add some global options before the command and some command specific
options after the command.
The man page of the program - contained in chapter 3 - goes into the details.

    Options:
      -api_key=key
              Provide the Leanpub API key to be used for all actions.
      ...
      -really State that you really intend to do the command (e.g. publish).
      ...
      -slug=your_book
              Provide the book's slug.
      ...

The global options `-api_key` and `-slug` are so important that I have to
provide them nearly every time I call the program.

Because this is tedious I can provide these in a configuration file called
*.leanpub*.

    $ cat .leanpub
    # configuration for leanpub
    #
    api_key = my_api_key_from_leanpub
    slug    = using-the-leanpub-api-with-perl

The option `-really' may look a little bit strange at first.

Really?

I could have named it `-do-as-I-say` but this seemed a little bit long for me.

This option is only active with the command `publish` and I need this option
for this command.
It's a kind of emergency break so that I do not publish the book
unintentionally when it is not ready to be published.

## Upshot

I use the module `WebService::Leanpub` or more precisely the command line
program `leanpub` together with an appropriate Makefile daily when working on
my books.
It fits seamless into my workflow which consists of an editor window, a shell
and a PDF viewer.
Using this I avoid interruptions - and possible distractions - from switching
to the web browser.

