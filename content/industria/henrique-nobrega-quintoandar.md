+++
title = "Henrique Nóbrega (Quinto Andar)"
date = 2026-10-28
draft = false
description = "Henrique Nóbrega, engenheiro de software no Quinto Andar, com passagens por Alura e Uber, fala sobre entendimento de domínio, tracking de decisões e onde a IA entra no dia a dia de desenvolvimento."
youtube = "https://www.youtube.com/watch?v=H5X4cZ435UQ"
empresa = "Quinto Andar"
slides = ""

[[convidados]]
  nome = "Henrique Nóbrega"
  cargo = "Engenheiro de Software"
  link = ""
+++

***

Conversa com o Henrique Nóbrega, engenheiro de software no Quinto Andar, com passagens por Alura e Uber, sobre como ele e o time usam IA no dia a dia de desenvolvimento. A apresentação de Henrique está disponível no youtube. Aqui eu vou escrever um pouco sobre o que discutimos.

**Minha principal observação:** Henrique começou a apresentação falando de engenharia de software, incluindo mapa de domínio e tracking de decisões, antes de chegar a IA e agentes. Isso reforça algo que já apareceu nas conversas anteriores com Pedro Serey e Jonhnny Weslley: o que diferencia quem usa bem agente não é o agente, <strong>é o entendimento do domínio e do problema que vem antes dele</strong>.

# Alguns pontos

* Mapa de domínio e tracking de decisões são práticas que Henrique usa de forma cotidiana. Entender o domínio e registrar por que uma decisão foi tomada é o que permite usar agente com mais segurança depois, porque dá ao agente (e a quem revisa) o contexto que normalmente só existiria na cabeça de quem decidiu.

* <strong>IA como "decorator" dos sistemas que o time já usa.</strong> Em vez de substituir ferramenta ou processo existente, a IA se encaixa em cima deles, reforçando o que já vimos em aula sobre contexto: o ganho não vem de trocar o workflow, vem de adicionar uma camada que opera dentro dele.

* Como o custo de errar caiu, o time pode assumir mais riscos: é barato voltar atrás numa decisão quando boa parte do trabalho foi feita (ou refeita) por um agente.

* <strong>Fadiga de leitura de texto gerado por IA.</strong> Revisar PRs, specs e documentos gerados por agente cansa de um jeito diferente de revisar texto escrito por humano.

* `/investigate` no workflow do time, no mesmo espírito dos comandos que já vimos em aula (`/research`, `/grill-me`): usado para entender o problema antes de qualquer mudança.

# Perguntas e dores

Algumas perguntas ficaram no ar durante a conversa e vale assistir ao vídeo completo para ver como Henrique respondeu:

* Quando algo quebra em produção, como saber se o trecho problemático foi escrito por humano ou gerado por IA, sobretudo no backend? Isso importa para quem vai debugar e para quem assume a responsabilidade pelo código.

* Como é a gerência de skills do time: quem escreve, quem revisa, e como evitam divergência entre o que a skill descreve e o que o time de fato faz.

* O caso do GitHub de desenvolver código pensando em quem (ou o que) vai lê-lo depois,inclusive a própria IA. Onde esse tipo de decisão fica documentado?

* Dores: nem todo uso de agente deu certo. Quais casos falharam e por quê?