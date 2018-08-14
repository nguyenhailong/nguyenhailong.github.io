---
layout: post
title: "Bash Scripting for Hive"
author: long_nguyen
modified:
excerpt: "Basics about bash scripts, quick guide for running Hive in command lines, and essential tips with example for running Hive script on servers."
tags: []
---
This post will provide basics about bash scripts, quick guide for running Hive in command lines, and essential tips with example for running Hive script on servers with input parameters and loop.

## Bash Scripting Basic
### Builtin shell variables

```
$0             #Name of this shell script itself.
$1             #Value of first command line parameter (similarly $2, $3, etc)
$#             #In a shell script, the number of command line parameters.
$*             #All of the command line parameters.
$-             #Options given to the shell.
$?             #Return the exit status of the last command.
$$             #Process id of script (really id of the shell running the script)
```

### IF and CASE condition statements

```
if [ "$VAR1" = "$VAR2" ]; then
	echo "expression evaluated as true"
else
	echo "expression evaluated as false"
fi
```

```
case "$C" in
"1")
	do_this()
	;;
"2" | "3")
	do_what_you_are_supposed_to_do()
	;;
*)    #otherwise
	do_nothing()
	;;
esac
```

### FOR loop

```
for i in 1 2 3 4 5    # can also be written for i in {1..5} or {start..end..increment}
do
	echo "Welcome $i times"
done
```

### WHILE loop

```
while [ condition ]
do
	command
done
```

## Hive CLI (Command Line Interface)

### Run Query
```
hive -e 'select a.col from tab1 a'
```

### Run Non-Interactive Script	
```
hive -f script.sql
```

### Run script inside shell
```
source file_name
```

### Setting Configuration Property for current Hive 
```
hive --hiveconf year_mm=201808 -f script.sql
```

### Run a Hive Script Even after You Logout
Nohup is very helpful when you have to execute a shell-script or command that take a long time to finish. In that case, you don’t want to be connected to the shell and waiting for the command to complete. Instead, execute it with nohup, exit the shell and continue with your other work.


```
nohup hive --hiveconf year_mm=201808 -f script.sql 
```

By default, the standard output will be redirected to *nohup.out* file in the current directory. And the standard error will be redirected to stdout, thus it will also go to nohup.out. So, your nohup.out will contain both standard output and error messages from the script that you’ve executed using nohup command.

## Bash Scripting for Hive
Suppose that your Hive script needs to loop over several months (or years) and requires two parameters, year_mm and pre_year_mm. The Hive script is named as my_hive_script.hql

Below bash script can help to execute your Hive script over the months and notify you by sending an email when the job is done.

```
EMAIL="nlong@gmail.com"

echo "CMI: Finding common accounts within same months of two years ...."

yyyymm=(201808 201807 201806 201805 201804 201803 201802)
prev_yyyymm=(201807 201806 201805 201804 201803 201802 201801)
for((i=0;i<${#yyyymm[@]};i++))
do
  echo "Running for month: ${yyyymm[i]}, previous month: ${prev_yyyymm[i]}"
  hive --hiveconf year_mm=${yyyymm[i]} --hiveconf pre_year_mm=${prev_yyyymm[i]} -f my_hive_script.hql
done
    
mail -s "Script $0 completed." $EMAIL 
exit
```

Save the above example as *my_hive_script.sh*. Remember to use nohup when executing the script
```
nohup ./my_hive_script.sh
```

## Bugs when switch from Window to Unix

Suppose you are using a Window system and want to execute your scripts on your Unix servers. 
You may have some bugs of mismatch decodes between the two systesms (it took me a few hours to figure out the bug.)
To fix it, you can use Notepad++ to convert the format as follow:
- Notepad++ > Edit > Unix (LF)

Then, save it and copy to your Unix server before running commands.