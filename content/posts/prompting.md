+++
title = "Prompting"
date = 2026-08-28
tags = ["prompting"]
categories = []
ppt = "https://docs.google.com/presentation/d/1aaQdQAV4OZ6xnqF2WDyF1D92mMXxCVjbVLy7khU2WF0/edit?usp=sharing"
youtube = ""
+++

***

Ao final dessa aula, o aluno deve ser capaz de entender e implementar técnicas eficientes de prompting.

***

# Preâmbulo

**Sobre as notas de aulas:** i) Conteúdo escrito por mim em sua totalidade. Tabelas e figuras foram geradas com auxílio do Claude. Estou deixando isso claro para que você tenha ciência de que se houver erro aqui ou achar o material ruim, a culpa é minha mesmo;  e ii) as notas de aula são um guia para a discussão em sala de aula e servem muito pouco como conteúdo para estudo.


# Disclaimer

> Há um movimento natural de shift do prompting manual para formas mais estruturadas de especificação, engenharia de contexto e configuração do agente.

Contudo, muitas das técnicas e frameworks atuais (SDD, RePPIT, RPI etc) preservam princípios fundamentais do *prompt engineering*.

Quando você escreve uma spec para um agente, você ainda está, em última instância, escrevendo instruções para um modelo.

Hoje vamos aprender a andar para depois correr. 

> It's all about the principles.

O jeito que a gente instrui os agentes pode mudar, os princípios se mantém.


# O modo como vamos ver isso hoje

- Prompt ruim, resultado ruim
- Melhoria no prompt, melhoria no resultado

Com esse esquema, meu objetivo é ir apresentando os conceitos importantes, como zero/few-shots, Chain of Thought, entre outros.

# Definições

- Prompt se divide em system prompt, user prompt e a tarefa específica.

Exemplo de System Prompt

```
Você é um assistente de engenharia de software especializado em Python.

## Comportamento
- Responda de forma concisa e direta, sem preâmbulos.
- Antes de editar código, leia os arquivos envolvidos para entender as convenções.
- Nunca adicione comentários ao código, a menos que solicitado.
- Sempre rode os testes após implementar uma feature.

## Restrições
- Não comita o código sem permissão explícita.
- Não modifique arquivos fora do escopo da tarefa.
- Não exponha segredos ou chaves de API.

## Formato de resposta
- Para perguntas: responda em no máximo 3 frases.
- Para código: use code blocks com a linguagem correta.
```

# Zero-shot

Prompt ruim: "Escreva uma função que valida se um voucher pode ser usado no checkout."

Prompt bom:
```
<role>
Engenheiro no módulo de descontos do Saleor. Convenção da equipe: lançar exceção de domínio, não retornar bool.
</role>
<task>
Implemente validate_voucher(voucher, total_price, quantity, customer_email, channel, customer) -> None
</task>
<constraints>
- Valor mínimo gasto
- Quantidade mínima de itens
- Uso único por cliente, se apply_once_per_customer
- Exclusivo para staff, se only_for_staff
</constraints>
```

# k/few-shot

Pedindo com exemplos.

Prompt ruim: "Escreva uma função Python que calcula o desconto de frete dado um voucher e o preço do frete."

Saída ruim:
```python
def get_shipping_voucher_discount(voucher, shipping_price, channel):
    discount = voucher.get_discount_amount_for(shipping_price, channel)
    if discount > shipping_price:
        return shipping_price
    return discount
```

Prompt bom:
```
# Role
Engenheiro no módulo de descontos do Saleor.


# Exemplo
def get_products_voucher_discount(voucher: "Voucher", prices: Iterable[Money], channel: Channel) -> Money:
    """Calculate discount value for a voucher of product or category type."""
    ...

# Task
No mesmo estilo do exemplo acima, implemente:
get_shipping_voucher_discount(voucher: "Voucher", shipping_price: Money, channel: Channel) -> Money
```

Saída boa:
```python
def get_shipping_voucher_discount(
    voucher: "Voucher", shipping_price: Money, channel: Channel
) -> Money:
    """Calculate discount value for a shipping voucher."""
    discount = voucher.get_discount_amount_for(shipping_price, channel)
    return min(discount, shipping_price)
```

Aqui usamos o few-shot para mostrar o estilo. Como disse anteriormente, hoje usamos o contexto para instruir o modelo sobre questões
como essa.

Perceba o uso de marcadores. Para o claude, inclusive, funciona muito bem tags html.

Markdown como *lingua franca*.


# Chain of thought

O conceito de ensinar o raciocínio, não a resposta.


- Regra real: percentual incide sobre o valor restante, nunca fica negativo; tarefa `applyPromotions`.

Prompt ruim: "Escreva uma função em TypeScript que aplica uma lista de promoções (percentual ou valor fixo) a um item e retorna o valor final."

Saída ruim:
```typescript
function applyPromotions(originalAmount: number, promotions: Promotion[]): number {
  let total = originalAmount;
  for (const promo of promotions) {
    if (promo.type === "percentage") {
      total -= originalAmount * (promo.value / 100); // usa o valor original, não o restante
    } else {
      total -= promo.value;
    }
  }
  return Math.max(total, 0);
}
```

R$ 200 com duas promoções de 10% daria R$162, não R$160.

