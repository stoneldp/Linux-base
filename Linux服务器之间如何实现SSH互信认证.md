## SSH互信认证
### 逻辑：在两台机器之间如何实现互信认证？
```markdown
- 第一步在机器A中生成公私钥；
- 第二步需要把公钥拷贝到机器B；
- 这样机器A——>机器B就可以免密登录及传文件了。
- 如要实现互信，就只需机器B生成公私钥，将公钥考到机器A即可实现。
```
### 具体实现过程：
- 1、在机器A生成公私钥
```linux
durgin@macro-008:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/durgin/.ssh/id_rsa): 
/home/durgin/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/durgin/.ssh/id_rsa
Your public key has been saved in /home/durgin/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:nTJmC6Tnz6/p/U2PV8uQvGN/kNlaPTflkpIgqIszwvI durgin@macro-008
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|      ..         |
|     o. ....    .|
|    ..o S.o....*o|
|    .o + +  o+*oB|
|.  . .. .    .==*|
|o.+ .  o o   *.=o|
|.oEo   .*oo.o =oo|
+----[SHA256]-----+

```

    - Your identification has been saved in /home/durgin/.ssh/**id_rsa**私钥 
    - Your public key has been saved in /home/durgin/.ssh/**id_rsa.pub**公钥

- 2、需要把机器A公钥拷贝到机器B
```linux
ssh-copy-id user@remote-host

输入密码：

- 注：公钥复制到机器B的~/.ssh/authorized_keys文件中。
durgin@macro-008:~/.ssh$ pwd
/home/durgin/.ssh
durgin@macro-008:~/.ssh$ cat authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC6iDPXXX.....F
```

### 脚本实现：
- 在SSH中设置互信认证（也称为密钥对认证）通常涉及几个步骤，包括生成SSH密钥对、将公钥复制到远程服务器以及配置SSH客户端。下面是一个简单的Shell脚本示例，用于自动执行这些操作。此脚本假设你想要从本地机器向远程服务器设置无密码登录。
请注意，出于安全考虑，在使用此类脚本之前，请确保理解其功能，并仅在可信的环境中应用。
```shell
#!/bin/bash

# 变量定义
REMOTE_USER="your_remote_username"
REMOTE_HOST="your_remote_host"
LOCAL_SSH_DIR="${HOME}/.ssh"
IDENTITY_FILE="${LOCAL_SSH_DIR}/id_rsa"

# 检查是否已存在密钥对，如果没有则创建
if [ ! -f "${IDENTITY_FILE}" ]; then
    echo "Generating SSH key pair..."
    ssh-keygen -t rsa -b 4096 -f "${IDENTITY_FILE}" -N ""
fi

# 获取公钥内容
PUBLIC_KEY=$(cat "${IDENTITY_FILE}.pub")

# 将公钥发送到远程服务器
echo "Copying public key to remote server..."
ssh-copy-id -i "${IDENTITY_FILE}.pub" "${REMOTE_USER}@${REMOTE_HOST}"

# 验证连接
echo "Verifying the connection..."
ssh -o BatchMode=yes "${REMOTE_USER}@${REMOTE_HOST}" 'echo Success'

# 检查连接是否成功
if [ $? -eq 0 ]; then
    echo "SSH trust setup was successful."
else
    echo "There was an issue setting up the SSH trust. Please check the error messages."
fi
```
- 请按照以下步骤使用此脚本：
  - 将 your_remote_username 和 your_remote_host 替换为实际的远程用户名和主机地址。
  - 保存此脚本为一个文件，例如 setup_ssh_trust.sh。
  - 给予该脚本执行权限：在终端运行 chmod +x setup_ssh_trust.sh。 
  - 运行脚本：./setup_ssh_trust.sh。
- 重要提示： 
  - 确保你了解并信任执行此脚本的环境，因为它涉及到在远程服务器上设置认证信息。 
  - 如果你的SSH服务端配置要求特定的密钥类型或设置了其他限制，请相应调整脚本中的 ssh-keygen 命令。 
  - 使用 -N "" 参数在生成密钥时跳过了密码设置，这在某些场景下可能不符合安全性要求。根据需要，你可以移除此参数以设置密钥密码。