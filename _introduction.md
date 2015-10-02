
# Introdução #

Introduzida na versão 1.0, a API modo REST permite que você aloque informações à serem criadas, lidas, atualizadas, e deletadas usando o formato JSON.

Introduced in WooCommerce 2.1, the REST API allows store data to be created, read, updated, and deleted using the JSON format.

## Autenticação ##

Existem duas maneiras para se autenticar com a API, dependendo se o website em questão tem suporte à SSL ou não. Lembre-se que o índice indicará se o site suporta SSL ou não. 

There are two aways to authenticate with the API, depending on whether the site supports SSL or not.  Remember that the Index endpoint will indicate if the site supports SSL or not.

### Através de HTTPS ###

Você pode usar [Autenticação Básica via HTTP](http://en.wikipedia.org/wiki/Basic_access_authentication) fornecendo a chave de usuário API como o nome de usuário e o segredo de usuário API como a senha.

You may use [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication) by providing the API Consumer Key as the username and the API Consumer Secret as the password.

> Exemplo de autenticação básica HTTP

> HTTP Basic Auth example

```shell
curl https://www.domum.com.br/api/orders \
    -u consumer_key:consumer_secret
```

Ocasionalmente alguns servidores podem não parear o cabeçalho de autenticação (Se você obter um erro "Chave de usuário está faltando" ao autenticar via SSL, você certamente tem um problema de servidor). Neste caso, você pode fornecer a chave e segredo de usuário como parâmetros da consulta via string.

Occasionally some servers may not properly parse the Authorization header (if you see a "Consumer key is missing" error when authenticating over SSL, you have a server issue). In this case, you may provide the consumer key/secret as query string parameters.

> Exemplo para servidores que não pareiam corretamente o cabeçalho de autenticação:

> Example for servers that not properly parse the Authorization header:

```shell
curl https://www.domum.com.br/api/orders?consumer_key=123&consumer_secret=abc
```

### Através de HTTP ###

Você deve usar a [OAuth 1.0a "one-legged" authentication](http://tools.ietf.org/html/rfc5849) para assegurar que as credenciais da API não sejam interceptadas. Como de praxe, você pode usar qualquer padrão OAuth 1.0a de bibliotecas de sua linguagem de preferência para realizar a autenticação, ou gerar os parâmetros necessários seguindo as instruções.

You must use [OAuth 1.0a "one-legged" authentication](http://tools.ietf.org/html/rfc5849) to ensure API credentials cannot be intercepted. Typically you may use any standard OAuth 1.0a library in your language of choice to handle the authentication, or generate the necessary parameters by following these instructions.

#### Gerando uma assinatura OAuth ####

1) Defina o método de requisição de HTTP:

`GET`

2) Defina sua URL de requerimento básica -- esta é a URI de requisição completa sem os parâmetros de consulta -- e codificação de URL de acordo com RFC 3986:

`http://www.example.com/wc-api/v1/orders`

Quando codificado:

`http%3A%2F%2Fwww.example.com%2Fwc-api%2Fv1%2Forders`

3) Colete e normalize seus parâmetros de consulta. Isto inclui todos os `oauth_*` parâmetros exceto pela assinatura. Parâmetros devem ser normalizados pela codificação da URL de acordo com a RFC 3986 (`rawurlencode` no PHP) e porcentagem(`%`) caracteres deveriam ser codificados duplamente (e.g. `%` se tornaria`%25`.

4) Defina os parâmetros em ordem de bytes (`uksort( $params, 'strcmp' )` no PHP)

5) Junte cada parâmetro com uma assinatura codificada igual à (`%3D`):

`oauth_signature_method%3DHMAC-SHA1`

6) Junte cada parâmetro de chave/valor com um E comercial codificado (`%26`):

`oauth_consumer_key%3Dabc123%26oauth_signature_method%3DHMAC-SHA1`

7) Forme a string para se logar juntando o método HTTP, URI de requisição codificada, e o parâmetro codificado da string com um E comercial não codificado (&):

`GET&http%3A%2F%2Fwww.example.com%2Fwc-api%2Fv1%2Forders&oauth_consumer_key%3Dabc123%26oauth_signature_method%3DHMAC-SHA1`

8) Gere a assinatura usando a chave de string e seu segredo de usuário. 

Se você estiver tendo problemas ao gerar a assinatura correta, você irá ter que rever sua codificação da string da assinatura. A documentação da [Autenticação](https://github.com/woothemes/woocommerce/blob/master/includes/api/class-wc-api-authentication.php#L177) pode ser muito útil para entender como gerar devidamente a assinatura.

#### Dicas OAuth ####

* Os parâmetros OAuth devem ser adicionados como strings na consulta e *não* incluídos no cabeçalho da autorização. Isso se deve porque não existe uma forma confiável de multi-plataformas para pegar as requisições de cabeçalho do WordPress.
* Os parâmetros de requerimento são: `oauth_consumer_key`, `oauth_timestamp`, `oauth_nonce`, `oauth_signature`, e `oauth_signature_method`. `oauth_version` não são obrigatórios e devem ser omitidos.
* HMAC-SHA1 or HMAC-SHA256 são os únicos algoritmos permitidos via hash.
* A OAuth nonce pode ser qualquer string que contenha 32 caracteres (recomendado) sendo única para a chave de consumidor. Leia mais em [generating a nonce](https://dev.twitter.com/discussions/12445) nos fóruns da API do Twitter.
* A OAuth timestamp deve ser única na hora da requisição. A API irá bloquear quaisquer requerimentos que incluam uma timestamp gerada a mais de 15 minutos para prevenir ataques de repetição.
* Você deve usar a URL fornecida pelo índice quando formado a string base usada para a assinatura, justamente porque é esta assinatura que o servidor irá usar. (e.g. se a URL guardada possui o sub-domínio `www`, você deveria usá-la para requerimentos)
* Você pode testar suas assinaturas geradas usando o LinkedIn em [Console de testes OAuth](http://developer.linkedinlabs.com/oauth-test/) -- deixe o token de membro e segredo em branco.
* Twitter possui ótimas instruções sobre como [gerar uma assinatura](https://dev.twitter.com/docs/auth/creating-signature) com OAuth 1.0a, mas lembre-se que os tokens não são utilizados nesta implementação.
* Lembre-se que o corpo da solicitação não está assinado conforme a especificação OAuth, para tal, consulte a [extensão do Google OAuth 1.0](https://oauth.googlecode.com/svn/spec/ext/body_hash/1.0/oauth-bodyhash.html) para obter mais detalhes sobre o motivo.
* Se incluído campos de filtro em sua solicitação, você pode salvar tempo se ordenar seus campos de filtro em ordem alfabética antes de enviar. Muitas bibliotecas OAuth não vão ordenar consultas de subcampos adequadamente, resultando em assinaturas inválidas.