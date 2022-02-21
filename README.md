# linux
Book notes: "How Linux Works"


### Chapter 1 - The Big picture
- (page 4)
    
    Because it's common to refer to the state in abstract terms rather than to the actual bits, the term _image_ refers to a particular physical arrangement of bits.

- (page 5)

    [The kernel] runs _between_ process time slices during a context switch.

- (page 7)

    Other than init, all new user processes on a Linux system start as a result of fork()

### Chapter 2 - Basic Commands and Directory Hierarchy
- `echo *` has the same output as `ls`
    - the shell will always replace wildcards/globs with matched files even if the command doesn't expect any filename
    - (19) if no files match the glob, the bash shell performs no expansion and the command runs with literal characters

- [The Jargon File](http://www.catb.org/jargon/html/)
- (25 table 2-2) command line keystrokes