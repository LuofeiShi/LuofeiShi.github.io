## The Linux Command Line notes

### Redirection

We are going to unleash a super cool feature of the command line: *I/O redirection*.
Some commands that I will cover here:
* cat -- Concatenate files
* sort -- Sort lines of text
* uniq -- Report or omit repeated lines
* wc -- Print newline, word, and byte counts for each file
* grep -- Print lines matching a pattern
* head -- Output the first part of a file
* tail -- Output the last part of a file
* tee -- Read from standard input and write to standard output and files

#### Standard Input, Output, and Error
**Redirect standard output**
I/O redirection allows us to redefine where standard output goes. To redirect standard output to antoher file intsead of
the screen, we use the `>` redirection operator followed by the name of the file. It's often useful to store the output
of a command in a file, like when you bootstrap an instance in cloud service, etc.

For example, we could tell the shell to send the output of the `ls` command to the file `ls_output.log` instead of the
screen:

```console
 $ ls -l /usr > ls_output.log
```

Here, we create a long listing of the `/usr` directory and sent the results to the file `ls_output.log`. Let's examine
the redirected output of the command:

```console
 $ ls -l ls_output.log
-rw-r--r--  1 luofeishi  wheel  552 Jul 21 23:43 ls_output.log
```

What if we change the name of the directory to one that doesn't exist?
```console
 $ ls -l /bin/usr > ls_output.log
ls: /bin/usr: No such file or directory
```

When we receive an error message, say for `ls` command, it doesn't send its error messages to standard output. Instead,
it sends its error messages to standard error. Since we redirected only standard output and not starnded error, the
error message was still sent to the screen.

Let's take a look of the output file:

```console
 $ ls -l ls_output.log
-rw-r--r--  1 luofeishi  wheel  0 Jul 22 00:14 ls_output.log
```

The length is zero! This is because when we redirect output with the `>` operator, the destination file is always
rewritten from the beginning. Since our `ls` command generated no results and only an error message, the redirection
operation started to rewrite the file and then stopped because of the error, resulting in its truncation.

In fact, if we ever need to actually truncate a file we can do this:

```console
 $  > ls_output.log
```

Using the redirection operator with no command preceding it will truncate an existing file or create a new, empty file.

To append redirected output to a file instead of overwriting the file from the beginning, we use the `>>` redirection
operator, like:

```console
 $ ls -l /usr >> ls_output.log
```

Using the `>>` operator will result in the output being appended to the file. If the file does not actually exist, it is
created just as though the `>` operator had been used.

**Redirect standard error**
To redirect standard error we must refer to its *file descriptor*. 

A program can produce output on any of several numberred file streams:
* standard input: 0
* standard output: 1
* standard error: 2
The shell provides a notation for redirecting files using the file descriptor number. Since standard error is the same
as file descriptor 2, we can redirect standard error with this notation:

```console
 $ ls -l /bin/usr 2> ls_output.log
```

The file descriptor 2 is placed immediately before the redirection operator to perform the redirection of standard error
to the file `ls_output.log`

**Redirecting standard output and standard errors to one file**
To capture all of the output of a command to  a single file, we can redirect both standard output and standard error at
the same time. There are two ways to do this:

The traditional way, which works with old version of the shell and probably **doesn't work** for most shells you are
using now:

```console
 $ ls -l /bin/usr > ls_output.log 2>&1
```

We perform two redirections with above command.

First we redirect standard output to the file `ls_output.log`, then we redirect file descriptor 2 (std err) to file
descriptor 1 (std out) using the notation `2>&1`.

**Notice that the order of the redirections is significant.**

A better (more recent) version is more streamlined method for performing this combined redirection:

```console
 $ ls -l /bin/usr &> ls_output.log
```

We use the single notation `&>` to redirect both standard output and standard error to the file `ls_output.log`.

**Disposing of unwanted output**
When we don't want output from a command, we just want to throw it away. This applies particularly to error and status
messages.

There is a special file called `/dev/null`. This file is a system device called a `bit bucket`, which accepts input and
does nothing with it. To suppress error messages from a command, we do this:

```console
 $ ls -l /bin/usr 2> /dev/null
```

>The bit bucket is an ancient Unix concept: https://en.wikipedia.org/wiki/Null_device A lot of nerds love to make jokes
>using /dev/null, like sending your messages to /dev/null :)

**Redirecting standard input**

The cat (concatenate files) command reads one or more files and copies them to standard output like so:

```console
cat [file]
```

Since `cat` can accept more than one file as an argument, it can also be used to join files together. Say we have
downloaded a large file that has been split into multiple parts, and we want to join them back together. This happens
a lot for my work when we have to deal with large csv files that split into pieces. If the files were named:

```
iad_1.csv, iad_2.csv, ..., iad_99.csv
```

we could rejoin them with this command:

```
$ cat iad_*.csv > iad_99.csv
```

Since wildcards always expand in sorted order, the arguments will be arranged in the correct order.

*What if we don't give any input to cat?*

```
> $ cat
```

If cat is not given any arguments, it reads from standard input, and since standard input is, by default, attached to
the keyboard, it's waiting for us to type something.

Try this:

```
 $ cat
Razorbill is the best bird on the earth.
Razorbill is the best bird on the earth.
```

Next, type `CTRL-D` to tell `cat` that it has reached *end-of-file (EOF)* on standard input.

In the absence of filename arguments, `cat` copies standard input to standard output, so we see our line of text
repeated. We can use this behavior to create short text files. Say we wanted to create a file called *I_love_cat.txt*
containing the text in our example, we would do this:

```
 $ cat > I_love_cat.txt
Razorbill is the best bird on the earth, but cat can hunt them.
```

And then to see the reuslt:

```
 $ cat I_love_cat.txt
Razorbill is the best bird on the earth, but cat can hunt them.
```

Now that we know how `cat` accepts standard input in addition to filename arguments, let's try redirecting standard
input:

```
 $ cat < I_love_cat.txt
Razorbill is the best bird on the earth, but cat can hunt them.
```

Using the `<` redirection operator, we change the source of standard input from the keyboard to the file
`I_love_cat.txt` This is not particularly useful compared to passing a filename argument, but it serves to demonstrate
using a file as a source of standard input. Other commands make better use of standard input, I will cover them soon.

Also, check out the man page for cat, it has several interesting options.

