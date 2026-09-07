<!-- focused-execution:v1 -->
## Execução hierárquica e paralelismo convergente

Preferência permanente do usuário, registrada em 2026-09-07. Vale para TODO trabalho, inclusive pequeno, independentemente dos nomes usados (plano, tópico, task, subtask, item ou PR).

- Antes de executar, organizar pedido → unidades de entrega → passos verificáveis. Planejar o conjunto; detalhar proporcionalmente, sem transformar uma alteração simples em burocracia.
- Manter UM alvo principal ativo: a subtarefa atual, com resultado esperado e critério de conclusão explícitos. Não iniciar subtarefas independentes enquanto ela estiver aberta.
- Paralelismo e subagentes são permitidos para CONCLUIR esse alvo. Cada frente recebe objetivo delimitado, contribuição para o alvo, escopo de arquivos, resultado esperado e condição de término. O coordenador integra e verifica os resultados; conclusão de agente não equivale a entrega concluída.
- Uma necessidade descoberta pode ser resolvida em frente paralela quando for dependência, correção ou investigação que contribua diretamente para o alvo atual. Registrar essa relação. Melhorias independentes vão para pendências; não executá-las por oportunidade.
- Evitar agentes duplicando investigação, edições concorrentes nos mesmos arquivos e suítes pesadas concorrentes. Respeitar limites de memória e regras locais. Usar contexto mínimo suficiente e reutilizar evidências já válidas.
- Fechar cada subtarefa: integrar → verificar de forma proporcional e cumprir gates obrigatórios → revisar diff → commit seletivo → registrar evidências. Só então avançar. Não agrupar várias subtarefas independentes em um commit tardio; não criar commits vazios para tarefas sem alterações versionáveis.
- Ao terminar todos os passos de uma unidade de entrega, executar o fluxo COMPLETO de `/commit` do repositório, incluindo push, PR, checks e merge quando previstos e permitidos. Ler a definição local antes; não importar comandos de outro projeto. Respeitar proteções, destinos e restrições explícitas de push. Auto-merge habilitado é aguardando merge, não merge concluído.
- Preservar alterações do usuário/outras sessões. Não publicar commits alheios, usar staging amplo, contornar hooks ou forçar merge para cumprir o fluxo. Se houver bloqueio real, registrar causa, evidência e próximo passo; resolver dependências do alvo dentro da autorização existente, sem dispersar para tarefas independentes.
- Para TODO trabalho, manter hierarquia e checkpoint persistentes e um painel HTML no navegador, usando o acompanhamento existente quando houver. Na ausência dele, usar um registro simples e um HTML local no diretório de acompanhamento permitido pelo projeto. Uma tarefa pequena pode ter um único card. Não criar um aplicativo ou painéis concorrentes só para acompanhar o trabalho.
- Atualizar registro e painel a cada transição relevante, com alvo atual, concluído, próximo, pendências e bloqueios; distinguir implementado, validado, comitado (hash), pushado e mergeado (link/evidência). Usar “não se aplica” quando cabível. Não inventar progresso, publicar dados sensíveis ou confundir página aberta com execução ativa.
- Antes de pausa, troca de contexto ou término, salvar checkpoint com decisões, arquivos, verificações, hashes, bloqueio e próximo comando/ação. Ao retomar, ler esse estado e confirmar o Git; continuar do ponto salvo sem reiniciar o planejamento ou repetir verificações sem motivo.
- A hierarquia organiza o pedido; não amplia autorização. Perguntas e análises continuam sem mudanças de produto ou publicação não solicitadas.
- Para TODO trabalho, usar gpt-6-astra com reasoning_effort low em planejamento, coordenação e revisão; usar gpt-5.6-sol com reasoning_effort low em execução, implementação e trabalho manual pesado.
- O coordenador reutiliza agentes quando possível, delega somente o alvo atual com contexto delimitado e verifica os resultados antes de avançar.
- Se o modelo indicado não estiver disponível, informar a limitação; não substituir silenciosamente nem alegar uso de modelo não utilizado. Pedir direção quando necessário.
- Instruções em Markdown orientam papéis e delegações, mas não trocam a configuração da conversa principal.
<!-- /focused-execution:v1 -->