Prompt bom:
```
<context>
Promoções são aplicadas em cascata sobre um item de carrinho.
</context>
<reasoning_steps>
1. Liste as regras de negócio de promoções em cascata.
2. Confira cada regra contra "o valor nunca fica negativo".
3. Só depois implemente.
</reasoning_steps>
<task>
Implemente applyPromotions(originalAmount, promotions), um comentário por regra.
</task>
```

Saída boa:
```typescript
function applyPromotions(originalAmount: number, promotions: Promotion[]): number {
  let remaining = originalAmount;
  for (const promo of promotions) {
    if (promo.type === "percentage") {
      remaining -= remaining * (promo.value / 100); // regra 1: incide sobre o valor restante
    } else {
      remaining -= promo.value;
    }
    remaining = Math.max(remaining, 0); // regra 2: nunca negativo
  }
  return remaining;
}
```

## Importante

O CoT foi um dos comportamentos que foram descobertos depois dos modelos serem lancados e deu tão certo que os provedores passaram a treinar modelos com eles.

"Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (NeurIPS 2022) [1]. Pesquisadores descobriram que, via prompting, os modelos melhoravam muito em tarefas de raciocínio se fossem instruídos a mostrar os passos intermediários antes da resposta final. 

"Large Language Models are Zero-Shot Reasoners" (NeurIPS 2022) [2] mostrou que nem precisa de exemplo, só pedir "vamos pensar passo a passo" já melhora. Isso é tipicamente chamado de zero-shot CoT.

Pedir para incluir o "reasoning" ajuda bastante para futuras i(n)terações.

A OpenAI afirma explicitamente que treinou o modelo com reinforcement learning para usar e aperfeiçoar sua chain of thought, aprendendo estratégias como decompor problemas, detectar/corrigir erros e tentar abordagens alternativas [3].

Outro relato "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning" [4].


## Contrato + Contraste

- Regra real: ordem de aplicação (buyget antes de standard, decrescente dentro do tipo).

Prompt ruim: "Refatore essa função de aplicar promoções para ficar mais legível, separe em funções menores."

Saída ruim:
```typescript
function sortPromotions(promotions: Promotion[]): Promotion[] {
  return [...promotions].sort((a, b) => a.code.localeCompare(b.code)); // muda a ordem de aplicação
}
```

- O bug passa despercebido sem teste que combine múltiplas promoções.

Prompt bom:
```
<constraints>
A ordem de aplicação (buyget antes de standard, decrescente dentro do tipo) é comportamento, não pode mudar.
</constraints>
<example label="aceitável">
... extrai nomes e formatação, não toca a ordenação ...
</example>
<example label="não fazer">
... reordena por código alfabético ...
</example>
<task>
Refatore para ficar mais legível, mantendo o contrato acima.
</task>
```

## Técnicas mais avançadas

Self-consistence. 3 CoTs. Paper google [5].

Tree of thoughts. Paper google e princeton [6]. Na prática: [7].

Skeleton of Thought (Esqueleto de Pensamento). É mais para otimizar tempo, não qualidade [8].

ReAct [9].

## Diretrizes

> "Como eu pediria para um dev sem experiência/contexto?"

Markdown é a lingua franca.

Mostre. Se ele ficou consufo, o modelo ficará. Essa é a golden rule da Anthropic.


Seja claro. Direto. 
    - Sobre formato de saída, sobre possíveis passos, restrições, guardrails etc.

Adicione contexto.

Adicione exemplos (```<example> </example>```)

Coloque arquivos longos no início. (Lost in the Middle)

Dê tempo (inclusive é técnica de otimização time/inference scaling): "Read everything and Think hard before answering"

Decomponha em tarefas menores.

Ref: [11]


## Mais a frente

Planeje! Itere! Valide!

As técnicas e diretrizes são a base para os frameworks que estão sendo adotados: SDD, RPI, RePPIT, etc.

De novo:

> Há um movimento natural de shift do prompting manual para formas mais estruturadas de especificação, engenharia de contexto e configuração do agente.


É isso que vamos ver em detalhes nos próximos capítulos.

# Referências

[1] Wei et al. "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (NeurIPS 2022). https://arxiv.org/abs/2201.11903

[2] Kojima et al. "Large Language Models are Zero-Shot Reasoners" (NeurIPS 2022). https://arxiv.org/abs/2205.11916

[3] OpenAI. "Learning to reason with LLMs". https://openai.com/index/learning-to-reason-with-llms

[4] DeepSeek-AI. "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning". https://arxiv.org/abs/2501.12948

[5] Wang et al. "Self-Consistency Improves Chain of Thought Reasoning in Language Models". https://arxiv.org/abs/2203.11171

[6] Yao et al. "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (Princeton e Google DeepMind). https://arxiv.org/abs/2305.10601

[7] Implementação prática de Tree of Thoughts prompting. https://github.com/dave1010/tree-of-thought-prompting

[8] Ning et al. "Skeleton-of-Thought: Prompting LLMs for Efficient Parallel Generation". https://arxiv.org/abs/2307.15337

[9] ReAct — técnica de prompting. https://www.promptingguide.ai/techniques/react

[10] Prompt Engineering Guide — Techniques. https://www.promptingguide.ai/techniques

[11] Claude — Prompt engineering best practices. https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices





