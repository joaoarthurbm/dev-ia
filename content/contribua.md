+++
title = "Contribua"
date = "2026-01-01"
menu = "main"
weight = "50"
meta = "false"
+++

***

Você pode contribuir com o material da disciplina: correções, sugestões, novos conteúdos, figuras e exemplos.

## Como contribuir

1. Leia o [README](https://github.com/joaoarthurbm/dev-ia) para entender a estrutura do repositório;
2. Crie um *fork* do repositório;
3. Produza o conteúdo seguindo o [guia de posts](/dev-ia/contribua/#como-criar-um-post);
4. Envie um *Pull Request* com o arquivo `.md` e as figuras, se houver.

## Como criar um post

Um post é um arquivo `.md` dentro da pasta `content/posts`. Veja o modelo abaixo:

<pre>
+++
title = "Título do post"
date = 2026-01-01
tags = []
categories = []
+++

Conteúdo do post em markdown.
</pre>

Se o post tiver figuras, crie um diretório com o mesmo nome do arquivo dentro de `content/posts` e referencie as imagens a partir dele. Por exemplo, para o post `minha-aula.md`, crie `content/posts/minha-aula/` e use `minha-aula/figura.png` no markdown.

## Validação local

Para gerar o site localmente:

<pre>
hugo server
</pre>

E acesse o endereço indicado no final da saída (normalmente http://localhost:1313/dev-ia/).