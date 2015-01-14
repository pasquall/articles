# Tutorial API para dispositivos mobile

O desenvolvimento de API tem aumentado cada vez mais devido a demanda de desenvolvimento de aplicativos mobile.

Então aqui vai um turorial que abrange diversos aspectos sobre desenvolvimento desse tipo de sistema, desde os aspectos relacionados a segurança até documentação.

## Processamento e número de requests

É importante ter em mente que dispositivos mobile, apesar de atualmente possuírem uma grande capacidade de processamento, devem processar o mínimo possível de informações.

Por que?
Porque um servidor pode processar muito mais rapidamente que um mobile, e simplesmente porque a maior barreira de aplicativos é o custo envolvido em requisições web, ou seja, quanto menos requisições você fazer, melhor!

### Exemplo

Observem o aplicativo do Gmail para iPhone e Android.
Suponha que existam 20 emails no seu inbox, e que dos 20 emails você deseja excluir 10.

O aplicativo até permite excluir 1 por 1, mas note que o checkbox está em evidência!
Ou seja, sua API deverá suportar deleção de emails em massa, tudo em uma única request.

Além disso, perceba que logo depois que os emails são deletados, os mesmos são instantaneamente escondidos da lista.
É obvio que o Gmail não fez mais uma requisição pra API para obter a nova lista de emails que deverão aparecer no inbox.

Resumindo: quando criar suas API endpoints, sempre considere o coletivo, ou seja, remoção e atualização em massa. O que importa é diminuir número de requests.

## Versionamento

Imagina o seguinte...

A sua API é accesível via http://api.super_sexy_tapes.com .

Aí você lança na AppleStore e no GooglePlay o app "SexTape App", que se conecta na API acima.

100000 de pessoas solteiras ou sexualmente mal-resolvidas instalaram o SexTape App.

Aí você descobre um bug, você conserta, faz o deploy pra produção, mas há um pequeno detalhe, o `response body` de um dos endpoints agora está diferente.

__!!! BANG !!!__
Você acabou de fazer a vida de 100000 ainda mais difícil porque o SexeTape App que eles tem instalados em seus mobiles não estão preparados para fazer o parse do novo `response body`, conclusão: __CRASH!!!__

O que você faz?
Lança uma atualização, publica na AppleStore e no GooglePlay um novo SexTape App que suporta a alteração que você fez na API.

Mas, o que acontece com as pessoas que não quiserem atualizar o aplicativo?
Eles nunca consiguirão usar o SexTape App.

O que fazer pra evitar essa confusão?
Um sistema de versionamento descente.

Crie sua API dessa forma: http://api.super_sexy_tapes.com/v1
E lance o SexTape App preparado pra API v1.

Se descobriu um bug, arrume e faça o deploy para: http://api.super_sexy_tapes.com/v1.1 (por exemplo)
E lance o SexTape App preparado pra API v1.1.

A idéia é, manter a API e o APP sincronizados, sempre!

