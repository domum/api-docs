
# Introdução #

Introduzida na versão 1.9, a API REST permite que você aloque informações à serem criadas, lidas, atualizadas, e deletadas usando o formato JSON.

## Autenticação ##

Existem duas maneiras para se autenticar na API, dependendo se o estiver disponível o suporte à SSL ou não. Lembre-se que o índice indicará se o sistema já está suportando SSL ou não. 

### Através de HTTPS ###

Você pode usar [Autenticação Básica via HTTP](http://en.wikipedia.org/wiki/Basic_access_authentication) fornecendo a chave API de usuário como usuário e o segredo API de usuário como senha.

> Exemplo de autenticação básica HTTP

```shell
curl https://api.domum.com.br/clientes \
    -u chave_usuario:segredo_usuario
```

Ocasionalmente alguns servidores podem não parear o cabeçalho de autenticação (Se você obter um erro "Chave de usuário faltando" ao autenticar via SSL, você certamente tem um problema de servidor). Neste caso, você pode fornecer a chave e segredo de usuário como parâmetros da consulta via string.

> Exemplo para servidores que não pareiam corretamente o cabeçalho de autenticação:

```shell
curl https://api.domum.com.br/clientes?chave_usuario=123&segredo_usuario=abc
```

### Através de HTTP ###

Você deve usar a [autenticação OAuth 1.0a "one-legged"](http://tools.ietf.org/html/rfc5849) para assegurar que as credenciais da API não sejam interceptadas. Como de praxe, você pode usar qualquer biblioteca padrão OAuth 1.0a da linguagem de sua preferência para realizar a autenticação, ou gerar os parâmetros necessários seguindo as instruções.

#### Gerando uma assinatura OAuth ####

1) Defina o método de requisição de HTTP:

`GET`

2) Defina sua URL de requerimento básica -- esta é a URI de requisição completa sem os parâmetros de consulta -- e codificação de URL de acordo com RFC 3986:

`http://api.domum.com.br/v1/clientes`

Quando codificado:

`http%3A%2F%2Fapi.domum.com.br%2Fv1%2Fclientes`

3) Colete e normalize seus parâmetros de consulta. Isto inclui todos os parâmetros `oauth_*` exceto pela assinatura. Parâmetros devem ser normalizados pela codificação da URL de acordo com a RFC 3986 (`rawurlencode` no PHP) e caracter `%` (porcentagem) deve ser codificados duplamente (ex.: `%` se tornaria `%25`).

4) Defina os parâmetros em ordem de bytes (`uksort( $params, 'strcmp' )` no PHP)

5) Junte cada parâmetro com uma assinatura codificada igual à (`%3D`):

`oauth_signature_method%3DHMAC-SHA1`

6) Junte cada parâmetro de chave/valor com um `&` (e comercial) codificado (`%26`):

`oauth_consumer_key%3Dabc123%26oauth_signature_method%3DHMAC-SHA1`

7) Para se logar forme uma string juntando o método HTTP, URI de requisição codificada, e o parâmetro codificado da string com um `&` (e comercial) não codificado (`&`):

`GET&http%3A%2F%2Fapi.domum.com.br%2Fv1%2Fclientes&oauth_consumer_key%3Dabc123%26oauth_signature_method%3DHMAC-SHA1`

8) Gere a assinatura usando essa chave string e seu segredo de usuário. 

Se você estiver tendo problemas ao gerar a assinatura correta, você irá ter que rever sua codificação da string da assinatura.

#### Dicas OAuth ####

* Os parâmetros OAuth devem ser adicionados como strings na consulta e *não* incluídos no cabeçalho da autorização. Isso se deve porque não existe uma forma multi-plataforma confiável para pegar as requisições de cabeçalho.
* Os parâmetros de requerimento são: `oauth_consumer_key`, `oauth_timestamp`, `oauth_nonce`, `oauth_signature`, e `oauth_signature_method`. `oauth_version` não são obrigatórios e devem ser omitidos.
* HMAC-SHA1 or HMAC-SHA256 são os únicos algoritmos permitidos para hash.
* A OAuth nonce pode ser qualquer string que contenha 32 caracteres (recomendado) sendo única para a chave de consumidor. Leia mais em [gerando um nonce (inglês)](https://dev.twitter.com/discussions/12445) nos fóruns da API do Twitter.
* A OAuth timestamp deve ser única, a hora da requisição. A API irá bloquear quaisquer requerimentos que incluam uma timestamp gerada a mais de 15 minutos para prevenir ataques de repetição.
* Você deve usar a URL fornecida pelo índice quando formar a string base usada para a assinatura, justamente porque é esta assinatura que o servidor irá usar.
* Você pode testar suas assinaturas geradas usando o LinkedIn em [Console de testes OAuth](http://developer.linkedinlabs.com/oauth-test/) -- deixe o token de membro e segredo em branco.
* Twitter possui ótimas instruções sobre como [gerar uma assinatura (inglês)](https://dev.twitter.com/docs/auth/creating-signature) com OAuth 1.0a, mas lembre-se que os tokens não são utilizados nesta implementação.
* Lembre-se que o corpo da solicitação não está assinado conforme a especificação OAuth, para tal, consulte a [extensão do Google OAuth 1.0 (inglês)](https://oauth.googlecode.com/svn/spec/ext/body_hash/1.0/oauth-bodyhash.html) para obter mais detalhes sobre o motivo.
* Se incluído campos de filtro em sua solicitação, recomensa-se ordenar esses campos de filtro em ordem alfabética antes de enviar. Muitas bibliotecas OAuth não vão ordenar consultas de subcampos adequadamente, resultando em assinaturas inválidas.
