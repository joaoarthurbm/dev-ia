+++
title = "Anatomia dos Agentes"
date = 2026-08-22
tags = ["agentes", "ai coding agents"]
categories = ["contexto"]
ppt = "https://docs.google.com/presentation/d/1kzahCFbo8UqkQPw1i8N28BvcP3WFpAHUJ_-hJTUBtGo/edit?usp=sharing"
youtube = ""
+++
***
Ao final dessa aula, o aluno deve ser capaz de explicar o funcionamento interno de agentes; explicar o conceito de ***Agent-computer interface***;
; explicar o papel de guardrails e avaliação como componente do loop do agente; identificar os componentes da anatomia (loop, contexto, tools, skills, guardrails) em uma execução real de um *AI coding agent* e implementar um protótipo simples de agente (loop simples de percepção-decisão-ação) para consolidar o entendimento mecânico do loop.
***

**Disclaimer.**  As notas de aula foram escritas por mim em sua totalidade. Não utilizei auxílio de nenhum modelo, embora não veja problema nesse auxílio. Estou deixando isso claro para que você tenha ciência de que, se houver erro aqui ou achar o material ruim, a culpa é minha mesmo. Vale destacar também que as notas de aula são um guia para a discussão em sala de aula e servem muito pouco como conteúdo para estudo.

# AI Coding Agents

O conceito de agentes é antigo. Não nasceu agora. Tem havido uma sobreposição semântica no termo, isto é, tem se interpretado ultimamente agentes como  “LLM based agents”.

De maneira simplória, agente é um loop com tomadas de decisões baseadas em um conjunto de critérios. O que mudou não foi a existência do loop, foi **quem preenche o espaço da decisão dentro dele**. Antes, alguém definia uma heurística, agora a decisão é terceirizada para uma chamada a um LLM a cada iteração do loop. 

LLMs não são agentes. LLMs geram texto.  Eles são um componente da decisão que o agente (loop) consulta. Eles estão intrinsecamente relacionados ao raciocínio, não às ações. 

O LLM **propõe** a ação; quem **executa,** quem de fato mexe no sistema de arquivos, roda o subprocesso, chama a API é o agente (harness/loop).

Antes, LLMs tinham resultados expressivos em tarefas relacionadas ao entendimento de linguagem natural e tomada de decisão interativa, mas habilidades como raciocínio (chain-of-thought) e planejamento de ações eram estudadas de forma separada.

Depois o estudo "ReAct: Synergizing Reasoning and Acting in Language Models", ICLR 2023)  formalizou o padrão Thought → Action → Observation. 

Estudaremos a fundo essa abordagem na aula de prompting. O que precisamos saber agora é que a combinação do LLM (raciocínio) com chamadas de ferramentas é o que dá origem ao loop do agente.

O CoT verbalizado ajuda o modelo a decompor a tarefa. 

Chamar uma ferramenta, fazer uma chamada de API, buscar etc. É esse repertório de ações que dá forma concreta ao "Action" do ReAct. 

Depois de agir, o agente reavalia onde está. Usa o que observou para decidir entre responder de vez ou formular o próximo Thought.

Um AI Coding Agent é um agente especializado em desenvolvimento de software.

# Anatomia de um agente: exemplo concreto

O vocabulário que está por vir: loop, contexto, tools e guardrails/avaliação. 

O cenário abaixo é pequeno de propósito: um repositório com uma função `apply_discount` que tem um bug, e um teste que falha por causa dele. O prompt usado está abaixo. Depois vamos discutir porque esse pode não ser um bom prompt. Por ora, vamos com ele.

```jsx
**"conserte o teste que está falhando em test_inventory.py"**. 
```

## O trace

**Thought:** Preciso entender a tarefa antes de agir. O teste test_inventory.py está
falhando — vou ler o código-fonte e o teste para ver o que está em jogo.

