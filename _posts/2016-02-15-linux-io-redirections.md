---
layout: post
title:  Linux I/O redirection examples
tags:
- Hacker
- Programming
- Linux
---

<br>

I/O redirections are one of the prettiest things we have in linux (IMO!) Following are commands and their usage.
<br><br>
`command_output >> file `
&nbsp; Redirects stdout to a file. Creates the file if not present, otherwise appends.

` > filename `
&nbsp; Truncates the file to zero length. If file is not present, creates zero-length file (same effect as `touch`).

` 1>filename `
&nbsp; Redirects stdout to the file "filename".

` 1>>filename `
&nbsp; Redirects and appends stdout to file "filename".

` 2>filename `
&nbsp; Redirects stderr to file "filename".

` 2>>filename `
&nbsp; Redirects and appends stderr to file "filename".

` &>filename `
&nbsp; Redirects both stdout and stderr to file "filename".

` 2>&1 `
&nbsp; Redirects stderr to stdout. Error messages get sent to same place as standard output.
<hr>
Some quality explanation now ;) Take the example of this command: 
<br>` cmd >> file.log 2>&1 `
<br>
This command will redirect all the output of command(cmd) into `file.log`.<br>
`2` refers to Second file descriptor of the process i.e., stderr<br>
`>` refers to redirection<br>
`&1` means that the target of redirection would be same as `1` i.e, first descriptor i.e, stdout.<br>


Thanks!

<style type="text/css">
code {
    font-weight: bold;
    font-size: 18px;
    background: #ddd;
    padding: 3px;
}   
</style>