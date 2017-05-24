### Ruby one-liners

### Some fun stuff
```
ruby -h
Usage: ruby [switches] [--] [programfile] [arguments]
 -0[octal]       specify record separator (\0, if no argument)
 -a              autosplit mode with -n or -p (splits $_ into $F)
 -c              check syntax only
 -Cdirectory     cd to directory before executing your script
 -d              set debugging flags (set $DEBUG to true)
 -e 'command'    one line of script. Several -e's allowed. Omit [programfile]
 -Eex[:in]       specify the default external and internal character encodings
 -Fpattern       split() pattern for autosplit (-a)
 -i[extension]   edit ARGV files in place (make backup if extension supplied)
 -Idirectory     specify $LOAD_PATH directory (may be used more than once)
 -l              enable line ending processing
 -n              assume 'while gets(); ... end' loop around your script
 -p              assume loop like -n but print line also like sed
 -rlibrary       require the library before executing your script
 -s              enable some switch parsing for switches after script name
 -S              look for the script using PATH environment variable
 -T[level=1]     turn on tainting checks
 -v              print version number, then turn on verbose mode
 -w              turn warnings on for your script
 -W[level=2]     set warning level; 0=silence, 1=medium, 2=verbose
 -x[directory]   strip off text before #!ruby line and perhaps cd to directory
 -h              show this message, --help for more info
```


```
$ ruby -e 'puts "Hello, world!"'

$ cat ~/.ssh/devtools_geix_rsa
$ ruby -e 'puts File.read(ARGV[0])' ~/.ssh/devtools_geix_rsa

$ ruby -e 'puts File.read(ARGV[0]).inspect' ~/.ssh/devtools_geix_rsa
$ ruby -e 'puts File.read(ARGV[0]).strip.inspect' ~/.ssh/devtools_geix_rsa

# The best way: use AFGF object:
# ARGF a stream designed for use in scripts that process files given as
# command-line arguments or passed in via STDIN.

$ ruby -e 'puts ARGF.read.strip.inspect' ~/.ssh/devtools_geix_rsa
```


# Reviewing log data, extract event counts

Output top 15 lines (url only) grouped by url from data

```
head -n 1 NASA_access_log_Jul95

grep '01/Jul/1995' NASA_access_log_Jul95 |
  awk '{print $7}' |
  sort |
  uniq -c |
  sort -n -r |
  head -n 10

```

### Shell way
```
grep '01/Jul/1995' NASA_access_log_Jul95
grep '01/Jul/1995:23' NASA_access_log_Jul95
grep '01/Jul/1995:23' NASA_access_log_Jul95 | awk '{print $7}'
...

head -n 50 NASA_access_log_Jul95 | awk '{print $7}' | sort | uniq -c | sort -n -r | head -n 10

```

### Ruby way
Some ruby 'dirty' secrets:

1. Special global variable $_ ($LAST_READ_LINE)
2. `gets` method (IO class): reads the next “line” from the I/O stream ... The line read in will be returned and also assigned to `$_`.
3. `print` method (IO class): Writes the given object(s) to ios. When called without arguments, prints the contents of `$_`.

`-n` switch acts as
```ruby
while gets
  # execute code passed in -e here
end
```
So we can streaming lines with `-n` switch
```
ruby -ne 'print if $_.include?("01/Jul/1995")' NASA_access_log_Jul95
ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95

ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95 | ruby -ane 'puts $F[6]'
ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95 | ruby -ane 'puts $F[6]' | sort
ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95 | ruby -ane 'puts $F[6]' | sort | uniq -c
ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95 | ruby -ane 'puts $F[6]' | sort | uniq -c | sort -n -r
ruby -ne 'print if $_.include?("01/Jul/1995:23")' NASA_access_log_Jul95 | ruby -ane 'puts $F[6]' | sort | uniq -c | sort -n -r | head -n 10

```

### Printing lines with the -p switch
`-p` switch acts as

```ruby
while gets
  # execute code passed in -e here
  puts $_
end
```

So
```
echo 'Bash is the best!' | ruby -pe '$_.gsub!("Bash", "Ruby")'
```

Global `gsub` method operates on `$_`, but it also modifies it:
```
echo 'Bash is the best!' | ruby -pe 'gsub("Bash", "Ruby")'
```

### Inplace-Editing files
Using the -i option, you can modify files directy (just like sed's -i mode). For example, removing all trailing spaces:
```
$ ruby -ne 'puts $_.rstrip!' -i filename
```

### Auto-splitting Lines
The -a option will run `$F = $_.split` for every line:
```
$ ruby -nae 'puts $F.reverse.join(" ")' filename
```

### Prepending and Appending
```
printf "foo\nbar\nbaz\n" | ruby -ne 'BEGIN { i = 0 }; puts "#{i+1} #{$_}"; i+=1'
printf "foo\nbar\nbaz\n" | ruby -ne 'BEGIN { i = 0 }; puts "#{i+1} #{$_}"; i+=1; END {puts "Total: #{i} lines" }'

printf "foo\nbar\nbaz\n" | ruby -ne 'END{ puts $. }'
printf "foo\nbar\nbaz\n" | ruby -ne 'END{ puts "Total: #{$.} lines" }'
```

### The Ruby One-Liner Toolbox

CLI Options: `-n -p -0 -F -a -i -l`

Global Variables: `$_ $/ $\ $; $F $.`

Methods that operate on `$_`, implicitly: print ~

The special BEGIN{} and END{} blocks

|  Option	| Variable |	Description |
|---------|:--------:|:-------------|
|  -0	    | $/	|       Sets the input record separator, which is used by Kernel#gets. Character to use must be given as octal number. If no number is given (-0), it will use null bytes as separator. Using -0777 will read in the whole file at once. Another special value is -00, which will set $_ to "\n\n" (paragraph mode).|
|  -F	    |$;	  | Sets the input field separator, which is used by Array#split. Useful in combination with the -a option.|
|  -l	    | $\  |	Sets the output record separator to the value of the input record separator ($/). Also runs String#chop! on every line!|

### Links

[ruby-one-liners](http://reference.jumpingmonkey.org/programming_languages/ruby/ruby-one-liners.html)
