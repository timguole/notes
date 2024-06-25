# ssh-agent

## Config

Add ssh-agent starting code in ~/.profile or ~/.bash_profile. This makes ssh-agent to start at login. If the default system temporary directory is not accessible, use '-a' option to point the socket to a different location.

```shell
eval $(ssh-agent);
# or
eval $(ssh-agent -a $PATH_TO_SOCK);
```

Add ssh-agent stopping code in ~/.logout or ~/.bash_logout.

```shell
eval $(ssh-agent -k)

# if the above code does not work well, kill agent by env
if [[ -n "$SSH_AGENT_PID" ]]; then
        kill -s 9 $SSH_AGENT_PID;
fi
if [[ -S "$SSH_AUTH_SOCK" ]]; then
        rm -f $SSH_AUTH_SOCK;
fi
```

## Usage

Generate and encrypt a ssh key

```shell
ssh-keygen # provides a strong password instead of an empty one
```

Add ssh key in session

```shell
ssh-add PATH-TO-PRIVIATE-KEY-FILE
```

