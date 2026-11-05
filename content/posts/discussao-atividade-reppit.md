+++
title = "Discussão da atividade com o RePPIT"
date = 2026-11-04
tags = ["reppit", "sdd", "atividade", "validação"]
categories = ["ciclo-dev"]
ppt = ""
youtube = ""
+++

***

Notas da aula em que discutimos a atividade com o workflow RePPIT. Mais do que o resultado, o que interessa aqui é o processo: o que funcionou, o que faltou e o que eu faria diferente.

***

## Preâmbulo

**Sobre estas notas.** Texto organizado a partir da gravação da aula (64 min, 4 de novembro de 2026), com auxílio do Claude na transcrição e na organização. As ideias e opiniões são minhas e da turma.

## Contexto

Na atividade, vocês implementaram uma feature de busca de músicas por sequência de acordes (cifra) num projeto que não conheciam, seguindo o workflow RePPIT. O RePPIT é uma implementação de terceiros distribuída como comandos e skills, com fases de *research* e *propose* antes da implementação. Como as licenças de vocês não são Pro, havia limite de recursos.

> A atividade vale pelo processo, não pelo resultado.

## Janela de contexto e divisão em sessões

Vários de vocês tiveram problemas com consumo de tokens e com a janela de contexto. Uma saída é o `/compact`, que já discutimos nas aulas de engenharia de contexto. Outra, que também melhora a qualidade do processo, é **não fazer tudo numa única sessão**. Uma sessão sobrecarregada de informação pode levar o modelo a respostas piores.

A técnica que vimos em aula é separar o trabalho em unidades menores, sempre guardando o estado em arquivos `.md`:

1. **Planejamento** numa sessão, que termina gerando um documento.
2. **Especificação detalhada** em outra.
3. **Implementação** e **validação** em outras.

A feature não era trivial. Se você fez tudo numa sessão só, reveja essa escolha.

Minha orientação sobre limites é conservadora. Chegar a 50% da janela já é indesejável. A experiência de quem usa muito esses modelos indica que **entre 40% e 45% é hora de compactar ou trocar de sessão**. O agente que travou para um de vocês provavelmente teve um problema desse tipo, agravado por uma instrução ambígua da própria atividade sobre o log da sessão (essa parte é culpa minha). De todo modo, o sofrimento ensina: às vezes a dor comunica melhor do que a paciência.

**Uma ideia que ainda não testei:** gerar o DeepWiki do projeto e entregá-lo ao agente na fase de research, para que ele não refaça do zero o entendimento do repositório. Ele gastaria tokens para entender o DeepWiki, mas provavelmente menos do que para entender o código inteiro. Ninguém da turma tentou.

## O que faltou no RePPIT: a fase de entendimento

Esta é a minha crítica central ao workflow: **a fase de research olha só para o código.** Ela é apenas uma parte do que chamo de **fase de entendimento**, que inclui:

* entender a base de código;
* discutir a feature;
* nivelar o vocabulário e os conceitos do domínio;
* construir um glossário.

Vocês não conheciam o projeto, e a feature exigia conhecimento musical. O research sozinho não bastava, e o propose também não serve bem para discutir a feature como um todo. Eu esperava que vocês criticassem isso e complementassem a fase com outras skills, como o *grill-me*, que faz o modelo perguntar exaustivamente sobre a tarefa. O próprio Claude já faz algo parecido quando você pede uma tarefa, mas o grill-me torna isso explícito. Vocês podiam alterar os comandos do RePPIT se julgassem necessário, ou usá-los como estão e sugerir mudanças depois.

Um aluno usou o grill-me na fase inicial. A maioria usou só o research. Ainda não analisei as entregas, mas acredito que o resultado seria melhor com a fase de entendimento ampliada.

## Requisitos e fonte da verdade

Vocês tomaram decisões de requisito funcional conversando apenas com o agente, sem consultar o cliente (eu). O exemplo que discutimos foi a semântica da busca:

| Semântica | Regra | |
|---|---|---|
| Restritiva | a sequência aparece contígua e na mesma ordem na cifra | a que eu teria escolhido |
| Permissiva | todos os acordes da sequência aparecem na música, em qualquer ordem | |

Nenhuma das duas é errada em si. **Mas a decisão precisava ser tomada com quem decide.**

Havia ainda um mecanismo que dispensaria a pergunta: o enunciado trazia sequências já definidas, com resultados esperados, que funcionavam como gabarito. Quem as usou como testes de regressão tinha uma fonte de verdade e está dispensado de ter perguntado. Quem tornou o critério mais permissivo e alterou resultados que já eram verdade tomou uma decisão contrária ao que estava especificado.

> A fonte de verdade é sempre a especificação ou os testes de regressão.

## Bugs que exigem conhecimento de domínio

Vocês encontraram dois bugs:

* **7M ≠ 7m.** Na notação de acordes, 7M (sétima maior) e 7m (sétima menor) são coisas diferentes. Normalizar tudo para maiúsculas ou minúsculas quebra o significado. Quem conhecia música percebeu; quem não conhecia, não.
* **Vírgula vs. ponto e vírgula.** Os dois separadores eram tratados de forma diferente. Eu nem sabia desse bug; vocês o descobriram no processo.

Os dois reforçam a importância de **entender o domínio antes de implementar**. E esse tipo de dúvida deveria ter chegado a mim pelo classroom.

## Segurança, REST e requisitos não funcionais

**Segurança.** Uma implementação incluiu uma rota de delete que permite a qualquer visitante apagar sequências. Como o sistema não tem login, isso é um problema, assim como permitir adições ilimitadas. O ambiente é educacional, mas a decisão deveria ter sido discutida.

