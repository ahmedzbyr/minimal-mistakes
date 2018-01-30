---
title: Remove Old Files using find Command
category: ['Linux', 'Rhel', 'Ubuntu', 'Commands']
tags: ['ubuntu', 'centos', 'linux', 'commands']
---

GNU find searches the directory tree rooted at each given file name by evaluating the given expression from left to right, according to the rules of precedence, until the outcome is known (the left hand side is false for and operations, true for or), at which point find moves on to the next file name. Remove old files which are older than a specific time using `find` Command.

### Command

~~~ ruby
find /path/to/files* -mtime +5 -exec rm {} \;
~~~
Note that there are spaces between `rm`, `{}`, and `\;` 

### Command Explanation.

~~~ ruby
-mtime n
	File's data was last modified n*24 hours ago. See the comments for -atime to 
	understand how rounding affects the interpretation of file modification times. 

-exec command {} +
	This variant of the -exec action runs the specified command on the selected 
	files, but the command line is built by appending each selected file name at 
	the end; the total number of invocations of the command will be much less than 
	the number of matched files. 
	The command line is built in much the same way that xargs builds its command lines. 
	Only one instance of '{}' is allowed within the command. 
	The command is executed in the starting directory.
~~~

Thats it.
