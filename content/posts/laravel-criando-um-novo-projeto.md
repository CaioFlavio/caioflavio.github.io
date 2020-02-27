---
title: Laravel - Criando um novo projeto
date: '2020-02-27T13:00:55-03:00'
categories:
  - PHP
  - Laravel
tags:
  - PHP
  - Laravel
  - Iniciante
autoThumbnailImage: false
thumbnailImagePosition: top
coverImage: /images/uploads/cover-post.jpg
---
## Introdução

Neste artigo aprendemos como iniciar um novo projeto no laravel e iniciar um servidor local para desenvolvimento utilizando o artisan.

## Iniciando um novo projeto via composer

Pra instalar a versão mais recente do larevel podemos utilizar o seguinte comando:

```
composer create-project --prefer-dist laravel/laravel project_name
```

Caso você precise de alguma versão especifica para o seu projeto devido a compatibilidade com algum pacote a versão pode ser especificada como no comando abaixo:
```
composer create-project --prefer-dist laravel/laravel project_name "5.8.*"
```

## Built-in server

Se você tiver o PHP instalado localmente é possivel rodar um servidor Built-in para servir a sua aplicação utilizando o comando:

```
php artisan serve
```

## Referencias
[Documentação Instalação Laravel](https://laravel.com/docs/6.x#installation "Laravel Docs")