**Action:** read([inventory.py](http://inventory.py/))
**Action:** read(test_inventory.py)

**Observation:**
def apply_discount(price, percent):
"""Aplica um desconto percentual ao preço."""
return price - percent

```
from inventory import apply_discount

def test_apply_discount():
    assert apply_discount(200, 10) == 180
```

---

**Thought:** Já tenho o código e o teste, mas ainda não sei o estado atual da
execução. Antes de mexer em qualquer coisa, vou rodar o teste para confirmar
a falha e ver a mensagem de erro exata.

**Action:** execute(pytest test_inventory.py -v)

**Observation:**
FAILED test_inventory.py::test_apply_discount
assert 190 == 180
+  where 190 = apply_discount(200, 10)

---

**Thought:** O teste espera 180, e a função devolveu 190 fazendo price - percent
(200 - 10 = 190). O docstring diz "desconto percentual" — isso sugere que
percent deveria ser tratado como fração de 100, não subtraído diretamente.
A fórmula correta é price * (1 - percent / 100): 200 * 0.9 = 180, que bate
com o esperado. Vou aplicar essa correção.

**Action:** edit([inventory.py](http://inventory.py/))

**Observation:**
- return price - percent
+ return price * (1 - percent / 100)

---

**Thought:** Apliquei a correção com base numa hipótese, não numa certeza. Preciso
rodar o teste de novo para confirmar que a hipótese estava certa antes de
considerar a tarefa concluída.

**Action:** execute(pytest test_inventory.py -v)

**Observation:**
test_inventory.py::test_apply_discount PASSED
1 passed in 0.01s

---

**Thought:** O teste passou. Não há mais falhas conhecidas — a tarefa está concluída.

## O ponto de partida

O agente recebe a instrução do usuário. 

Antes de decidir qualquer coisa, o agente ainda não sabe o que está no repositório. Então a primeira ação é reunir informação.

### Tool Calling

```
read inventory.py
read test_inventory.py
```

python

```python
def apply_discount(price, percent):
    """Aplica um desconto percentual ao preço."""
    return price - percent
```

python

```python
from inventory import apply_discount

def test_apply_discount():
    assert apply_discount(200, 10) == 180
```

Duas chamadas de ferramenta. Vamos abrir um parênteses aqui para tratar especificamente disso.

---

Mas como isso é (pode ser) feito? 

- *Padronização do texto + parsing.* o prompt ensina o modelo a escrever `Action: nome[argumento]`, e quem está por fora fica caçando esse padrão no texto com regex.
- *Pedir em formato JSON.* Um meio-termo: pede-se ao modelo que responda em JSON seguindo um formato descrito no próprio prompt. Melhora o parsing, mas ainda não há garantia.
- *Tool calling nativo.* O provedor do modelo treina o próprio modelo pra emitir a chamada como um tipo de saída estruturado, validado contra um schema que o desenvolvedor declara antes (nome, tipos, quais argumentos são obrigatórios). Quem recebe não faz mais parsing de texto solto, recebe algo já tipado e checado. É isso assim que os agentes funcionam hoje.

Como projetar uma interface para um consumidor que não lê documentação, não é determinístico, e é sensível a tudo que está no contexto ao seu redor?  Projetando interfaces para os mesmos: **Agent-Computer Interface.**


É tudo sobre os princípios:
- *Actions should be simple and easy to understand for agents.*
- *Actions should be compact and efficient.*
- *Environment feedback should be informative but concise.*
- *Guardrails mitigate error propagation and hasten recovery.*

---

O conteúdo desses dois arquivos agora faz parte do contexto do agente. 

O contexto não é um dado que o modelo possui, precisa ser ativamente montado, turno a turno.

Refs

YANG, John et al. Swe-agent: Agent-computer interfaces enable automated software engineering. **Advances in Neural Information Processing Systems, 2024.**

## A primeira volta do loop: agir e observar a falha

Com os arquivos lidos, o agente decide que o próximo passo natural é confirmar o problema rodando o teste. Isso já é uma decisão, ainda que pequena: a escolha de investigar antes de agir sobre a causa.

```
execute: pytest test_inventory.py
```

```
FAILED test_inventory.py::test_apply_discount - assert 190 == 180
E       assert 190 == 180
E        +  where 190 = apply_discount(200, 10)
```

Essa saída de erro é o segundo tipo de contexto que aparece no exemplo, e é diferente do primeiro: não veio de um arquivo estático, veio do *resultado de uma ação do próprio agente*. É exatamente esse ciclo: **agir**, **observar** o resultado, e **usar** esse resultado para decidir a próxima ação.. Sem essa realimentação, o agente teria apenas um palpite sobre o código.

## Thought: O passo que normalmente não aparece

Entre observar `190 == 180` e decidir editar o arquivo, há um raciocínio: "`percent` está sendo subtraído do preço em vez de aplicado como uma fração". Esse passo não gera nenhuma chamada de ferramenta. Ele faz parte do “thought”. Normalmente as ferramentas escondem isso, mas aqui o que foi gerado pelo log no nosso exemplo:

```jsx
**Thought:** O teste espera 180, e a função devolveu 190 fazendo price - percent
(200 - 10 = 190).

O docstring diz "desconto percentual" — isso sugere que
percent deveria ser tratado como fração de 100, não subtraído diretamente.

A fórmula correta é price * (1 - percent / 100): 200 * 0.9 = 180, que bate
com o esperado.

Vou aplicar essa correção.
```

<strong> Isso aqui é o LLM processando o contexto acumulado (a função, a documentação, o teste, o erro) e decidindo o que fazer a seguir.</strong>

O docstring diz "aplica um desconto **percentual**". O LLM viu esse padrão de código milhares de vezes no treinamento. Isso não é raciocínio matemático, é reconhecimento de padrão: "desconto percentual" quase sempre vira `price * (1 - percent/100)` em código real.

O teste dá **um único par entrada/saída.** Isso não especifica a fórmula, só restringe o espaço de fórmulas possíveis. Muitas soluções passariam nesse teste. O agente escolheu a que casa com o padrão textual do passo anterior.

## Segunda volta do loop: agir sobre a causa

Com o diagnóstico feito, o agente edita o arquivo:

diff

```diff
- return price - percent
+ return price * (1 - percent / 100)
```

Essa é a ação central da tarefa, mas note que ela só foi possível porque as duas voltas anteriores do loop (ler, depois rodar e observar a falha) já tinham montado o contexto necessário. Se o agente tivesse editado o arquivo na primeira iteração, sem rodar o teste antes, estaria decidindo às cegas.

## O guardrail: como o agente sabe que terminou

```
execute: pytest test_inventory.py
```

```
test_inventory.py::test_apply_discount PASSED
1 passed in 0.01s
```

Agente decide parar porque um critério verificável (o teste passou) foi satisfeito. Esse é o papel de guardrails e avaliação dentro do loop: não é uma etapa de qualidade de código feita depois, por um humano ou por outra ferramenta. É o mecanismo que o agente usa, em tempo real, para decidir entre continuar, tentar de novo, ou parar. Tirem esse passo do exemplo e o agente não tem como saber se `price * (1 - percent / 100)` era a correção certa ou apenas mais uma tentativa.

# Generalizando

```jsx
contexto = [tarefa inicial]
enquanto não terminou:
    thought, action = LLM(contexto)
    se action == "parar":
        terminou = verdadeiro
    senão:
        observation = executar(action)
        contexto = contexto + [thought, action, observation]
```

# Referências

ReAct: Synergizing Reasoning and Acting in Language Models", ICLR 2023)