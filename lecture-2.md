# Lecture 2: Shell Tools and Scripting
## Exercises:
1. Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner:
   - Includes all files, including hidden files  
   - Sizes are listed in human readable format (e.g. 454M instead of 454279954)  
   - Files are ordered by recency  
   - Output is colorized  

   <br>

   A sample output would look like this:  
   ```bash
   -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
   drwxr-xr-x   5 user group  160 Jan 14 09:53 .
   -rw-r--r--   1 user group  514 Jan 14 06:42 bar
   -rw-r--r--   1 user group 106M Jan 13 12:12 foo
   drwx------+ 47 user group 1.5K Jan 12 18:08 ..
   ```
   <br>

   **Solution**: 
   ```bash
   $ ls -laht --color
   total 56K
   drwxr-x--- 5 farhaan farhaan 4.0K Jul 11 00:05 .
   -rw------- 1 farhaan farhaan   20 Jul 11 00:05 .lesshst
   -rw-r--r-- 1 farhaan farhaan    0 Jul 10 23:47 .motd_shown
   -rw------- 1 farhaan farhaan 3.4K Jul  4 23:30 .bash_history
   drwxr-xr-x 3 farhaan farhaan 4.0K Jul  2 23:10 missing-semester
   -rw-r--r-- 1 farhaan farhaan   46 Jul  2 22:29 last-modified.txt
   -rw-r--r-- 1 farhaan farhaan   67 Jun 29 19:25 .gitconfig
   drwxr-xr-x 5 farhaan farhaan 4.0K Jun 29 19:18 .vscode-server
   -rw-r--r-- 1 farhaan farhaan  183 Jun 29 19:18 .wget-hsts
   -rw------- 1 farhaan farhaan 1.1K Jun 25 22:09 .viminfo
   drwxr-xr-x 3 farhaan farhaan 4.0K Jun 25 22:07 .local
   -rw-r--r-- 1 farhaan farhaan  220 Jun 25 21:58 .bash_logout
   -rw-r--r-- 1 farhaan farhaan 3.7K Jun 25 21:58 .bashrc
   -rw-r--r-- 1 farhaan farhaan  807 Jun 25 21:58 .profile
   drwxr-xr-x 3 root    root    4.0K Jun 25 21:58 ..
   ```  
   
