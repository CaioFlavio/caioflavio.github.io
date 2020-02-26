---
title: Laravel - Criando tabelas para o banco de dados
date: '2020-02-26T19:05:16-03:00'
categories:
  - PHP
  - Laravel
tags:
  - Laravel
autoThumbnailImage: false
thumbnailImagePosition: left
coverImage: /images/uploads/cover-post.jpg
---
Esse artigo será focado em listar os passos pra criação de tabelas no banco de dados utilizando o laravel. Para fazer isso iremos utilizar as migrations.

## Criando migrations pelo artisan
 
Podemos criar os arquivos de migrations no Laravel utilizando os seguintes comandos:

```
php artisan make:migration create_users_table --create=users
ou
php artisan make:migration create_users_table --table=users
```
