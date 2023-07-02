# Lecture 1: Course overview + the shell
## Exercises:
### 1. Create a new directory called `missing` under `/tmp` 

   > ### **Solution**:
   >> ```$ cd /tmp```  
   >> ```$ mkdir missing```  
   
### 2. Look up the `touch` program. The `man` program is your friend.
   
   > ### **Solution**:
   >> ```$ man touch```  

### 3. Use `touch` to create a new file called `semester` in `missing`.
   > ### **Solution**:
   >> ```$ touch semester```  

### 4. Write the following into that file, one line at a time:

  > ```#!/bin/sh```   
  > ```curl --head --silent https://missing.csail.mit.edu``` 

### The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.

> ### **Solution**:
   >> ```$ echo '#!/bin/sh' > semester```  
   >> ```$ echo 'curl --head --silent https://missing.csail.mit.edu' >> semester```

### 5. Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).

> ### **Solution**:
   >> ```$ ./semester```  
   >> ```bash: ./semester: Permission denied```  
   >> ```$ ls -l```  
   >> ```total 4```  
   >> ```-rw-r--r-- 1 farhaan farhaan 61 Jul  2 19:01 semester```  
> ### The file does not have the execute permission therefore it fails to run

### 6. Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?

> ### **Solution**:
   >> ```$ sh semester```  
   >> ```HTTP/2 200 ```  
   >> ```server: GitHub.com```  
   >> ```content-type: text/html; charset=utf-8```  
   >> ```last-modified: Sat, 10 Jun 2023 13:13:30 GMT```  
   >> ```access-control-allow-origin: *```  
   >> ```etag: "648476fa-1f86"```  
   >> ```expires: Sun, 02 Jul 2023 14:20:53 GMT```  
   >> ```cache-control: max-age=600```  
   >> ```x-proxy-cache: MISS```  
   >> ```x-github-request-id: 4862:0FA2:2F10D9:38D380:64A18569```  
   >> ```accept-ranges: bytes```  
   >> ```date: Sun, 02 Jul 2023 14:10:53 GMT```  
   >> ```via: 1.1 varnish```  
   >> ```age: 0```  
   >> ```x-served-by: cache-fjr990028-FJR```  
   >> ```x-cache: MISS```  
   >> ```x-cache-hits: 0```  
   >> ```x-timer: S1688307053.246796,VS0,VE245```  
   >> ```vary: Accept-Encoding```  
   >> ```x-fastly-request-id: 7145995a7d4e63fb2afb290ea067182ad06b4fc2```  
   >> ```content-length: 8070```  
> ### This works because the `sh` interpreter treats the `semester` file as a shell script and runs it, irrespective of whether it has execute permission or not. Running it directly i.e. `./semester` doesn't work because the file doesn't have the execute permission

### 7. Look up the chmod program (e.g. use `man chmod`)
> ### **Solution**:
   >> ```$ man chmod```  

### 8. Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information

> ### **Solution**:
   >> ```$ chmod +x semester```  
   >> ```$ ./semester```  
   >> ```HTTP/2 200 ```  
   >> ```server: GitHub.com```  
   >> ```content-type: text/html; charset=utf-8```  
   >> ```last-modified: Sat, 10 Jun 2023 13:13:30 GMT```  
   >> ```access-control-allow-origin: *```  
   >> ```etag: "648476fa-1f86"```  
   >> ```expires: Sun, 02 Jul 2023 14:20:53 GMT```  
   >> ```cache-control: max-age=600```  
   >> ```x-proxy-cache: MISS```  
   >> ```x-github-request-id: 4862:0FA2:2F10D9:38D380:64A18569```  
   >> ```accept-ranges: bytes```  
   >> ```date: Sun, 02 Jul 2023 17:23:05 GMT```  
   >> ```via: 1.1 varnish```  
   >> ```age: 0```  
   >> ```x-served-by: cache-fjr990028-FJR```  
   >> ```x-cache: MISS```  
   >> ```x-cache-hits: 0```  
   >> ```x-timer: S1688307053.246796,VS0,VE245```  
   >> ```vary: Accept-Encoding```  
   >> ```x-fastly-request-id: 7145995a7d4e63fb2afb290ea067182ad06b4fc2```  
   >> ```content-length: 8070```  
> ### The shebang line (`#!/bin/sh`) tells the shell that the `sh` interpreter is to be used for interpreting the `semester` file

### 9. Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.

> ### **Solution**:
   >> ```$ ./semester | grep "last-modified" > ~/last-modified.txt```  
   
### 10. Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from /sys. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

> ### **Solution**:
   >> ```$ cat /sys/class/power_supply/BAT1/capacity```  
   >> ```46```
> ### The above command reads the laptop's battery power level and outputs it