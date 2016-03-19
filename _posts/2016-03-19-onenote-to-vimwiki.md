---
title: "Converting a OneNote notebook to VimWiki"
author: "Alexey Shiklomanov"
date: 2016-03-19
layout: post
comments: true
---

While I was using Windows, I tried to use OneNote for all of my notetaking 
needs. However, as somebody that does a lot of coding, I was pretty much 
constantly dissatisfied with Windows and missed my Linux days, so a few days 
ago, I caved and made the switch back. I hardly ever use Word (even in 
Windows, I preferred to write my assignments in Markdown and compile them in 
Pandoc, or if I had to collaborate, to do so with Google Docs), but I did make 
extensive use of OneNote, and while I knew I *could* run it on Linux with 
Wine or a VM, I really wanted to ditch it altogether for an open source 
solution -- in my case, VimWiki.

For those of you like me that love Vim but haven't heard of vimwiki, it's is a 
powerful, markdown-like notetaking powerhouse that sits natively and happily 
inside vim and provides a great text-based tool for organizing your thoughts.

So my challenge was to export all of my OneNote notes into a format VimWiki 
could work with. Here are the caveats:

1. OneNote can't export plain text -- only `docx` and HTML (as well as some 
   other formats I don't really care about)

2. OneNote can't batch export multiple notes. If you try to export multiple 
   notes, it will combine them into a single document.

I can address (1) simply enough with Pandoc -- the Linux writer's Swiss army 
knife. For (2), I wrote a bash script (admittedly, a clunky one -- it was my 
first non-trivial bash script I've ever written, and if you have suggestions, 
please leave them in the comments!).


# OneNote to Docx to md

