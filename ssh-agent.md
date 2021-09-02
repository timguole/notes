# ssh-agent

## Config

Start ssh-agent in .profile/.bash_profile

```shell
eval $(ssh-agent);
```

Stop ssh-agent in .logout/.bash_logout

```shell
eval $(ssh-agent -k)
```

## Usage

Generate and encrypt a ssh key

```shell
ssh-keygen
```

Add ssh key in session

```shell
ssh-add PATH-TO-PRIVIATE-KEY-FILE
```

