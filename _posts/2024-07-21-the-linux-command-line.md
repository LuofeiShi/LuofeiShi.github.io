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


