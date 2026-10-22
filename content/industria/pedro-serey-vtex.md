+++
title = "Pedro Serey (VTEX)"
date = 2026-10-21
draft = false
description = "Pedro Serey, engenheiro de software na VTEX, apresenta o workflow que usa no dia a dia com OpenCode."
youtube = "https://www.youtube.com/watch?v=ULa2a43pLg4"
empresa = "VTEX"
slides = "https://docs.google.com/presentation/d/1Co2t5XOVs8nrcRZ6NeBSpJNNPF4iBbScDMbM6QUbaaQ/edit"

[[convidados]]
  nome = "Pedro Serey"
  cargo = "Engenheiro de Software"
  link = ""
+++

***

Conversa com o Pedro Serey, engenheiro de software de Developer Experience na VTEX, sobre o workflow pessoal que ele usa com agentes de código, usando o OpenCode como ferramenta. A apresentação do Pedro está disponível no youtube. Aqui eu vou escrever um pouco sobre o que discutimos.


**Minha principal observação:** mais uma vez ficou claro que, por trás de workflows diferentes, estão os mesmos princípios fundamentais. O workflow de Pedro não é igual ao do Matt Pocock, que não é igual ao do Jonhnny Weslley e assim por diante. Cada um organiza etapas, ferramentas e nomes do seu jeito. Mas todos giram em torno do mesmo núcleo: entender, alinhar, especificar, implementar e validar. A forma muda, <strong>o princípio não muda</strong>.

A outra observação é sobre onde está o trabalho de quem desenvolve hoje. O modelo por trás do agente já sabe muito, mas é genérico. O que diferencia um bom uso de um uso mediano é a personalização: usar skills, MCPs e engenharia de contexto para que o agente produza o que aquele time e aquele projeto esperam. As preferências da equipe, os artefatos esperados, o jeito de programar daquele projeto, nada disso vem pronto no modelo, <strong>é contexto que a gente constrói</strong>.

# Alguns pontos

* O uso combinado do /grill-me com o /plannotator é excelente para a fase de entendimento.
* Muitas das tasks de Pedro envolvem vários repositórios ao mesmo tempo. A solução dele foi criar um diretório `workspace` reunindo todos os repositórios separadamente, com um `AGENTS.md` explicando a organização, e um diretório `tasks-artifacts` onde, para cada task, ele mantém o `PLAN.md`, o `TODO.md` e um diretório `evidencias`.

<img src="/industria/pedro/dirs.png">

# Referências de Pedro

* [Repositório com as skills e o AGENTS.md usados na demo](https://github.com/pserey/opencode-demo)
* [Plannotator](https://plannotator.ai/)
* [Herdr](https://herdr.dev/) — multiplexador de terminal para agentes
* [Tweet de Robert C. Martin](https://x.com/unclebobmartin/status/2080257779395154409)

