# Diario-de-bordo1
## O Dilema do Servidor em Nuvem

### Introdução

A CloudData utiliza um servidor de núcleo único para atender dois tipos diferentes de tarefas: requisições rápidas da interface web e processos mais pesados de geração de relatórios financeiros. Como apenas um processo pode utilizar o núcleo da CPU por vez, a forma como o Sistema Operacional organiza essa execução interfere diretamente no tempo de resposta da aplicação. Neste estudo de caso, são analisados o funcionamento das chamadas de sistema, o comportamento do escalonamento FCFS e uma possível alternativa para melhorar a responsividade do servidor.

### Chamadas de Sistema e Acesso ao Hardware

Os programas executados pelo usuário não possuem acesso direto ao hardware do computador. Por isso, quando o processo responsável pela geração do relatório precisa ler informações armazenadas no disco, ele precisa solicitar esse serviço ao Sistema Operacional por meio de uma **chamada de sistema, ou System Call**. As chamadas de sistema funcionam como uma interface entre as aplicações e o núcleo do Sistema Operacional. Em uma operação de leitura, por exemplo, pode ser utilizada uma chamada como `read()`.

Durante sua execução normal, a aplicação funciona em **Modo Usuário**, que possui acesso limitado aos recursos do computador. Quando uma chamada de sistema é realizada, a execução passa temporariamente para o **Modo Kernel**, permitindo que o Sistema Operacional execute a operação com os privilégios necessários. Depois que o serviço é realizado, o controle retorna para a aplicação, que continua sua execução em Modo Usuário.

Caso a leitura do disco não possa ser concluída imediatamente, o processo pode ficar bloqueado aguardando a operação de entrada e saída. Nesse período, a CPU pode ser utilizada por outro processo que esteja pronto para executar.

### Diagnóstico do Escalonamento

O algoritmo **FCFS (First-Come, First-Served)** organiza a execução de acordo com a ordem de chegada dos processos. Dessa forma, quem chega primeiro à fila é atendido primeiro. Esse funcionamento é simples, porém pode causar problemas quando existem processos com tempos de execução muito diferentes.

No caso da CloudData, a geração de um relatório pode exigir bastante processamento. Se esse processo começar a utilizar a CPU antes das requisições rápidas da interface web, essas requisições terão que aguardar. Como a interface depende de respostas rápidas, esse tempo de espera pode ser percebido pelos usuários como travamento ou congelamento da página. Esse comportamento pode estar relacionado ao chamado **efeito comboio**, no qual processos menores ficam aguardando atrás de um processo mais demorado.

O FCFS também é considerado um algoritmo **não preemptivo**. Isso significa que o Sistema Operacional não retira a CPU de um processo apenas porque outro processo chegou à fila. Depois que um processo começa a executar, ele normalmente continua até terminar sua etapa de processamento ou até ficar bloqueado aguardando algum recurso, como uma operação de entrada e saída.

Como o servidor da CloudData possui apenas um núcleo, um processo mais pesado pode ocupar a CPU por um período maior e atrasar diretamente os processos interativos. Na prática, o escalonamento é controlado pelo próprio Sistema Operacional e pelo seu kernel. Sistemas como o Linux possuem diferentes políticas de escalonamento para tratar processos com características distintas. Por isso, o uso do FCFS no estudo de caso pode ser entendido como uma forma didática de analisar esse comportamento.

### Proposta de Melhoria

Entre as opções SJF, SRTN e Round-Robin, o **Round-Robin** é uma alternativa adequada para melhorar a responsividade da interface. Nesse algoritmo, cada processo recebe a CPU durante um pequeno intervalo de tempo chamado **quantum**. Se o processo não terminar dentro desse período, ele é interrompido e colocado novamente na fila, permitindo que outro processo utilize o processador.

Com isso, um processo mais pesado, como a geração de relatório, não permanece utilizando a CPU por um período muito longo de forma contínua. No cenário da CloudData, esse funcionamento permite que as requisições da interface web tenham oportunidades de execução com maior frequência, melhorando o tempo de resposta percebido pelos usuários.

