# Bash Snippets

Change to  script directory

```shell
cd $(dirname $0);
```

Get script directory

```shell
BASE_DIR=$(cd $(dirname $0); pwd);
```

Append new value to an array

```shell
a=(a b c);
a+=(d);
a+=(e f);
```

Wait for multiple jobs

```shell
job_pids=();
for j in $job_list; do
	$j &
	job_pids+=($!);
done

for p in ${job_pids[@]}; do
	wait -n $p;
done
```

