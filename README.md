# guaran-tee
A clone of the POSIX `tee` utility that doesn't copy output to stdout on a write error to the file.

## Rationale

The `tee` utility we all know and love typically ignores write errors to the given FILE.
Let's make up an example:

 - `data-src-cmd` is a command that gets data from somewhere that we want
   to back up before deleting.  It outputs to stdout.
 - `backup-file.txt` is the file we want to back stuff up to.
 - `delete-cmd` is a command that reads stuff from stdin and deletes it
   from the original source; like an awful database or something.
   
Now, you _could_ just do something like this:

    $ data-src-cmd > backup-file.txt
    $ cat backup-file.txt | delete-cmd
    $
    
But lets say you like to live on the edge, and want to do it all in one pipe:

    $ data-src-cmd | tee backup-file.txt | delete-cmd
    $
    
So, that's pretty rad but what happens if `backup-file.txt` doesn't have write
permission?  What happens if the disk fills up before this completes?  Anything
that'd cause a write error in `tee` while writing to `backup-file.txt` would
prevent that data from making to the file.  But on the other side of the `tee`,
`delete-cmd` is using stdin to delete stuff!  What happens in this case?

Take a look at the 
[POSIX spec](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/tee.html)
for `tee` and we'd find:

> If a write to any successfully opened file operand fails, writes to other successfully opened file operands and standard output shall continue, but the exit status shall be non-zero.
    
So `tee` will let us know via exit status, but will merrily forward stuff to
stdout while writes to the file fail.  For lots of usecases, that's probably
fine, but in our's we just deleted a bunch of stuff without getting a copy in
our archive.  FeelsBadMan

So the idea here is to make `guaran-tee` a clone of `tee` that only forwards
input to stdout once the write to the file suceeeds.  So we can be sure that
if anything makes it down to the other end of the pipe, we know it also made
it into the file.

## Possible issues

One of the uses flung around in the GNU coreutils manual for `tee` is using the
shell's process substiution syntax as the FILE argument to do stuff like write
a file through gzip to get a compressed file.  Our tee clone can reasonably
determine if the write to the file was successsul, but can it determine if,
in the case of process substitution, the other command got the data into a file?

## TODOs

 - Figure out if this is actually a good idea or not.
 
