## Seeing The World As The Shell Sees It

> What actually happens when you hit the `ENTER` key?

###Expansion

Let's first take a look at the `echo` command. `echo` is a shell builtin that performs a very simple task: it prints out
its text arguments on standard output.

```console
 $ echo razorbill
razorbill
```

That's very straightforward. Another example:

```console
 /usr                                                                                                                                 ✔  8109  22:41:00
 $ echo *
X11 X11R6 bin lib libexec local sbin share standalone
```

Why didn't `echo` just print "\*"? If you are familiar with wildcards, the `*` character means "match any characters in
a filename," but how the shell does that?

The simple answer is that the shell expands the `*` into something else -- in above example, the names of the files in
the current working directory -- before the `echo` command is executed. When the `ENTER` key is pressed, the shell
automatically expands any qualifying characters on the command line before the command is carried out, so the `echo`
command never saw the `*`, only its expanded result. Knowing this, we can see that `echo` behaved as expected.

#### Pathname Expansion

The mechanism by which wildcards work is called *pathname expansion*.

Given a home directory that looks like this:

```console
 $ ls
X11        X11R6      bin        lib        libexec    local      sbin       share      standalone
```

we could carry out the following expansions:

```console
 $ echo s*
sbin share standalone
```

or

```console
 $ echo [[:upper:]]*
X11 X11R6
```

> Pathname expansion of hidden files
> `echo .*`
> This almost works, however, if we examine the result closely, we will see that the name . and .. will also appear in
> the results.
> To correct perform pathname expansion in this situation, we have to employ a more specific pattern. This will work
> correctly:
> `ls -d .[!.]?*`
> This pattern expands into every filename that begins with a period, does not include a second period, contains at
> least one additional character, and may be followed by any other characters.

#### Tilde Expansion


