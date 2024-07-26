## The Linux Command Line notes -- Redirection

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

#### Pipelines

The ability of commands to read data from standard input and send to standard output is utilized by a shell feature
called *pipelines*. Using the pipeline operator `|` (vertical bar), the standard output of one command can be *piped*
into the standard input of another.

```
command1 | command2
```

>The first Computer System course I took in college assigned a homework to write a shell, and writing the pipeline part
>literally took me a week. Well, mostly debugging. But there are so many edge cases to take care of!

To fully demonstrate this:

```
 $ ls -l /usr | less
```

This extremely handy! We can converniently examine the output of any command that produces standard output.

**Filter**

Pipelines are often used to perform complex operations on data. It is possible to put several commands together into
a pipeline. Frequently, the commands used this way are referred to as *filters*. Filters take input, change it somehow,
and then output it. The first one we will try is sort. Imagine we want to make a combined list of all of the executable
program in `/bin` and `/usr/bin`, put them in sorted order, and then view the list:

```
 $ ls /bin /usr/bin | sort | less
```

Since we specified two directories (`/bin` and `/usr/bin`), the output of `ls` would have consisted of two sorted lists,
one for each directory. By including `sort`in our pipeline, we changed the data to producea single, sorted list.

**uniq--report or omit repeated lines**

The `uniq` command is often used in conjunction with `sort`. `uniq` accepts a sorted list of data from either standard
input or a single filename argument (see the `uniq` man page for details) and, by default, removes any duplicates from
the list. So, to make sure our list has no duplicates (that is, programs of the same name that appears in both the `bin`
and `usr/bin` directories) we will add `uniq` to our pipeline:

```console
 $ ls /bin /usr/bin | sort | uniq | less
```

In this example, we use `uniq` to remove any duplicates from the output of the `sort` command. If we want to see the
list of duplicates instead, we add the `-d` option to `uniq` like so:

```
 $ ls /bin /usr/bin | sort | uniq -d | less
```

**wc--print line, word, and byte counts**

The `wc` (word count) command is used to display the number of lines, words, and bytes contained in files. For example:

```
 $ wc ls_output.log
        1       7      40 ls_output.log
```

In this case it prints out three numbers: lines, words, and bytes contained in `ls_output.txt`. Liek our previous
commands, if executed without command-line arguments, `wc` accepts standard input. The `-l` option limits its output to
only report lines. Adding it to a pipeline is a handy way to count things. To see the number of items we have in our
sorted list, we can do this:

```
 $ ls /bin /usr/bin | sort | uniq | wc -l
    1023
```

**grep--print lines matching a pattern**

`grep` is a powerful program used to find text patterns within files, like this:

```
grep pattern [file...]
```

When `grep` encounters a "pattern" in the file, it prints out the lines containing it. The patterns that grep can match
can be very complex, but for now we will concentrate on simple text matches.

Let's say we want to find all the files in our list of program that have the word `zip` in the name. Such a search might
give us an idea of which programs on our system have something to do with file compression. We would do this:

```console
 $ ls /bin /usr/bin | sort | uniq | grep zip
bunzip2
bzip2
bzip2recover
funzip
gunzip
gzip
...
```
There are a couple of handy option for grep: `-i`, which causes grep to ignore case when performing the search (normally
searches are case sensitive) and `-v`, which tells grep to print only lines that do not match the pattern.

**head/tail--print first/last part of files**

Sometimes you don't want all the output from a command. You may just want the first or last couple lines. The `head`
command prints the first 10 lines of a file, and the `tail` command prints the last 10 lines. By default, both commands
print 10 lines of text, but this can be adjusted with the `-n` option:

```console
 $ head -n 5 adobegc.log
07/13/24 10:04:17:479 | [INFO] |  |  |  | AdobeGCInvokerUtilityHelper |  |  | 5405 | AdobeGCInvokerUtility : inside getLatestGCApplication
07/13/24 10:04:17:482 | [INFO] |  |  |  | AdobeGCUpdater |  |  | 5405 | ***********AdobeGC Updater library invoked = 8.4.0.73 ************
07/13/24 10:04:17:482 | [INFO] |  |  |  | AdobeGCUpdaterCFU |  |  | 5405 | Perform WF started
07/13/24 10:04:17:482 | [INFO] |  |  |  | AdobeGCUpdaterCFU |  |  | 5405 | Perform WF completed
07/13/24 10:04:17:482 | [ERROR] |  |  |  | AdobeGCInvokerUtilityHelper |  |  | 5405 | AdobeGCInvokerUtility :inside waitForCFUCallBack.
```

