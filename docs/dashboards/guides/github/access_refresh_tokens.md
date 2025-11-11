## O que são Access Token e Refresh Token?  

Segundo [Auth0](https://auth0.com/blog/pt-refresh-tokens-what-are-they-and-when-to-use-them/), quando um usuário faz login, o servidor de autorização emite um token de acesso (Access Token), que é um artefato que as aplicações cliente podem usar para fazer chamadas seguras para um servidor de API. Quando uma aplicação cliente precisa acessar recursos protegidos em um servidor em nome de um usuário, o token de acesso permite que o cliente sinalize ao servidor que recebeu autorização do usuário para executar determinadas tarefas ou acessar determinados recursos. 

É fundamental ter estratégias de segurança que minimizem o risco de comprometer os tokens de acesso. Um método de mitigação é criar tokens de acesso com vida útil curta: eles são válidos apenas por um curto período definido em termos de horas ou dias. 

Depois que expiram, as aplicações cliente podem usar um Refresh Token para "atualizar" o token de acesso. Ou seja, um Refresh Token é um artefato de credencial que permite que uma aplicação cliente obtenha novos tokens de acesso sem precisar solicitar que o usuário faça login novamente. 