**REST.** Outro problema recorrente foi nomear rotas com verbos de ação (`/cadastra`, `/remove`, `/lista`). Isso contraria o estilo REST, em que o recurso é o substantivo e a ação vem do verbo HTTP: `POST /sequencias` cadastra, `DELETE /sequencias/{id}` remove. É um antipadrão que vocês já viram em Web e em Projeto de Software.

> O agente não necessariamente avisa quando foge da especificação. Você precisa de mecanismos para verificar que ela está sendo seguida.

Testes ou análise estática que confirmem quais rotas foram criadas são exemplos desses mecanismos.

**Desempenho.** O tempo de busca variou muito entre as entregas, e algumas ficaram acima do aceitável para uma aplicação web, onde a regra geral é responder em poucas centenas de milissegundos. Poucos de vocês se preocuparam com isso. Percorrer todas as músicas a cada busca é caro. Algumas alternativas:

* pré-calcular as sequências populares;
* usar cache para as mais buscadas ou as mais recentes;
* ao cadastrar uma sequência que já existe, simplesmente retornar a existente, sem refazer a busca no banco.

Em geral, segurança e desempenho foram pouco considerados. **Requisitos não funcionais fazem parte do trabalho.**

## Validação e controle humano

O agente costuma fazer muito mais do que foi pedido e alterar partes do código existente, o que torna difícil acompanhar tudo. Gosto de pensar nele como **um labrador solto numa sala: é preciso colocar a coleira.**

Por isso é útil um prompt de retroalimentação que pergunta *se foi preciso fazer algo diferente do planejado*. Ele faz parte da validação: serve para enxergar os desvios e combater o excesso de engenharia. A validação pode ser determinística, com testes, ou feita perguntando ao agente que arquivos ele mudou e por quê.

Quando perguntei quem discordou do agente em algum momento, ninguém respondeu.

> Human in the loop precisa significar human in control.

Se o processo é spec driven, como discutimos duas semanas antes, o caminho quando o resultado não agrada é **voltar e refazer a especificação**, que é a fonte de verdade. Se você mexe só na implementação, especificação e código divergem, e a especificação acaba descartada. E quando o agente propõe duas alternativas, quem decide é você.

## Front-end e validação manual

Ninguém usou skills para entender o front-end ou seguir as regras visuais existentes. Eu teria feito assim:

1. abrir uma sessão separada para o agente entender a tela atual;
2. pedir propostas de como cadastrar novas sequências;
3. decidir;
4. pedir que ele mantivesse o padrão visual;
5. registrar as decisões de front-end em arquivo, para reuso em outras sessões.

Como a feature é vertical, com mudanças em front-end, back-end e dados, também dá para fazer o front-end primeiro, com rotas simuladas que retornam sempre o mesmo resultado.

Para quem usa o sistema, o que vale é o que se vê no front-end. Por isso **a validação manual ganha importância com agentes**. O custo é o trabalho humano, mas os velhos roteiros de teste de interface, que todo mundo achava enfadonhos, voltam a fazer sentido. Há empresas que dedicam semanas inteiras a isso.

## Tempo de trabalho e fadiga

Os relatos de tempo variaram bastante: uma sessão direta de cerca de duas horas, três a quatro horas para alguns, cerca de dez horas brutas para um aluno, e um que dividiu o trabalho em dois dias.

Alguns disseram ter sentido que deviam seguir o plano gerado pelo agente. Acho esse sentimento natural, mas o controle precisa ser mantido. Um plano de duas páginas que mexe no front-end, cria banco e faz deploy tira você do controle, e a proatividade constante do agente é um risco. A saída é **picotar o trabalho em etapas pequenas**, entendendo bem uma antes de passar à próxima.

Como diretriz, sugiro **sessões de duas a três horas**. Se a tarefa não cabe nisso, divida em duas sessões. Tentar fazer tudo de uma vez é um erro estratégico, e o custo é cansaço e perda de atenção: você entrega porque precisa, sem perceber o que o agente fez.

Reconheço a pressão de mercado, mas processos bem feitos levam tempo. Sou refratário a vídeos que mostram, por exemplo, dois mil PRs em um mês, porque ali o controle humano é nulo.

> O desenvolvedor deve comandar o loop, e não ser jogado para fora dele.

## Sobre o futuro da profissão

Costumo errar previsões. Mas desconfio de quem diz que tudo será muito fácil e muito rápido, ou que ninguém mais vai programar, principalmente quando isso serve para vender produto ou consultoria.

O front-end é complexo porque envolve visual e usabilidade. Os requisitos não funcionais arquiteturais (segurança, desempenho, latência, microsserviços ou não, persistência) trazem uma complexidade que só percebe quem dedica a vida ao assunto. O vibe coding pode servir a um arquiteto, ou a um médico que faz um protótipo para a própria clínica. Para o restante, o modelo ajuda, mas não resolve, porque o trabalho envolve decisão: saber o que testar e como, e o que importa naquele contexto específico.

Há benchmarks que humanos resolvem com facilidade e modelos não. E o modelo precisa de instruções muito bem definidas, que só um engenheiro experiente sabe dar.

> A profissão não está em risco, mas passa por uma mudança forte. Saber os conceitos fundamentais importa cada vez mais.

## Lições

1. **Invista tempo e recursos na fase de entendimento**, com skills, não só para gerar um plano, mas para alinhá-lo na sua cabeça e com o modelo. A grande dificuldade do RePPIT foi justamente pular essa etapa.
2. **Particione o trabalho em sessões diferentes**, mesmo sem limitação de créditos.
3. **Planeje o tempo.** Muitas horas seguidas com o modelo levam à fadiga de aprovação e de leitura. Algo como duas horas por etapa é um bom ponto de partida.
