Personal Guides
======

Personal Guides for getting things done, notes on programming well, and programming in style. Various guides for
setting bash,multiple ssh's etc.

*  Guides on IPTABLES Rules with Example.
*  [Guides for Multiple SSH Keys settings for different github account.](/multiple_ssh_keys_settings.md)
*  [Rails Project Guidelines](/rails_guidelines.md)
*  [Sass Writing Guidelines](/sass_writing_practices.md)
*  [GIT Version Control Good Practices](/version_control_good_practices.md)
*  [GIT Cheat Sheet](/git_sheet.md)
   *  Git handy commands which might increase your productivity.

> 1 is stdout. 2 is stderr.
> Here is one way to remember this construct (altough it is not entirely accurate): at first, 2>1 may look like a good way to redirect stderr to stdout. However, it will actually be interpreted as "redirect stderr to a file named 1". & indicates that what follows is a file descriptor and not a filename. So the construct becomes: 2>&1.

```
The numbers refer to the file descriptors (fd).

Zero is stdin
One is stdout
Two is stderr
2>&1 redirects fd 2 to 1.

This works for any number of file descriptors if the program uses them.

You can look at /usr/include/unistd.h if you forget them:

/* Standard file descriptors.  */
#define STDIN_FILENO    0   /* Standard input.  */
#define STDOUT_FILENO   1   /* Standard output.  */
#define STDERR_FILENO   2   /* Standard error output.  */
That said I have written C tools that use non-standard file descriptors for custom logging so you don't see it unless you redirect it to a file or something.
```
