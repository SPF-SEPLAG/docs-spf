# Configurar proxy prodemge nas variáveis de ambiente
Casos em que é necessário: bloqueio github, requisições bloquedas, etc.

![Erro Camg Proxy Github](../../images/tutoriais_camg/error-proxy-github.png)

# Windows
Editar variáveis de ambiente > Nova variável de sistema

http_proxy
http://usuario:senha@proxyint.prodemge.gov.br:8080
https_proxy
http://usuario:senha@proxyint.prodemge.gov.br:8080

# Linux (wsl)
Adicionar ao final do arquivo `.bashrc` na raiz do wsl:

```bash
export HTTP_PROXY="http://usuario:senha@proxyint.prodemge.gov.br:8080"
export HTTPS_PROXY="http://usuario:senha@proxyint.prodemge.gov.br:8080"
export NO_PROXY="localhost,127.0.0.1,::1"

export http_proxy="http://usuario:senha@proxyint.prodemge.gov.br:8080"
export https_proxy="http://usuario:senha@proxyint.prodemge.gov.br:8080"
export no_proxy="localhost,127.0.0.1,::1"
```

> - Usuário é o login ca (`m753270`, por exemplo).
- A senha tem que ser encapsulada em formato encoding URL. 
- Se o usuário/senha do usuário logado não funcionar, pode ser que tentar usar o login do Administrador de rede funcione.


