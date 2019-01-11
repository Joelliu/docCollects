## ssh connect to a remote server

First generate keyfiles.


```shell
ssh-keygen -t rsa -b 2048
```

 Give the key file a proper name,  and choose a secret passphrase. And you will get finally the following keyfiles: 
 - ~/.ssh/bandwagong
 - ~/.ssh/bandwagong.pub


 ```shell
 eval $(ssh-agent -s)
 ssh-add ~/.ssh/bandwagong
 ```

 Using ssh-agent as 'agent' for the future connections, so that you can further connect to server without inputting your passphrase anymore.

 > ssh-agent support also 'agent forwording', which means you can firstly log into one remote server, and ssh to another server directly. Otherwise, you will need to provide(type) your password to the middle server, which is unsafe.


 ```shell
 ssh-copy-id -i ~/.ssh/bandwagong.pub user@hostname -p port
 ```

 Copy the keys to remote server. (you will be asked for the user password, for the first connection)


Then you can log into your server with just the following command (without password anymore!):

```shell
ssh user@hostname -p port
```

### Remove keys:

```shell
## local host
ssh-add -D # remove all identities
ssh-keygen -R hostname_of_remote_server

## remote server
rm ~/.ssh/authorized_keys
```