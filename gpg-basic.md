# gpg 基础

gpg 是 GNU Privacy Guard项目的一部分，实现了 OpenPGP 标准。

### 基本步骤

#### 生成新的密钥对

```shell
gpg --gen-key
```

> 这个过程是交互式的，根据提示操作即可。生成的密钥对中，私钥用来解密或者签名，公钥用来分发给其他人，其他人就可以使用其加密信息。

#### 查看已有的密钥信息

```shell
# show public keys
gpg --list-keys

# show private keys
gpg --list-secret-keys
```

#### 导出公钥

```shell
# binary format
gpg --export -u UID -o name.pub

# ascii format
gpg --armor --export UID -o name.pub.txt
```

> 可以将 name.pub 文件分发给别人，以便别人使用其加密信息或者验证你发布的文件的签名

#### 导出私钥

```shell
# binary format
gpg --export-secret-keys -o name.pub

# ascii format
gpg --armor --export-secret-keys -o name.pub.txt
```

#### 导入其它人的公钥

```shell
gpg --import file.pub
```

#### 加密与解密

```shell
# encrypt
gpg -e -r UID -o file.enc file

# decrypt
gpg -d -o file file.enc
```

#### 签名与验证

签名

```shell
gpg -s file

# dettached sign. a file.sig is created
gpg -b file
```

验证

```shell
# for dettached sign
gpg --verify file.sig file

# for normal sign
gpg --verify file.gpg
```

> 验证可能会产生一个警告：
>
> WARNING: This key is not certified with a trusted signature!
>
> 此时，如果信任公钥，可以使用自己的密钥对其进行签名
>
> gpg --sign-key UID
>
> 使用签名之后的公钥去验证其它签名就不会产生警告了

#### 杂项

如果gpg生成密钥时间很久，或者提示没有足够的随机数据，需要使用rng工具

```shell
sudo rngd -r /dev/urandom
```

