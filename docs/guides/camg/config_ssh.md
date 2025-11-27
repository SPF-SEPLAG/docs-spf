# Configurar uso de chaves SSH na CAMG
É preciso mudar a porta utilizada quando tentando usar chaves SSH na CAMG.

# Linux (wsl)
- Na raiz do wsl, na pasta `.ssh`, onde estarão armazenadas as chaves SSH, criar arquivo `config` (é só `config`, não `config.txt`).

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```