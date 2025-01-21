# nc basics

## run remote shell

Use `nc` to provide a remote shell.

#### On remote server side

``` shell
mkfifo /tmp/nc-shell.fifo
cat /tmp/nc-shell.fifo | nc -l server-ip server-port | bash -li > /tmp/nc-shell.fifo 2>&1
```

#### On local client side

``` shell
nc -N server-ip server-port
```



## test port connectivity

```shell
# -z: zero-I/O mode. establish connection and exit.
# -w: wait at most N seconds before connected, then timeout
nc -z -w 3 IP PORT
```

