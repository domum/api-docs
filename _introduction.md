
# Introdução #

Introduzida na versão 1.9, a API REST permite que você aloque informações à serem criadas, lidas, atualizadas, e deletadas usando o formato JSON.

Introduced in 1.9 version, the REST API allows store data to be created, read, updated, and deleted using the JSON format.

## Autenticação ##

Existem duas maneiras para se autenticar na API, dependendo se o estiver disponível o suporte à SSL ou não. Lembre-se que o índice indicará se o sistema já está suportando SSL ou não. 

There are two aways to authenticate with the API, depending on whether the system supports SSL or not. Remember that the Index endpoint will indicate if the system already supports SSL or not.

### Através de HTTPS ###

Você pode usar [Autenticação Básica via HTTP](http://en.wikipedia.org/wiki/Basic_access_authentication) fornecendo a chave API de usuário como usuário e o segredo API de usuário como senha.

You may use [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication) by providing the API User Key as the username and the API User Secret as the password.

> Exemplo de autenticação básica HTTP

> HTTP Basic Auth example

```shell
curl https://api.domum.com.br/clientes \
    -u chave_usuario:segredo_usuario
```

Ocasionalmente alguns servidores podem não parear o cabeçalho de autenticação (Se você obter um erro "Chave de usuário faltando" ao autenticar via SSL, você certamente tem um problema de servidor). Neste caso, você pode fornecer a chave e segredo de usuário como parâmetros da consulta via string.

Occasionally some servers may not properly parse the Authorization header (if you see a "Consumer key is missing" error when authenticating over SSL, you have a server issue). In this case, you may provide the consumer key/secret as query string parameters.

> Exemplo para servidores que não pareiam corretamente o cabeçalho de autenticação:

> Example for servers that not properly parse the Authorization header:

```shell
curl https://api.domum.com.br/clientes?chave_usuario=123&segredo_usuario=abc
```

### Através de HTTP ###

Você deve usar a [autenticação OAuth 1.0a "one-legged"](http://tools.ietf.org/html/rfc5849) para assegurar que as credenciais da API não sejam interceptadas. Como de praxe, você pode usar qualquer biblioteca padrão OAuth 1.0a da linguagem de sua preferência para realizar a autenticação, ou gerar os parâmetros necessários seguindo as instruções.

You must use [OAuth 1.0a "one-legged" authentication](http://tools.ietf.org/html/rfc5849) to ensure API credentials cannot be intercepted. Typically you may use any standard OAuth 1.0a library in your language of choice to handle the authentication, or generate the necessary parameters by following these instructions.

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

Se você estiver tendo problemas ao gerar a assinatura correta, você irá ter que rever sua codificação da string da assinatura. A documentação da [Autenticação](https://github.com/woothemes/woocommerce/blob/master/includes/api/class-wc-api-authentication.php#L177) pode ser muito útil para entender como gerar devidamente a assinatura.

#### Dicas OAuth ####

* The OAuth parameters must be added as query string parameters and *not* included in the Authorization header. This is because there is no reliable cross-platform way to get the raw request headers in WordPress.
* The require parameters are: `oauth_consumer_key`, `oauth_timestamp`, `oauth_nonce`, `oauth_signature`, and `oauth_signature_method`. `oauth_version` is not required and must be omitted.
* HMAC-SHA1 or HMAC-SHA256 are the only accepted hash algorithms.
* The OAuth nonce can be any randomly generated 32 character (recommended) string that is unique to the consumer key. Read more suggestions on [generating a nonce](https://dev.twitter.com/discussions/12445) on the Twitter API forums.
* The OAuth timestamp should be the unix timestamp at the time of the request. The API will deny any requests that include a timestamp that is outside of a 15 minute window to prevent replay attacks.
* You must use the store URL provided by the index when forming the base string used for the signature, as this is what the server will use. (e.g. if the store URL includes a `www` sub-domain, you should use it for requests)
* You may test your generated signature using LinkedIn's [OAuth test console](http://developer.linkedinlabs.com/oauth-test/) -- leave the member token/secret blank.
* Twitter has great instructions on [generating a signature](https://dev.twitter.com/docs/auth/creating-signature) with OAuth 1.0a, but remember tokens are not used with this implementation.
* Note that the request body is *not* signed as per the OAuth spec, see [Google's OAuth 1.0 extension](https://oauth.googlecode.com/svn/spec/ext/body_hash/1.0/oauth-bodyhash.html) for details on why.
* If including filter fields in your request, it saves a lot of trouble if you can order your filter fields alphabetically before submitting. Many Oauth libraries won't order subquery fields properly, resulting in invalid signatures.
