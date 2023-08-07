# Lecture 5: Command-line Environment

## Exercises

### Job control

1. From what we have seen, we can use some `ps aux | grep` commands to get our jobs’ pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).

   **Solution**:

   ```bash
   $ sleep 10000
   ^Z
   [1]+  Stopped                 sleep 10000
   $ bg %1
   [1]+  Stopped                 sleep 10000
   $ pgrep -a sleep
   8164 sleep 10000
   $ pgrep -f sleep
   [1]+  Terminated              sleep 10000
   $ jobs

   ```

1. Say you don’t want to start a process until another completes. How would you go about it? In this exercise, our limiting process will always be `sleep 60 &`. One way to achieve this is to use the [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes. <br><br>However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command’s exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist. Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily

   **Solution**:

   ```bash
   $ cat pidwait.sh
   #!/usr/bin/env bash

   function pidwait(){
      while true
      do
         kill -0 $1
         if [[ $? -ne 0 ]]; then
            break
         fi
         sleep 1
      done
   }

   $ source pidwait.sh
   $ sleep 60 &
   $ pgrep sleep
   11539
   $ pidwait 11539
   [1]+  Done                    sleep 60
   bash: kill: (11539) - No such process
   ```

### Terminal multiplexer

### Aliases

1. Create an alias `dc` that resolves to `cd` for when you type it wrongly.

   **Solution**:

   ```bash
   $ vim ~/.bash_aliases
   $ cat ~/.bash_aliases
   alias dc='cd'
   ```

1. Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you’re using ZSH, use `history 1` instead of just `history`.

   ```bash
   $ cat ~/.bash_aliases
   alias dc=cd
   alias sl=ls
   alias gs='git status'
   alias gaa='git add .'
   alias gpsh='git push'
   alias gpshf='git push -f'
   alias gc='git commit'
   alias gca='git commit --amend'
   alias gcan='git commit --amend --no-edit'
   alias gpl='git pull'
   alias gco='git checkout'
   alias gl='git log'
   ```

### Dotfiles

### Remote Machines