This step is pretty straightforward. First, I browse to the relevant section 
of my OneNote notebook, go to `File -- Export`, and select `docx`. (Why not 
`doc` you ask? Interestingly enough, pandoc can handle `docx` but not `doc`, 
possibly because the former is XML based while the latter is something less 
malleable? Don't quote me on that.) Then, I copy this over to my Linux box.

To check that the document looks OK, I open it in LibreOffice Writer. Since my 
notes are formatted in a pretty straightforward way and I don't really care 
about non-standard formatting, I also select the entire document and "Clear 
Direct Formatting" so it doesn't trip up pandoc too much.

Finally, once that's done, I run the following command:

    pandoc note.docx -o note.md

Yep, it's that simple! Now my entire notebook is in a single long Markdown 
file, so the challenge is to split it intelligently. On to step 2...


# Splitting the markdown file

The full script is reproduced at the bottom of the page, but here is an 
overview of the steps.

In a nutshell, the script leverages the face that the heading for every 
OneNote note looks almost exactly like this:

    Note title

    Monday, January 1, 2016

    12:35 PM

The title is variable and I have timestamps elsewhere, but I never write the 
date in exactly that format, so I can use that to identify individual notes. 
From there, I just use basic bash functions to cut the file and export the 
pieces.

To identify the date, I use `grep` with a very long pattern (because I have 
notes from every weekday and month).

    IFS=$'\n'
    matches=($(grep -nP "^(Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday), (January|February|March|April|May|June|July|August|September|October|November|December) \d{1,2}, \d{4}" note.md))

A few things to note:

* `IFS` stands for "internal field splitting" and is a bash variable that 
  determines what it considers "words". The default is a space, but here, I 
  set it to a newline (`\n`) because I will be working with lines that have 
  multiple space-separated words in them. In other words, my search will 
  return lines like `1768:Wednesday, 13 October, 2015` over which I want to 
  loop, but by default, bash would treat this as 4 objects: `1768:Wednesday, 
  `, `13`, `October,`, and `2015`. Setting the `IFS` variable fixes this.

  A note of caution though: Because `IFS` is a global variable, it 
  influences every command in my script, not just `grep`. In this script, this 
  isn't a problem because I don't have to split words anywhere else, but it's 
  something you should be aware of when changing its value manually.
  
* The `-nP` is two arguments: `-n` prints out the line numbers in addition to 
  the lines, and `-P` allows me to use the more extensible (or, at least, more 
  familiar to me) Perl-like regular expressions, rather than bash-like ones. 

* In Perl regular expressions, `^` indicates the start of the line, 
  `(abc|def)` means match `abc` **or** `def` exactly at that place in the 
  line, `\d` is a shortcut meaning a digit (there are others out there for 
  letters, alphanumeric numbers, whitespace, and more), and `{1,2}` indicates 
  that I match the previous item (in this case, a digit) at least once, but no 
  more than twice (later, the `\d{4}` means match exactly 4 digits).

* The `$(...)` construct stores the output of the `...` command as a variable. 
  The second set of parentheses, all the way on the outside, is bash syntax 
  for creating an array over which I can loop over (or access individual 
  elements). 

Next, I'm going to be looping over the file, but I want to do so in reverse 
order, so I can iteratively chop off the tail preserve the line numbers of 
each section. The cleanest way I found to do this is via a C-style loop over 
the indices:

    indices=( ${!matches[@]} )
    for ((i=${#indices[@]} - 1; i >=0; i--)); do

The first line uses bash object expansion syntax, signified by the `${...}`.  
The `[@]` (or, synonymously, `[*]`) selects every element in `matches` (unlike 
many other languages, entering just the array name `matches` returns only the 
first item. This probably designed to facilitate working with arrays of 
arguments and stuff, but I'm just speculating). The `!` tells bash to select 
the indices -- rather than the values -- of `matches`.

The second line defines the loop. In English, it translates to: "Starting with 
the final index (length [`#`] of `indices` minus one, since bash array indices 
start from zero), while `i` is greater than or equal to zero, reduce `i` by 1 
(`i--`) at every iteration". This loop syntax (`start; condition; change`) is 
identical to C and similar to Fortran, among others.

Next, recall that each matched line looks like this:

    4242:Saturday, February 30, 2015

We only want the line number (4242), so we can use more bash variable 
expansion syntax to extract it:

    linenum=${matches[i]%:*}

Breaking it down: `matches[i]` selects the ith element of `matches`. The `%` 
indicates that we want to delete stuff from the end of the object, and the 
pattern we use as a basis for that deletion is `:*`; in other words, "delete 
the last colon and everything after it". (If I wanted to instead keep only the 
stuff *after* the colon, I would use `#*:`, where the `#` matches from the 
front and `*:` is bash wildcard syntax meaning "everything up to and including 
the colon".) 

Next, I want to extract the note title and use it as the file name for my 
note. As I mentioned earlier, the note title is *always* 2 lines above the 
note date:

    linenum=${matches[i]%:*}

The title is whatever appears on that line. There are several commands that 
can get a line from a file by the line number, but according to a 
[StackOverflow 
comment](http://stackoverflow.com/questions/6022384/bash-tool-to-get-nth-line-from-a-file#comment34453410_6022431), 
the fastest command is this:

    title=$(tail -n+$titleline note.md | head -n1)

...which takes the end ("tail") of `note.md` starting from line number (`-n`) 
`$titleline` and, from that (`|`), takes the first line (`head -n1`).

I could be done there, but I'm actually a little more picky. Vimwiki diary entries are automatically named by date, and I want to preserve this in my imported files (especially since there are a lot of them, and I want to keep them organized in a logical manner). My OneNote notes also have names, but they are mostly written out in full ("Wednesday, 32 October 2015"), and don't always correspond to the day I created them, which is the date that is listed below the title (I like to plan days in advance and stick those plans into my daily notebook for that day). So my naming logic is as follows:

* If the note name can be interpreted as a date (by the useful bash command 
  `date -d`) then use that as the note name.

* Otherwise, use the original name, but prepend the note creation date to it.

Here's the resulting code:

    if date -d$title; then
        title=$(date -d$title +%Y-%m-%d)
    else 
        linedate=${matches[i]#*:}
        linedate=$(date -d+$linedate +%Y-%m-%d)
        title=$linedate'--'${title//\//_}
    fi
    outfile=$outdir/${title}.wiki

All bash commands return an integer code when they complete. Code 0 means the 
command executed successfully, while other codes mean something went wrong.  
This regularity can be leveraged by conditional statements. Here, the `date 
-d` command tries to interpret `$title` as a date. If it can, it converts this 
date to the `YYYY-MM-DD` format (`+%Y-%m-%d`). If it can't, it throws an error 
, which is picked up by the logic processing as `False` and proceeds to the 
`else` block. There, I grab the date (from the original matched line), convert 
it to vimwiki format, and concatenate it with the original title...with 
one caveat. In some of my older titles, I included the date in slash form 
(e.g.  12/7), which confuses Unix systems because they interpret slashes in 
paths as directories.  So I perform an internal character substitution via 
some fairly confusing bash variable expansion syntax to replace every (`//` -- 
a single slash means only replace the first) slash (`\/` -- note that it has 
to be escaped with a backslash) with an underscore (`_`). (Since that 
particular example was about as confusing as it could have been, I'll add for 
the record that the standard form for this syntax is `${var//x/y}`, meaning 
replace every instance of `x` with `y` in `var`; to only replace the first 
`x`, the syntax becomes `${var/x/y}`.)

Finally, with everything in place, I use `tail` to grab the section of 
`file.md` I need and paste it into a file with the appropriate title.

    tail -n+$titleline file.md > $outfile

Lather, rinse, repeat for every matched date line in `file.md` and we're done! 

The full bash script (with a few minor modifications for extensibility) is 
reproduced below:

    #!/bin/bash

    # Get arguments -- default to daily.format.md
    infile=${1-'daily.format.md'}

    outdir="output"
    mkdir -p $outdir
    cp -rf $infile $scratchfile

    # Find all lines that start with a date, formatted EXACTLY this way
    IFS=$'\n'
    matches=($(grep -nP "^(Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday), (January|February|March|April|May|June|July|August|September|October|November|December) \d{1,2}, \d{4}" $scratchfile))

    # Loop over indices in reverse
    indices=( ${!matches[@]} )
    for ((i=${#indices[@]} - 1; i >=0; i--)); do

        # Separate the line number and date from the full match string (% matches beginning, # matches end)
        linenum=${matches[i]%:*}

        # Line of note title (2 lines above date)
        titleline=$(($linenum-2))
        title=$(tail -n+$titleline $scratchfile | head -n1)
        if date -d$title; then
            title=$(date -d+$title +%Y-%m-%d)
        else 
            linedate=${matches[i]#*:}
            linedate=$(date -d+$linedate +%Y-%m-%d)
            title=$linedate'--'${title//\//_}
        fi
        outfile=$outdir/${title}.wiki

        ## Get output file string
        tail -n+$titleline $infile > $outfile

    done

    rm $scratchfile

    exit 0