2. Write bash functions `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`. For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`. 
   
   **Solution**:
   ```bash
   $ cat marco.sh
   
   #!/usr/bin/env bash
   marco () {
      export LAST_WORKING_DIRECTORY=$(pwd)
   }

   polo () {
      cd "$LAST_WORKING_DIRECTORY" && echo "Successfully changed directory to $LAST_WORKING_DIRECTORY"
   }

   $ source marco.sh
   $ marco
   $ cd tmp/foo/bar
   $ polo
   Successfully changed directory to /home/farhaan
   ```

3. Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.

   ```bash
   #!/usr/bin/env bash

   n=$(( RANDOM % 100 ))

   if [[ n -eq 42 ]]; then
      echo "Something went wrong"
      >&2 echo "The error was using magic numbers"
      exit 1
   fi

   echo "Everything went according to plan"
   ```

   **Solution**:
   ```bash
   $ cat debug.sh

   #!/usr/bin/env bash
   i=1

   while true
   do
      chmod +x test.sh
      ./test.sh > output.txt 2> error.txt

      if [[ $? -ne 0 ]]; then
         break
      fi

      ((i++))
   done

   echo "It took $i runs for the script to fail" 
   echo

   echo "Contents of output.txt:"
   cat output.txt
   echo    


   echo "Contents of error.txt:"
   cat error.txt

   $ chmod +x debug.sh
   $ ./debug.sh

   It took 197 runs for the script to fail

   Contents of output.txt:
   Something went wrong

   Contents of error.txt:
   The error was using magic numbers
   ```

4. As we covered in the lecture `find’s` `-exec` can be very powerful for performing operations over the files we are searching for. However, what if we want to do something with **all** the files, like creating a zip file? As you have seen so far commands will take input from both arguments and STDIN. When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments. To bridge this disconnect there’s the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments. For example `ls | xargs rm` will delete the files in the current directory. <br/><br/>Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`).<br/><br/> If you’re on macOS, note that the default BSD `find` is different from the one included in [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands). You can use `-print0` on `find` and the `-0` flag on `xargs`. As a macOS user, you should be aware that command-line utilities shipped with macOS may differ from the GNU counterparts; you can install the GNU versions if you like by [using brew](https://formulae.brew.sh/formula/coreutils).

   **Solution**:
   ```bash
   $ tree
   .
   ├── dir_1
   │   ├── test1.html
   │   ├── test2.html
   │   ├── test3.c
   │   ├── test4.py
   │   ├── test5.py
   │   └── yet another file with spaces.html
   ├── dir_2
   │   ├── test 3.html
   │   ├── test 4.html
   │   ├── test5.py
   │   ├── test6.c
   │   └── this is another file with spaces.html
   ├── file with many spaces.html
   ├── file1.html
   ├── file2.c
   ├── file3.py
   └── file4.html

   2 directories, 16 files

   $ find -name "*.html" | xargs -d '\n' tar -zvcf html_files.tar.gz

   ./file1.html
   ./dir_1/yet another file with spaces.html
   ./dir_1/test2.html
   ./dir_1/test1.html
   ./file4.html
   ./dir_2/test 3.html
   ./dir_2/test 4.html
   ./dir_2/this is another file with spaces.html
   ./file with many spaces.html
   
   $ tar -ztvf html_files.tar.gz 
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:28 ./file1.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-14 00:22 ./dir_1/yet another file with spaces.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:35 ./dir_1/test2.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:35 ./dir_1/test1.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:28 ./file4.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:51 ./dir_2/test 3.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-13 23:51 ./dir_2/test 4.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-14 00:21 ./dir_2/this is another file with spaces.html
   -rw-r--r-- farhaan/farhaan   0 2023-07-14 00:21 ./file with many spaces.html   
    ```

5. (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?

   **Solution:**
   ```bash
   % cat list_files_by_recency.sh
   #!/usr/bin/env bash

   readarray -t FILE_LIST < <(find . -type f -printf '%T@ %p\n' | sort -n -r)

   for file in "${FILE_LIST[@]}"; do
      if [[ "${file}" =~ ^([0-9]+\.[0-9]+)[[:space:]]+(.*) ]]; then
         timestamp="@${BASH_REMATCH[1]}"
         filename="${BASH_REMATCH[2]}"
         date_string=$(date -d $timestamp)
         echo "${filename} ${date_string}"
      fi
   done

   $ source list_files_by_recency
   ./list_files_by_recency.sh Sat Jul 15 17:50:33 PKT 2023
   ./dir_1/yet another file with spaces.html Fri Jul 14 00:22:02 PKT 2023
   ./dir_2/this is another file with spaces.html Fri Jul 14 00:21:43 PKT 2023
   ./file with many spaces.html Fri Jul 14 00:21:19 PKT 2023
   ./dir_2/test 4.html Thu Jul 13 23:51:51 PKT 2023
   ./dir_2/test6.c Thu Jul 13 23:51:19 PKT 2023
   ./dir_2/test5.py Thu Jul 13 23:51:19 PKT 2023
   ./dir_2/test 3.html Thu Jul 13 23:51:19 PKT 2023
   ./dir_1/test5.py Thu Jul 13 23:35:48 PKT 2023
   ./dir_1/test4.py Thu Jul 13 23:35:48 PKT 2023
   ./dir_1/test3.c Thu Jul 13 23:35:48 PKT 2023
   ./dir_1/test2.html Thu Jul 13 23:35:48 PKT 2023
   ./dir_1/test1.html Thu Jul 13 23:35:48 PKT 2023
   ./file4.html Thu Jul 13 23:28:30 PKT 2023
   ./file3.py Thu Jul 13 23:28:30 PKT 2023
   ./file2.c Thu Jul 13 23:28:30 PKT 2023
   ./file1.html Thu Jul 13 23:28:30 PKT 2023

   ```