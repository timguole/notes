# diff & patch

### diff two files

```shell
# -a: text mode
# -u: unified output format
diff -au old-file new-file > file.patch
```

### diff two directories

```shell
# -r: recursively diff
diff -rau old-dir new-dir > dir.patch
```

### patch a file

```shell
patch old-file patch-file
```

### patch a directory

```shell
mv patch-file upper-dir # the upper-dir contains old-dir
cd upper-dir
patch -p0 -i patch-file
```

