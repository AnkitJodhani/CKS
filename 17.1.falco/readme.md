# Detect runtime activity
- install falco

```bash

# Falco files are located at
cd /etc/falco/

# What are the files present here
-rw-r--r-- 1 root root  1096 Mar  9 15:32 falco_rules.local.yaml    # ADD YOUR CUSTOM RULE HERE
-rw-r--r-- 1 root root 63723 Mar  9 15:18 falco_rules.yaml          # DEFAULT RULES ARE HERE
drwxr-xr-x 2 root root  4096 Jan 28 09:34 config.d
drwxr-xr-x 2 root root  4096 Jan 28 09:34 rules.d
-rw-r--r-- 1 root root 58628 Jan 28 09:06 falco.yaml                # FALCO DEFAULT CONFIG

# To see list of files are applied - and the order of files
cat falco.yaml | grep -i "rules_files" -A5

# To see where the output is going
cat falco.yaml | grep -i "file_output" -A5

# if you made any mistake while writing the rules
# hit below command to debug

```