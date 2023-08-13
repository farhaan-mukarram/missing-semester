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

1. Create a folder for your dotfiles and set up version control.

   **Solution:**

   ```bash
   $ mkdir ~/dotfiles
   $ cd ~/dotfiles
   $ git init
   ```

1. Add a configuration for at least one program, e.g. your shell, with some customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).

   **Solution:**

   ```bash
   $ cp ~/.vimrc ~/dotfiles
   $ cp ~/.gitconfig ~/dotfiles
   ```

1. Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls ln -s for each file, or you could use a specialized utility.

   **Solution:**

   ```bash
   $ snap install chezmoi --classic
   $ chezmoi init
   $ chezmoi add ~/.vimrc
   $ chezmoi add ~/.gitconfig
   $ chezmoi cd
   $ git add .
   $ git commit -m "Add vim and gitconfig"
   $ git remote add origin https://github.com/farhaan-mukarram/dotfiles.git
   $ git branch -M main
   $ git push -u origin main
   ```

1. Test your installation script on a fresh virtual machine

   **Solution:**

   ```bash
   $ snap install chezmoi --classic
   $ chezmoi init https://github.com/farhaan-mukarram/dotfiles.git
   $ chezmoi diff
   $ chezmoi apply -v
   ```

1. Migrate all of your current tool configurations to your dotfiles repository

   **Solution:**
   Done in step 3

1. Publish your dotfiles on GitHub.

   **Solution:**
   Done in step 3

### Remote Machines

Install a Linux virtual machine (or use an already existing one) for this exercise. If you are not familiar with virtual machines check out [`this`](https://hibbard.eu/install-ubuntu-virtual-box/) tutorial for installing one.

1. Go to` ~/.ssh/` and check if you have a pair of SSH keys there. If not, generate them with `ssh-keygen -o -a 100 -t ed25519`. It is recommended that you use a password and use `ssh-agent`, more info [`here`](https://www.ssh.com/ssh/agent).
   **Solution**:

   ```bash
   $ ssh-keygen -o -a 100 -t ed25519
   Generating public/private ed25519 key pair.
   Enter file in which to save the key (/home/farhaan/.ssh/id_ed25519):
   Enter passphrase (empty for no passphrase):
   Enter same passphrase again:
   Your identification has been saved in /home/farhaan/.ssh/id_ed25519
   Your public key has been saved in /home/farhaan/.ssh/id_ed25519.pub
   ...
   ```

1. Edit `.ssh/config` to have an entry as follows

   ```bash
   Host vm
      User username_goes_here
      HostName ip_goes_here
      IdentityFile ~/.ssh/id_ed25519
      LocalForward 9999 localhost:8888
   ```

   **Solution**:

   ```bash
   $ vim ~/.ssh/config
   $ cat ~/.ssh/config
   Host vm
      User farhaan
      HostName XXX.XXX.XXX.XXX
      IdentityFile ~/.ssh/id_ed25519
      LocalForward 9999 localhost:8888
   ```

1. Use `ssh-copy-id vm` to copy your ssh key to the server

   **Solution**:

   ```bash
   $ ssh-keygen -o -a 100 -t ed25519
   /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
   /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
   farhaan@XXX.XXX.XXX.XXX's password:
   Number of key(s) added: 1
   Now try logging into the machine, with: "ssh 'vm'"
   and check to make sure that only the key(s) you wanted were added
   ```

1. Start a webserver in your VM by executing `python -m http.server 8888`. Access the VM webserver by navigating to `http://localhost:9999` in your machine

   **Solution**:

   ```bash
   $ ssh vm
   Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)
   ...
   ...
   Enable ESM Apps to receive additional future security updates.
   See https://ubuntu.com/esm or run: sudo pro status

   $ python3 -m http.server 8888
   Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/)
   127.0.0.1 - - [13/Aug/2023 11:25:45] "GET / HTTP/1.1" 200 -
   127.0.0.1 - - [13/Aug/2023 11:25:45] code 404, message File not found
   127.0.0.1 - - [13/Aug/2023 11:25:45] "GET /favicon.ico HTTP/1.1" 404 -
   ```

   Navigating to `localhost:8888` shows a page with the user's directory listing

1. Edit your SSH server config by doing `sudo vim /etc/ssh/sshd_config` and disable password authentication by editing the value of `PasswordAuthentication`. Disable root login by editing the value of `PermitRootLogin`. Restart the `ssh` service with `sudo service sshd restart`. Try sshing in again

   **Solution**:

   ```bash
   $ sudo vim /etc/ssh/sshd_config
   $ sudo service sshd restart
   $ exit
   logout
   Connection to XXX.XXX.XXX.XXX closed.
   $ ssh vm
   Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)
   ...
   ...
   Enable ESM Apps to receive additional future security updates.
   See https://ubuntu.com/esm or run: sudo pro status
   ```

   The value of `PasswordAuthentication` and `PermitRootLogin` in the `sshd_config` has to be changed to `no` for disabling password authentication and root login
