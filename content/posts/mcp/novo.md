# Contexto

Qual foi a grande revolução no uso de LLMs para tarefas? A capacidade de executar ferramentas. Vimos isso na contextualização e na aula sobre anatomia dos agentes, que são os responsáveis de fato por executar ferramentas.
 LLM só gera texto. Já vimos isso. 

E como são executadas as ferramentas? Pensa bem. O LMM diz "preciso ler o arquivo X" ou até de forma mais específica exec readFile("X"). Como o agente pode executar? Vamos ver 3 maneiras, da mais simplória e problemática para a forma como se faz hoje em dia.

A primeira é simplesmente fazer parse do texto gerado pelo LLM. Funciona em muitos casos, mas esse parse pode dar errado porque a string pode não ser a esperada, uma vez que o LLM é um gerador estocástico e não-determinístico.

Uma evolução seria pedir ao LLM que gere a chamada em formato estruturado. Um json, por exemplo. Isso melhora o cenário, mas ainda pode ser gerado um json que não seja bem estruturado. No final das contas, essa abordagem aplica uma melhoria no prompt para tentar controlar a saída do LLM, mas ainda sim não podemos controlar via pedido.

O que é feito então? Para as ferramentas nativas o esquema já é pré-determinado e o modelo não precisa inventar como executar uma operação: recebe schemas/descrições das ferramentas e produz uma chamada estruturada. Por exemplo, o esquema abaixo define como chamar a ferramenta de ler arquivo:

{
  "name": "files.read",
  "description": "Read content from a known file or range",
  "input_schema": {
    "type": "object",
    "properties": {
      "file_id": {
        "type": "string"
      },
      "start_line": {
        "type": "integer"
      },
      "end_line": {
        "type": "integer"
      }
    },
    "required": ["file_id"]
  }
}

É assim que os *AI Coding Agents* funcionam para as chamadas relacionadas às ferramentas "nativas". O que estou
chamando de ferramentas nativas aqui são as relacionadas a arquivos e shell, Search, WebSearch, Bash, SendFile, SendMessage entre outras. São ferramentas que os provedores já possuem implementadas. Ferramentas de prateleira.

Mas essa solução não é só um pouco mais sofisticada que a segunda? Ainda sim, com esquema e exemplo o LMM, por ser não-determinístico, pode gerar o texto de uma execução de forma errada, não é?

Sim. E ainda podemos ir além. Uma chamada por ser sintaticamente válida, mas não semânticamente válida. Temos, no final das contas, 3 níveis de verificação:

1. Sintaticamente válida — JSON/formato correto.
2. Schema-valid — argumentos respeitam os tipos, enumerações, campos obrigatórios etc.
3. Semanticamente correta — a chamada realmente corresponde ao que o usuário pediu.

O esquema resolve deterministicamente a parte sintática, retornando erro quando não houver consistência, mas não resolve as outras.

A boa notícia é que o harness usa de outros mecanismos para controlar ainda mais. Em geral, um pipeline validação de schema -> validação semântica -> guardrail -> autorização -> execução.


# Aumentando a capacidade de agentes com ferramentas

Ótimo. Entendi que se houver esquema e um bom harness, o agente consegue executar as ferramentas nativas sem maiores problemas. E vem fazendo isso bem. Mas a gente viu que isso é feito para as ferramentas nativas que os devs dos provedores implementam, certo? Mas tenho algumas perguntas:

1. E se eu quiser fazer uma ferramenta e quiser que o opencode consiga usar essa minha ferramenta?
2. E se eu quiser que qualquer agente consiga usar essa ferramenta?


Vamos tentar abordar essas perguntas por caso de uso. O primeiro é: fiz um programa/script python que resolve algo e quero que o meu agente seja capaz de executar ele. Nesse caso, você pode registrar ele como uma ferramenta personalizada disponível no seu AI Coding Agent que ele vai saber invocar. Veja, por exemplo, como fazer isso no OpenCode: https://opencode.ai/docs/custom-tools. Nele você usa o helper  `tool()` que fica responsável por gerar o esquema e lidar com segurança de tipo e validação.

Uma vez que você usou o helper `tool()` para o seu código, isso virou uma ferramenta. Agora basta registrar nas ferramentas disponíveis (em .opencode/tools/ ou ~/.config/opencode/tools/.) que o OpenCode vai saber executá-la.

O segundo é: eu quero que esse pedaço de código que eu fiz seja passível de ser utilizado não somente pelo agente do opencode, mas por qualquer AI Coding Agent. Mais, por qualquer agente. 

Nesse caso, assim como fiz com opencode, eu poderia usar as biblitecas do claude para isso, não? Pode. Se você tem uma aplicação e quer ensinar o claude a lidar com ela, você pode escrever um adapter para o claude. Depois, se você quiser que o CodeX saiba lidar com ela, você pode escrever um adapter para o CodeX. Já viu onde vamos chegar, né? Para cada provedor, você teria que escrever um adapter. Para N provedores e M aplicações, teríamos NxM adapters.

A solução para o problema apontado acima é, então, um protocolo que todos saibam falar. Esse protocolo é o MCP.

MCP define uma forma padrão de comunicação de um agente com ferramentas. Se você desenvolver um servidor MCP para a sua API, por exemplo, os agentes serão capazes de se comunicar com ela. E o número de integrações cai de N×M para N+M.



# Problemas


Isto é, eu quero ampliar a capacidade dos agentes para além de apenas utilizarem ferramentas nativas. 

Bom,


uma primeira solução é passar a documentação da sua API no contexto. O modelo teria informação sobre os recursos e como invocá-los. Isso pode funcionar, mas temos algumas grandes problemas com essa solução.

- Doc inteira consome muito contexto.
- Chamada gerada livre a cada vez pode gerar erro.
- Reuso: aprendizado via contexto não persiste entre sessões.























