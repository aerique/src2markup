# src2md

 author: Erik Winkels (<aerique@xs4all.nl>)  
created: 2010-03-17  
version: 0.7 (2018-10-16)  
license: BSD, see the end of this file

The canonical home page for this project is:
https://git.sr.ht/~aerique/src2markup

(The library is also pushed to GitLab and GitHub but those sites are not
monitored for support.)

## Introduction

`src2md` converts a source code file to a file parsable by
[Markdown](http://daringfireball.net/projects/markdown/).  This document
is an example of a source code file converted by `src2md`.

The plan for this software is to do 'inverse' literate programming.
Normal literate programming has its own format that needs to be run
through a tool to get compilable source code.  `src2md` starts from the
idea that one always has valid source code and that one extracts the
documentation from that.

Currently it works OK for single files but it needs to expand to be able
to compose nice docs from multiple source files with some directives.

### History

I have used [pbook.el](http://discontinuity.info/~pkhuong/pbook.el) in the
past and really like that approach to documenting code but it offered
too few options to markup the comments.  I could ofcourse have helped
improve pbook but this current approach is a hack that I had to get out of
my system (also known as the
[NIH](http://en.wikipedia.org/wiki/Not_Invented_Here) syndrome).

Also, `src2md` doesn't depend on Emacs which can be a good thing for some
people (not me).

## Download

<https://git.sr.ht/~aerique/src2markup/blob/master/src2md.lisp>

## Example Usage

*(assuming `src2md` has been made executable)*

* to see the generated Markdown: `./src2md src2md`
* to see the generated HTML: `./src2md src2md | markdown > src2md.html`

## Bugs / Problems

* Common Lisp: duplication of docstrings and comment documentation;
* Markdown: When you end your comment with a list the code that follows
            the comment will not be formatted as code, IMHO this is a bug
            in the standard Markdown
            (https://daringfireball.net/projects/markdown/).

## Changelog

v0.7: Update download URL. *(2018-10-16)*  
v0.6: Put code in div with background color. *(2013-02-27)*  
v0.5: Fixed some code getting appended to text. Removed coloring of code for now. *(2013-02-08)*  
v0.4: Added more comment tags and changed name from `cl2md` to `src2md`. *(2010-03-19)*  
v0.3: Some minor documentation changes. *(2010-03-18)*  
v0.2: Source code blocks are now in a different colour (\*code-colour\*). *(2010-03-18)*  
v0.1: Initial version. *(2010-03-17)*


## Parameters

   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defparameter *code-colour* "#6f0f00")

</div>

If a line starts with one of these it will treated as a comment (order is
important here because of `(length tag)` in `commentp`!):
   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defparameter *comment-tags* '("# " "#" "// " "//" "/* " "/*" "*/"
                                   ";;;; " ";;; " ";; " ";;;;" ";;;" ";;"))

</div>

If a line starts with one of these it will **not** be output:
   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defparameter *ignore-tags* '("#!" "#- " "#-" "//- " "//-" "/*- " "/*-"
                                  ";;;;- " ";;;- " ";;- " ";;;;-" ";;;-" ";;-"))


</div>

## Common Functions

This function is from [On Lisp](http://www.paulgraham.com/onlisp.html) by
Paul Graham:
   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defun mkstr (&rest args)
      (with-output-to-string (s)
        (dolist (a args) (princ a s))))


    (defun starts-with (sequence subsequence)
      "Returns T if SEQUENCE starts with SUBSEQUENCE otherwise returns NIL."
      (let ((sublen (length subsequence)))
        (when (and (> sublen 0)
                   (<= sublen (length sequence)))
          (equal (subseq sequence 0 sublen) subsequence))))


</div>

## Functions

### commentp

Returns either nil or the comment line with the comment characters (as
listed in `tags` / `*comment-tags*`) stripped away in front of it.
   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defun commentp (string &optional (tags *comment-tags*))
      "If STRING starts with any of the items in TAGS a substring of STRING is
      returned that has the TAG it matched on chopped off.
      If there are no matches NIL is returned."
      (loop for tag in tags
            when (starts-with string tag)
              do (return-from commentp (subseq string (length tag)))))


</div>

### ignorep & print-usage

These two functions speak for themselves:

   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defun ignorep (string &optional (tags *ignore-tags*))
      (loop for tag in tags
            when (starts-with string tag)
              do (return-from ignorep t)))


    (defun print-usage ()
      (format t "usage: src2md file
           Converts \"file\" to a format that can be parsed by the markdown-command
           and prints it to standard output.~%"))


</div>

### process-file

This is where most of the functionality resides.

`line` is set to a line from input and it is run through the `commentp`
function. Then:

* If `(ignorep line)` returns true the line is ignored;
* else if `commentp` contains a comment line it is printed with the comment
  characters stripped away;
* otherwise just `line` is output.

Normal lines which aren't comments (so actual code) are output with four
spaces in front of them so Markdown will treat them as code.
   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (defun process-file (file)
      (with-open-file (f file)
        (loop with last-line-comment = t
              for line = (read-line f nil nil)
              for commentp = (commentp line)
              while line
              do (cond ;; line that needs to be ignored
                       ((ignorep line))
                       ;; empty line
                       ((= 0 (length line))
                        (terpri))
                       ;; comment line
                       (commentp
                        (when (not last-line-comment)
                          (format t "</div>~%")
                          (terpri))
                        (setf last-line-comment t)
                        (format t "~A~%" commentp))
                       (t
                        (when (and last-line-comment
                                   (> (length line) 0))
                          (format t "   <div style=\"background-color: #efefef; margin: 16pt; padding: 4pt\">~%" *code-colour*)
                          (terpri))
                        (setf last-line-comment nil)
                        (format t "    ~A~%" line))))))


</div>

## Main Program

Very basic check for arguments.  If there are none help text will be
printed and if there are arguments then the first one is passed on to the
`process-file` function.

This should ofcourse be improved and extended bit it suffices for now.

   <div style="background-color: #efefef; margin: 16pt; padding: 4pt">

    (cond ((null *args*) (print-usage))
          (t (process-file (first *args*))))


</div>

## License

The BSD License

Copyright (c) 2010, Erik Winkels  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.

    * Redistributions in binary form must reproduce the above
      copyright notice, this list of conditions and the following
      disclaimer in the documentation and/or other materials provided
      with the distribution.

    * The name of its contributor may not be used to endorse or
      promote products derived from this software without specific
      prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
"AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
