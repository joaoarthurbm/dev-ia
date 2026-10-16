+++
title = "Jonhnny Weslley (NexHealth)"
date = 2026-10-16
draft = false
description = "Jonhnny Weslley, dev senior na NexHealth, fala sobre sessões de grilling com o agente, por que o workflow importa mais que o modelo, validação automática de SQL via hooks e os riscos de instalar skills sem ler o que elas fazem."
youtube = "https://youtu.be/rBFsKgc90Jw"
empresa = "NexHealth"
aula = ""

[[convidados]]
  nome = "Jonhnny Weslley"
  cargo = "Desenvolvedor Senior"
  link = ""
+++

***

Conversa com o Jonhnny Weslley, dev senior na NexHealth, sobre como o time dele usa agentes de código no dia a dia. A apresentação de Jonhnny está disponível no youtube. Aqui eu vou escrever um pouco sobre o que discutimos e colocar alguns shorts que selecionei para a discussão.

# O que aprendemos

* Uma sessão de "grilling" com o agente te entrevistando antes de implementar não é só para extrair requisitos. É o que fixa a decisão <strong>na sua cabeça</strong> e na do agente. Uma espécie de onboarding para a tarefa.

* Delegar decisão para o agente sem esse processo ganha produtividade, mas tem um custo: conhecimento que deveria ficar com você fica só no código, porque foi o agente que decidiu sem te envolver.

* O que separa um time que usa bem agentes de um que não usa não é o modelo ou o AI code agent escolhido, é o workflow. quebrar tarefas, especificar antes de implementar, ter roteiro de teste e sensores bem definidos. A parte determinística — revisão e controle de qualidade — continua na mão de quem desenvolve.

* Segurança de agente também é parte do workflow: a NexHealth construiu uma ferramenta que valida toda consulta SQL antes do agente executá-la no banco, bloqueando qualquer coisa que vaze dado sensível ou peça acesso a credenciais (AWS, por exemplo) sem motivo. Tudo implementado via hooks.

* Nunca instale uma skill da web sem ler o que ela executa, mesmo vindo de fonte confiável. O pacote pode ter sido comprometido no caminho. Ataques recentes ao GitHub miravam exatamente isso.

# Trechos da conversa

## Grilling: o agente te interrogando antes de implementar

<video controls preload="metadata" class="clip" src="/industria/jonhnny/grill.mp4"></video>

Aqui eu defendo sessões de "grilling". O ganho não é só alinhar o agente com o que você quer: o processo força você a pensar mais profundamente no problema, e esse conhecimento fica com você, não só com quem (ou o que) implementou. Delegar essa etapa sem passar por ela dá produtividade, mas tira de você decisões que talvez devessem ficar na sua cabeça. 

## Workflow importa mais que o modelo

<video controls preload="metadata" class="clip" src="/industria/jonhnny/workflow.mp4"></video>

Não adianta ter o melhor modelo ou o melhor AI code agent se você não quebra as tarefas, não define especificação antes de implementar e não tem roteiro de teste com sensores bem definidos. Monitorar e avaliar é fundamental nessa nova forma de desenvolver.

## Validação automática de SQL antes de qualquer consulta

<video controls preload="metadata" class="clip" src="https://youtu.be/rBFsKgc90Jw"></video>

O time da NexHealth criou uma ferramenta que intercepta toda consulta SQL que o agente tentar executar no banco. Antes de rodar, ela valida se aquela consulta vazaria alguma informação que não deveria — se vazar, o comando é bloqueado. O mesmo mecanismo serve para bloquear pedidos sem sentido, como o agente tentar ler credenciais da AWS no meio de uma tarefa que não precisa disso. Implementado com hooks.

## O perigo de instalar skills sem ler o que elas fazem

<video controls preload="metadata" class="clip" src="/industria/jonhnny/perigos.mp4"></video>

Alerta direto: não confie cegamente em skills disponíveis na web, mesmo as que prometem reduzir seu trabalho em 90%. Algumas, no fim do código, fazem scan de chaves de AWS ou GitHub e enviam para um endereço externo. E mesmo uma skill de fonte confiável pode ter sido comprometida durante o download. Jonhnny cita os ataques recentes ao GitHub, em que invasores conseguiram permissão para dar push em repositórios justamente para injetar comandos maliciosos que executam quando alguém baixa a skill.