These can be used in pipelines as well:

```console
 $ ls /usr/bin | tail -n 5
zipsplit
zless
zmore
znew
zprint
```

`tail` has an option that allows you to view files in real time. This is very useful for my work when I need to
bootstrap an application and need to trace log for signals or errors:

```
 $ tail -f /var/logs/keybagd.log.0

Thu Jul 25 02:54:29 2024 [118] <err> (0x700001897000) analytics_send_fv_status: AnalyticsEvent: fv_enabled: 0, fv_users_count: 1, icloud_recovery_key: 1, institutional_recovery_key: 0, personal_recovery_key: 0, mdm_recovery_key: 0, installer_user: 0, icloud_recovery_user: 1, institutional_recovery_user: 0, vek_device_protected: 0, vek_ephemeral: 0, vek_is_owner: 0, vek_boot_policy: 0, vek_imported: 0, kek_sidp_count: 0, kek_ps_count: 0, kek_last_count: 0, kek_imported_count: 0, kek_bad_sig_count: 0, kek_xart_policy_missing_count: 0, kek_ps_missing_count: 0
Thu Jul 25 02:54:29 2024 [118] <err> (0x700001897000) analytics_send_passcode_status: AnalyticsEvent: is_set: 1, type: 0, activation_status: 0
Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) data_analytics_init_block_invoke: data analytics activity
Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) analytics_send_kek_stats: AnalyticsEvent: xart_policy: 1, xart_policy_enforced: 0, xart_policy_missing: 0
Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) analytics_send_kek_stats: AnalyticsEvent: xart_policy: 1, xart_policy_enforced: 0, xart_policy_missing: 0
Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) dump_fv_blob_state: failed to get blob_state: (e007c013)

Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) analytics_send_fv_status: AnalyticsEvent: fv_enabled: 0, fv_users_count: 1, icloud_recovery_key: 1, institutional_recovery_key: 0, personal_recovery_key: 0, mdm_recovery_key: 0, installer_user: 0, icloud_recovery_user: 1, institutional_recovery_user: 0, vek_device_protected: 0, vek_ephemeral: 0, vek_is_owner: 0, vek_boot_policy: 0, vek_imported: 0, kek_sidp_count: 0, kek_ps_count: 0, kek_last_count: 0, kek_imported_count: 0, kek_bad_sig_count: 0, kek_xart_policy_missing_count: 0, kek_ps_missing_count: 0
Thu Jul 25 20:46:34 2024 [118] <err> (0x70000191a000) analytics_send_passcode_status: AnalyticsEvent: is_set: 1, type: 0, activation_status: 0
```

Using the `-f` option, tail continues to monitor the file and when new lines are appended, they immediately appear on
the display. This continues until you type `CTRL-C`.

**tee--read from stdin and output to stdout and files**

In keeping with our plumbing analogy, Linux provides a command called tee which creates a "T" fitting on our pipe. The
tee program reads standard input and copies it to both standard output (allowing the data to continue down the pipeline)
and to one or more files. This is useful for capturing a pipeline's contents at an intermediate stage of processing.
here we repeat one of our earlier example, this time including tee to capture the entire directory listing to the file
`ls.txt` before grep filter the pipeline's contents:

```
 $ ls /usr/bin | tee ls.txt | grep zip
bunzip2
bzip2
bzip2recover
funzip
gunzip
gzip
streamzip
unzip
unzipsfx
zip
zipcloak
zipdetails
zipdetails5.30
zipdetails5.34
zipgrep
zipinfo
zipnote
zipsplit
```

#### Final note

Check out the documentation of each of the commands we have covered in this blog. We have seen only their most basic
usage, and they all have a nuber of interesting options. As we gain Linux experience, we will see that the redirection
feature of the command line is extremely useful for solving specialized problems. Many commands make use of standard
input and output, and almost all command-line programs use standard error to display their informative messages.