O **SJF** também favorece processos menores, mas depende de uma estimativa do tempo de execução das tarefas. O **SRTN** utiliza uma lógica parecida, porém de maneira preemptiva, escolhendo o processo que possui o menor tempo restante de execução. O Round-Robin se adapta bem ao cenário apresentado porque distribui o tempo da CPU entre os processos e é bastante relacionado a sistemas interativos, nos quais o tempo de resposta é importante.

### Prioridades, Starvation e Aging

Outra possibilidade seria utilizar um algoritmo baseado em prioridades, dando maior prioridade às requisições da interface web. Apesar de melhorar o atendimento dos processos interativos, essa estratégia pode causar um problema para os processos de geração de relatórios.

Se processos de prioridade maior continuarem chegando à fila, um processo de prioridade mais baixa pode permanecer esperando por muito tempo. Essa situação é conhecida como **Starvation**, ou inanição. No caso da CloudData, isso poderia acontecer com os relatórios caso as requisições da interface fossem sempre tratadas com prioridade máxima.

Uma forma de evitar esse problema é utilizar o mecanismo de **Aging, ou envelhecimento**. Com o Aging, a prioridade de um processo aumenta gradualmente conforme o tempo que ele permanece esperando. Assim, mesmo um processo que começou com prioridade baixa pode alcançar uma prioridade maior e conseguir utilizar a CPU. Dessa maneira, o sistema consegue favorecer as tarefas interativas sem deixar os processos de relatório esperando indefinidamente.

### Pesquisa Multimídia

A pesquisa foi realizada utilizando materiais em diferentes formatos para complementar os conteúdos trabalhados em aula. O material de Carlos Maziero apresenta conceitos relacionados às chamadas de sistema, ao funcionamento dos modos usuário e kernel e aos principais algoritmos de escalonamento de processos. O vídeo sobre escalonamento de processos auxilia na compreensão de algoritmos como FCFS, SJF, Round-Robin, prioridades e SRT, permitindo comparar suas principais características. O episódio do Café Debug complementa a pesquisa abordando temas relacionados a Sistemas Operacionais, threads, paralelismo, concorrência, uso da CPU e troca de contexto. Esses conceitos ajudam a compreender como diferentes tarefas podem disputar recursos de processamento dentro de uma aplicação.

### Conclusão

O caso da CloudData mostra como a política de escalonamento pode influenciar diretamente o funcionamento de uma aplicação. O FCFS possui uma estrutura simples, mas pode causar tempos de espera maiores quando processos demorados são executados antes de tarefas rápidas. Em um servidor de núcleo único, esse problema fica ainda mais perceptível, pois apenas um processo pode utilizar a CPU por vez.

Para esse cenário, o Round-Robin representa uma alternativa que distribui o tempo do processador entre os processos e melhora as oportunidades de execução das tarefas interativas. Também é possível perceber a importância das chamadas de sistema, que permitem que aplicações solicitem serviços ao Sistema Operacional de forma controlada. Além disso, quando são utilizadas prioridades, mecanismos como Aging ajudam a evitar que processos de menor prioridade fiquem esperando por tempo indefinido.

### Referências

MAZIERO, Carlos Alberto. **Sistemas operacionais: conceitos e mecanismos**. Curitiba: Editora da UFPR, 2019. 456 p. Disponível em: https://wiki.inf.ufpr.br/maziero/lib/exe/fetch.php?media=socm:socm-livro.pdf. Acesso em: 1 out. 2026.

CARVALHO, Sidartha. **Sistemas operacionais: Parte 1 – Escalonamento de processos**. YouTube, 10 ago. 2020. 1 vídeo. Disponível em: https://www.youtube.com/watch?v=weKa9H88bjY. Acesso em: 1 out. 2026.

CAFÉ DEBUG. **#167 Threads, Paralelismo e SO na Prática para Devs**. Café Debug seu podcast de tecnologia, 14 jul. 2025. Podcast, 1 h 08 min. Disponível em: https://cafedebug.com.br/detalhes-epis%C3%B3dio?guid=16eec9ae-a6c4-4d28-9919-a773ee3c8738. Acesso em: 1 out. 2026.
