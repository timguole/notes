# nc remote shell

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

