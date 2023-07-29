# Lecture 4: Data Wrangling

## Exercises

1. Take this [short interactive regex tutorial](https://regexone.com/).

   **Solution**:

   Lesson 1: `pattern: abc`

   Lesson 1 &frac12;: `pattern: 123`

   Lesson 2: `pattern: ...\.`

   Lesson 3: `pattern: [cmf]an`

   Lesson 4: `pattern: [^b]og`

   Lesson 5: `pattern: [A-C][n-p][a-c]`

   Lesson 6: `pattern: waz{3,5}up`

   Lesson 7: `pattern: aa+b*c+`

   Lesson 8: `pattern: [0-9]+ file[s]? found\?`

   Lesson 9: `pattern: [0-9]\.\s+abc`

   Lesson 10: `pattern: ^Mission: successful$`

   Lesson 11: `pattern: (file.+)\.pdf$`

   Lesson 12: `pattern: (\w+ ([0-9]+)$)`

   Lesson 13: `pattern: ([0-9]+)x([0-9]+)`

   Lesson 14: `pattern: I love (cats|dogs)`

   Lesson 15: `pattern: (\w|\s|\.|%|[&$#*@!])+\.$`

1. Find the number of words (in `/usr/share/dict/words`) that contain at least three `a`s and don’t have a `'s` ending. What are the three most common last two letters of those words? sed’s `y` command, or the `tr` program, may help you with case insensitivity. How many of those two-letter combinations are there? And for a challenge: which combinations do not occur?

   **Solution**:

   ```bash
   $ cat /usr/share/dict/words | grep -E "(.*a){3}" | grep -Ev "'s"| sed -E 's/.+(\w{2}$)/\1/' | sort | uniq -c | sort -rn | head -n 3
        85 an
        57 ns
        52 as
   ```

1. To do in-place substitution it is quite tempting to do something like sed s/REGEX/SUBSTITUTION/ input.txt > input.txt. However this is a bad idea, why? Is this particular to sed? Use man sed to find out how to accomplish this.

   **Solution**:
   The `-i` flag can be used for in-place substitution

1. Find your average, median, and max system boot time over the last ten boots. Use `journalctl` on Linux and `log show` on macOS, and look for log timestamps near the beginning and end of each boot. On Linux, they may look something like:

   ```bash
   Logs begin at ...
   ```

   and

   ```bash
   systemd[577]: Startup finished in ...
   ```

   On macOS, [look for](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):

   ```bash
   === system boot:
   ```

   and

   ```bash
   Previous shutdown cause: 5
   ```

   **Solution**:

   ```bash
   $ journalctl | grep -E "systemd.* Startup finished in .+userspace.+= (.+)s" | sed -E 's/^.+systemd.* Startup finished in .+userspace.+= (.+)s\./\1/' | sed -e 's/min//' | tail -n 10 | awk 'NF==1; NF==2{seconds=($1*60);seconds=seconds+$2;print seconds}' | st --transpose-output --mean --median --max --N
   N       10
   median  49.2405
   max     74.682
   mean    46.7455
   ```

1. Look for boot messages that are not shared between your past three reboots (see journalctl’s -b flag). Break this task down into multiple steps. First, find a way to get just the logs from the past three boots. There may be an applicable flag on the tool you use to extract the boot logs, or you can use sed '0,/STRING/d' to remove all lines previous to one that matches STRING. Next, remove any parts of the line that always varies (like the timestamp). Then, de-duplicate the input lines and keep a count of each one (uniq is your friend). And finally, eliminate any line whose count is 3 (since it was shared among all the boots).

   **Solution:**

   ```bash
   $ journalctl -b -3 | grep -E '[A-Za-z]+ [0-9]+ [0-9]+:[0-9]+:[0-9]+ ' | sed -E 's/[A-Za-z]+ [0-9]+ [0-9]+:[0-9]+:[0-9]+//' | sed -E 's/(^.*)(ubuntu.*)\[[0-9]+\](.*)$/\1\2\3/' | sort | uniq -c | sort -n | grep -Ev "^[[:space:]]+3.*$" | less
      1  ubuntu accounts-daemon: started daemon version 22.07.5
      1  ubuntu acpid: 8 rules loaded
      1  ubuntu acpid: starting up with netlink and the input layer
      1  ubuntu acpid: waiting for events: event logging is off
      .
      .
      .
      62  ubuntu gnome-shell: == Stack trace for context 0x564945393490 ==
      244  ubuntu rtkit-daemon: Supervising 7 threads of 4 processes of 1 users.
   ```

1. Find an online data set like [this one](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [this one](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1), or maybe one [from here](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/). Fetch it using `curl` and extract out just two columns of numerical data. If you’re fetching HTML data, [`pup`](https://github.com/EricChiang/pup) might be helpful. For JSON data, try [`jq`](https://stedolan.github.io/jq/). Find the min and max of one column in a single command, and the difference of the sum of each column in another.

   **Solution:**

   ```bash
   $ curl -s "https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1" > data.txt && cat data.txt | pup "table td.group3 text{}" | \grep "\S" > crime_rate.txt && cat data.txt | pup "table td.group1 text{}" | \grep "\S" > population.txt && paste population.txt crime_rate.txt > merged.txt
   $ cat merged.txt | awk '{print $1}' | sed -E 's/,//g' | st --min --max --transpose-output
   min	2.67784e+08
   max	3.23128e+08
   $ cat merged.txt | sed -E 's/,//g' | awk 'BEGIN {col_1_sum = 0; col_2_sum = 0};NF==2 {col_1_sum += $1; col_2_sum += $2}; END {print col_1_sum, col_2_sum}' | awk '{print $1-$2}'
   5.97269e+09
   ```