Segue um exemplo de uma endpoint do [Twitter](https://dev.twitter.com/rest/reference/get/statuses/mentions_timeline).

## Paginação

Imagina se o Rafinha Bastos (que possui uma porrada de seguidores) abre o Twiiter, e o endpoint do Twitter retorna uma `response body` com 1000000 seguidores...

Cara, não acho que não há nada no arsenal mundial que consiga fazer um parse de tantos dados assim em um mobile.

Então, certifique-se que sua API possua paginação pra não fritar o iPhone do usuário.

### Exemplo:

Endpoint: http://api.super_sexy_tapes.com/v1/videos
Response Body:

```json
{
  videos: [
    ...
  ],
  pagination: {
    limit: 20,
    offset: 0,
    count: 20,
    total: 25
  }
}
```

Endpoint: http://api.super_sexy_tapes.com/v1/videos?page=1
Response Body:

```json
{
  videos: [
    ...
  ],
  pagination: {
    limit: 20,
    offset: 1,
    count: 5,
    total: 25
  }
}
```

Geralmente os aplicativos vão incrementando o parâmetro `page` com aquela técnica do Infinite Scroll.

## Response Body

É importante que seja em JSON ou XML, por favor, não me retorne um clear text.

Mas o fato é que existem diversas maneiras e sugestões sobre o formato do body.
A que eu particularmente mais estou acostumado é:

* uma key contendo os resources
* uma key contendo paginação
* uma key contendo informações extras, como quota de requests por minuto, versão da API, etc...

__!!! IMPORTANTE !!!__

Lembre-se que de manter o número de requests sempre baixo, então imagina o seguinte.

Endpoint: http://api.super_sexy_tapes.com/v1/videos?page=1
Response Body:

```json
{
  videos: [
    {
      title: "Minha vizinha é demais",
      category: 123
    }
  ],
  pagination: {
    limit: 20,
    offset: 1,
    count: 5,
    total: 25
  }
}
```

Agora suponha que você quer saber o que é a categoria de ID 123, então o que seu mobile faz? Outra requisição para obter o nome da categoria.

Endpoint: http://api.super_sexy_tapes.com/v1/categories/123
Response Body:

```json
{
  category: {
    id: 123,
    title: "Vizinhas Gostosas"
  }
}
```

Velho, isso não é necessário, você pode aninhar conteúdos no `response body`, exemplo:

```json
{
  videos: [
    {
      title: "Minha vizinha é demais",
      category: {
        id: 123
        title: "Vizinhas Gostosas"
      }
    }
  ],
  pagination: {
    limit: 20,
    offset: 1,
    count: 5,
    total: 25
  }
}
```

Em uma tacada só você obteve o vídeo e a categoria.

__!!! IMPORTANTE 2 !!!__

A não ser que seja realmente necessário, não legal aninhar muitas coisas em uma request só, ela começa a ficar genérica, tão genérica que eu duvído que você vai conseguir enfiar tanta informação numa tela de Galaxy S5.

Response Bodies muito gordo, consome tempo de requisição e de processamento durante o parse, tudo depende da situação, então você terá que ter uma balancinha pra pesar o ponto certo de enfiar mais coisas na mesma request ou não.

## Request / REST API

Eu acho isso top: http://pt.wikipedia.org/wiki/REST

Eu gosto muito da semântica e padronização que o REST proporciona.

### Exemplos:

O SexTape App quer remover um vídeo.
Então faz uma request com o verbo HTTP DELETE para http://api.super_sexy_tapes.com/v1/videos/1

O SexTape App quer adicionar um vídeo.
Então faz uma request com o verbo HTTP POST para http://api.super_sexy_tapes.com/v1/videos?title=...

O SexTape App quer atualizar um vídeo.
Então faz uma request com o verbo HTTP PATCH para http://api.super_sexy_tapes.com/v1/videos/1?title=...

O SexTape App quer obter detalhes de um vídeo.
Então faz uma request com o verbo HTTP GET para http://api.super_sexy_tapes.com/v1/videos/1

Perceba que você possui 2 endpoints parecidos mas que tomam ações absolutamente diferentes:
* DELETE http://api.super_sexy_tapes.com/v1/videos/1
* GET    http://api.super_sexy_tapes.com/v1/videos/1

## Status Code

O Status Code que vem na response é muitas vezes ignorado, mas ele simplesmente diz tudo o que ocorreu durante o processamento da request.

### Exemplos:

O SexTape App quer remover um vídeo.

Então faz uma request com o verbo HTTP DELETE para http://api.super_sexy_tapes.com/v1/videos/1

Ops... vídeo não pode ser removido, porque você não ter permissão pra isso: Status Code: `403 Forbidden`

Yeah... vídeo removido com sucesso: Status Code: `204 No Content`


O SexTape App quer adicionar um vídeo.

Então faz uma request com o verbo HTTP POST para http://api.super_sexy_tapes.com/v1/videos?title=...

Ops... vídeo não pode ser criado, porque você os parâmetros enviados são invalidos: Status Code: `422 Unprocessable 
Entity`

Yeah... vídeo criado com sucesso: Status Code: `200 OK`

Veja essa lista de Status Codes: http://www.restapitutorial.com/httpstatuscodes.html

Meu co-woker publicou esse post aqui sobre a importância de Status Code: http://blog.thefrontiergroup.com.au/2012/08/http-status-codes-and-restful-api-crafting/

## Autenticação

A não ser que exista algum propósito de criar uma API absolutamente pública, você deverá proteger sua API com um token.

No meu ponto de vista, lugar de token não é em query string, lugar de token é no cabeçalho da request!

Se você observar bem, verá que o padrão de HTTP Header suporta uma key chamada "Authorization": http://en.wikipedia.org/wiki/List_of_HTTP_header_fields

Eu não acho que faz muito sentido fazer esse tipo de requisição:
http://api.super_sexy_tapes.com/v1/videos/1?token=QWxhZGRpbjpvcGVuIHNlc2FtZQ==

Bota seu token no cabeçalho e deixe a query string livre pra ser usada para armazenar dados relacionados a ação da request.

## Tratamento de Erros

Bom, nem tudo é perfeito e merdas acontecem.

2 coisas importante pra falar a respeito disso:

* retorne o Status Code correto, então se deu merda, retorne um Status Code referente a isso.
* retorne um JSON com uma mensagem de erro user friendly e sempre com formato padronizado.

### Exemplos:

```json
{
  error: "Por favor, preencha todos os campos marcados como obrigatório e envie novamente."
}
```

Há um tempo atrás, eu costumava adicionar as keys:
* internal_error_message
* internal_error_code
* user_friedly_error_message

Com o intuito de identificar o erro com mais agilidade durante o desenvolvimento, mas eu percebi que era inútil, porque sempre que notava um erro eu ia direto ao ponto ao invés de ler a mensagem ou o código interno de erro.

Enfim, o que você achar melhor.
Mas a idéia é sempre manter o response body de erro no mesmo formato, padronizado, facilita no desenvolvimento do App.


## Padronização dos dados

Considere:

Endpoint: http://api.super_sexy_tapes.com/v1/categories/123
Response Body:

```json
{
  category: {
    id: 123,
    title: "Vizinhas Gostosas"
    description: "Esta categoria contém uma vasta seleção de vizinha gostosas......"
  }
}
```

E o endpoint:

Endpoint: http://api.super_sexy_tapes.com/v1/videos/1
Response Body:

```json
{
  video: {
    title: "Minha vizinha é demais",
    category: {
      title: "Vizinhas Gostosas"
    }
  }
}
```

Opa!!! Por que categoria no primeiro endpoint possui dados diferentes em relação ao segundo endpoint?
Isso não é legal, o correto seria:

Endpoint: http://api.super_sexy_tapes.com/v1/videos/1
Response Body:

```json
{
  video: {
    title: "Minha vizinha é demais",
    category: {
      id: 123,
      title: "Vizinhas Gostosas"
      description: "Esta categoria contém uma vasta seleção de vizinha gostosas......"
    }
  }
}
```

Dessa forma, o desenvolvimento de apps fica mais fácil, porque os parses de vídeo e categoria serão sempre os mesmos e poderão ser reaproveitados.

Me corrijam se estiver errado, mas o nome desse padrão é [Decorator](http://en.wikipedia.org/wiki/Decorator_pattern), eu acho.

## Documentação

Dispensa comentários...
Absolutamente imprescindível, ou você quer que eu advinhe como sua API funciona?

Minha inspiração é o Twitter: https://dev.twitter.com/rest/public

Na minha empresa atual, fizemos algo que me orgulho muito!

Adaptamos uma suite de teste que ao rodar, case o teste do endpoint da API seja "OK", então ele gera automaticamente a documentação em HTML.
Se o teste para o endpoint falar, ele gera a documentação mas acusa falha.

FYI: https://github.com/zipmark/rspec_api_documentation

## Dicas Finais

### Inserção de dados
É bem provável que você queira exibir os dados que você acabou de criar. Então é interessante que:

`HTTP POST para http://api.super_sexy_tapes.com/v1/videos?title=...`

Retorne um `response body` contendo os dados do vídeo que você mandou adicionar.

### Remoção de dados
Até hoje, eu nunca vi a necessidade de retornar uma `response body` com conteúdo após fazer uma request para remover algum dados.
Eu costumo retornar nada, ou apenas um:

```json
{ success: true }
```

### Validação
Quando o usuário preenche um form com dados inválidos e submete, comumente enviados a request pra API, a API identifica que há algo errado e retorna os erros.

Se possível, faça a validação localmente, e faça a request quando há maior chance da mesma suceder.

### Cache
De acordo com as circuntâncias e na medida do possível, faça cache mobile-sided, e obviamente server-sided.

### Postman - REST Client
Eu remomendo instalar essa extensão do Google Chrome: https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm

O Postman permite que você faça requisições pra sua API caso deseje utilizá-la durante o desenvolvimento ou análise de bugs.

### Cases
Api's que eu já usei e achei bem elaboradas:

* http://developer.echonest.com/docs/v4
* https://dev.twitter.com/rest/public
* https://developers.google.com/books/docs/v1/using
