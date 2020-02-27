---
title: Laravel - Configurando um servidor Graphql - Parte 1
date: '2020-02-27T15:45:20-03:00'
categories:
  - PHP
  - Laravel
  - GraphQL
tags:
  - PHP
  - Laravel
  - GraphQL
keywords:
  - PHP
  - Laravel
  - GraphQL
autoThumbnailImage: false
thumbnailImagePosition: top
coverImage: /images/uploads/cover-post.jpg
---
## Introdução

Iremos configurar um servidor GraphQL utilizando o laravel e os pacotes noh4ck/graphiql e rebing/graphql-laravel. Para este tutorial é recomendado uma versão do laravel >= 5.5.x e <= 5.8.x.

## Primeiros passos

A primeira coisa a fazer é instalar uma versão 5.x.x do laravel, pois o pacote noh4ck/graphiql ainda não possui suporte pra versão 6.x.x. Portanto iremos utilizar o seguinte comando:

```
composer create-project --prefer-dist laravel/laravel graphql-api "5.8.*"
```

Agora, dentro da pasta do nosso projeto instalaremos via composer os dois pacotes para podermos subir o servidor GraphQL.

```
composer require rebing/graphql-laravel && php artisan vendor:publish --provider="Rebing\GraphQL\GraphQLServiceProvider"

```
```
composer require "noh4ck/graphiql:@dev" && artisan graphiql:publish
```

Após a instalação dos pacotes, vamos testar a aplicação utilizando o servidor embutido do laravel, através do comando.
```
php artisan serve
```

No browser de sua preferencia, é possível acessar o GraphQl UI acessando o [http://127.0.0.1:8000/graphql-ui](http://127.0.0.1:8000/graphql-ui)

![GraphQL UI](https://i.imgur.com/F1rhVqo.png "Laravel GraphQL UI")

Pra ele funcionar corretamente ainda é necessário configurar as tabelas que disponibilizarão a informação para API. Para este tutorial utilizaremos um banco SQLite. No arquivo de configuração (.env) vamos remover todas as configurações relacionadas a banco de dados e deixar somente a opção DB_CONNECTION com o valor sqlite.

```
DB_CONNECTION=sqlite
```

e agora criaremos o arquivo de banco na pasta base do nosso projeto

```
touch database/database.sqlite
```

Agora rodaremos as _**Migrations**_ onde o Laravel criará a tabela de usuários.

```
php artisan migrate 
```

Vamos usar o tinker do laravel pra criar alguns usuários
```
php artisan tinker
```
e em seguinda no console do tinker, podemos usar o seguinte comando:
```
factory(App\User::class, 100)->create()
```

Agora precisamos criar os arquivos que irão lidar com as consultas do GraphQL a tabela de usuários. O primeiro passo para isso é criar um "type". De acordo com a documentação uma _**Eloquent Model**_ só é necessária quando especificamos relações entre tabelas. No diretório _app/GraphQL/Types_ crie o arquivo _UserType.php_, e adicione o seguinte conteúdo.


```php
<?php
namespace App\GraphQL\Types;

use App\User;
use GraphQL\Type\Definition\Type;
use Rebing\GraphQL\Support\Type as GraphQLType;

class UserType extends GraphQLType
{
    protected $attributes = [
        'name'          => 'User',
        'description'   => 'A user',
        'model'         => User::class,
    ];

    public function fields(): array
    {
        return [
            'id' => [
                'type' => Type::nonNull(Type::string()),
                'description' => 'The id of the user',
            ],
            'email' => [
                'type' => Type::string(),
                'description' => 'The email of user',
                'resolve' => function($root, $args) {
                    return strtolower($root->email);
                }
            ],
            'isMe' => [
                'type' => Type::boolean(),
                'description' => 'True, if the queried user is the current user',
                'selectable' => false,
            ]
        ];
    }

    protected function resolveEmailField($root, $args)
    {
        return strtolower($root->email);
    }
}
``` 

Agora precisamos adicionar essa classe ao arquivo de configuração do graphql (config/graphql.php):

```
'types' => [
    'user' => App\GraphQL\Types\UserType::class
]
```

Agora precisamos definir uma _**Query**_ que retorne este _**Type**_. No diretório _app/GraphQL/Queries_ crie o arquivo _UsersQuery.php_ e adicione o seguinte conteúdo.

```php
<?php

namespace App\GraphQL\Queries;

use Closure;
use App\User;
use Rebing\GraphQL\Support\Facades\GraphQL;
use GraphQL\Type\Definition\ResolveInfo;
use GraphQL\Type\Definition\Type;
use Rebing\GraphQL\Support\Query;

class UsersQuery extends Query
{
    protected $attributes = [
        'name' => 'Users query'
    ];

    public function type(): Type
    {
        return Type::listOf(GraphQL::type('user'));
    }

    public function args(): array
    {
        return [
            'id' => ['name' => 'id', 'type' => Type::string()],
            'email' => ['name' => 'email', 'type' => Type::string()],
            'name' => ['name' => 'name', 'type' => Type::string()],
        ];
    }

    public function resolve($root, $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
    {
        if (isset($args['id'])) {
            return User::where('id' , $args['id'])->get();
        }

        if (isset($args['email'])) {
            return User::where('email', $args['email'])->get();
        }

        if (isset($args['name'])) {
            return User::where('name', $args['email'])->get();
        }

        return User::all();
    }
}
```
Agora precisamos adicionar a _**Query**_ ao arquivo de configuração do GraphQL (config/graphql.php).

```php
'schemas' => [
    'default' => [
        'query' => [
            'users' => App\GraphQL\Queries\UsersQuery::class
        ],
        // ...
    ]
]
```

Desse modo podemos rodar novamente o nosso servidor embutido com o comando:

```
php artisan serve
```

Acessando [http://127.0.0.1:8000/graphql-ui](http://127.0.0.1:8000/graphql-ui), podemos rodar a seguinte query:

```
query FetchUsers {
    users {
        id
        email
    }
}
```

e obteremos o seguinte resultado:
![GraphQL UI Query](https://i.imgur.com/DwI4tm2.png "Laravel GraphQL Query Funcionando")

## Conclusão

Seguindo esses passos conseguimos configurar corretamente um servidor GraphQL utilizando laravel e os pacotes. Nosso próximo passo será criar um CRUD utilizando laravel e GraphQL.
 
## Referencias

[Rebing/GraphQL Repository](https://github.com/rebing/graphql-laravel)

[Nohac/GraphiQl Repository](https://github.com/Nohac/laravel-graphiql)

[PHP GraphQL API com Laravel — Parte 1](https://medium.com/criciumadev/php-graphql-api-com-laravel-part-1-4df9bcab3c9c)

[Repositorio no git do tutorial](
https://github.com/CaioFlavio/laravel-graphql-api/tree/tutorial-parte-1)
