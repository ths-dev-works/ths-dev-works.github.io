---
title: "Roadmap: Intensivo de 48 Horas para a Certificação Ultimate Go"
date: 2026-01-31
tags: ["Go", "Golang", "ArdanLabs", "Carreira"]
description: "Meu plano de estudo de 2 dias para dominar e passar no exame de 100 questões da Ardan Labs."
---

A certificação "Ultimate Go" da Ardan Labs não é sobre sintaxe; é sobre **Simpatia Mecânica**—entender como o software respeita o hardware. Para passar com 80% de acerto em 90 minutos, estou a seguir este roadmap de imersão de 2 dias.

## Dia 1: Simpatia Mecânica & Semântica de Dados
*Objetivo: Entender como cada byte se move na memória.*

### Manhã: Ponteiros e Stack/Heap (Peso: ~25%)
- [X] **Stack vs. Heap:** Entender isolamento vs. partilha.
- [X] **Análise de Escape:** Identificar o que dispara alocações no heap (ponteiros, interfaces).
- [X] **Leitura:** [Stacks e Ponteiros](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html)
- [X] **Leitura:** [Análise de Escape](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html)
- [X] **Leitura:** [Memory Profiling](https://www.ardanlabs.com/blog/2017/06/language-mechanics-on-memory-profiling.html)
- [X] **Prática:**
  - [X] Vá ao [Repositório "gotraining" da Ardan](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-the-memory-model.html) e execute os exemplos de ponteiros com a flag -gcflags="-m" para ver a análise de escape.
  - [X] Escrever uma função que retorna um ponteiro para uma variável local e outra que retorna o valor.
  - [X] Executar `go build -gcflags="-m"` para verificar qual delas escapa para o heap.
  - [X] Testar: Colocar uma variável dentro de uma `interface{}` causa escape? (Spoiler: Sim).
  - [X] Criar uma função que retorna um ponteiro e usar `go tool pprof` para identificar a alocação no heap.
  
### Tarde: Layout de Dados & Performance (Peso: ~20%)
- [X] **Mecânica de Slices:** Ponteiro, Comprimento, Capacidade.
- [X] **Linhas de Cache:** Porquê memória contígua (slices) supera listas ligadas.
- [X] **Leitura:** [Slices em Go](https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html)
- [X] **Leitura:** [Design Orientado a Dados](https://www.ardanlabs.com/blog/2017/06/design-philosophy-on-data-and-semantics.html)
- [X] **Prática:** 
  - [X] Criar uma slice com `make([]int, 0, 5)`. **Usar** um ciclo para adicionar 10 itens.
  - [X] Imprimir o `len`, `cap`, e o endereço de memória do primeiro elemento a cada passo.
  - [X] Observar exatamente quando o endereço de memória muda (a realocação do "array de suporte"). 
---

## Dia 2: Concorrência, Design & Integridade
*Objetivo: Gerir sistemas complexos e falhas.*

### Manhã: O Scheduler & Canais (Peso: ~35%)
- [X] **O Modelo G-M-P:** Como o scheduler gere Goroutines em threads do SO.
- [X] **Tabela de Estado de Canais:** Memorizando comportamento para canais nil, abertos e fechados.
- [X] **Semântica do Pacote Context:** Cancelamento adequado de goroutines e gestão de timeouts.
- [X] **Leitura:** [Scheduling em Go - Parte I](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html)
- [X] **Leitura:** [Scheduling em Go - Parte II](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)
- [X] **Leitura:** [Scheduling em Go - Parte III](https://www.ardanlabs.com/blog/2018/12/scheduling-in-go-part3.html)
- [X] **Leitura:** [O Comportamento de Canais](https://www.ardanlabs.com/blog/2017/10/the-behavior-of-channels.html)
- [X] **Leitura:** [Semântica do Pacote Context](https://www.ardanlabs.com/blog/2019/09/context-package-semantics-in-go.html)
- [X] **Prática:** 
  - [X] Escrever um programa que vaza uma goroutine (um sender bloqueado num canal não bufferizado sem receiver).
  - [X] Corrijir usando um bloco `select` com `context.WithTimeout`.
  - [X] Praticar o padrão "Fan-out": 10 goroutines a realizar uma tarefa e a reportar para um único canal coletor.
  - [X] Criar um handler de servidor que usa context para timeout de request e propagação de cancelamento.
  
### Tarde: Filosofia de Design (Peso: ~20%)
- [X] **Poluição de Interfaces:** Descobrir interfaces, não projetá-las.
- [X] **Integridade de Erros:** Tratar erros como valores.
- [X] **Leitura:** [Interface Semântica](https://www.ardanlabs.com/blog/2017/07/interface-semantics.html)
- [X] **Leitura:** [Interface Values Are Valueless](https://www.ardanlabs.com/blog/2018/03/interface-values-are-valueless.html)
- [X] **Leitura:** [Filosofia de Tratamento de Erros I](https://www.ardanlabs.com/blog/2014/10/error-handling-in-go-part-i.html)
- [X] **Leitura:** [Filosofia de Tratamento de Erros II](https://www.ardanlabs.com/blog/2014/11/error-handling-in-go-part-ii.html)
- [X] **Leitura:** [Methods, Interfaces and Embedded Types](https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html)
- [X] **Prática:**
  - [X] Criar um struct `User` concreto. Implemente uma interface `Printer` apenas **depois** de ter uma função que precisa dela. (Descoberta sobre Design).
  - [X] Criar um tipo de erro customizado que envolve outro erro usando `fmt.Errorf("... %w", err)`.
  - [X] **Usar** `errors.As` para recuperar o erro customizado e os seus campos.
---

## Regras Chave "Cheat Sheet" para o Exame
1. **Nunca** usar um ponteiro para uma interface.
2. **Nunca** iniciar uma goroutine sem saber como ela vai parar (prevenir vazamentos).
3. **Registar** o erro OU **retornar** o erro. **Nunca fazer** ambos.
4. **Semântica de Valor** = Isolamento/Stack. **Semântica de Ponteiro** = Partilha/Heap.

## Recursos Adicionais
- **Repositório GitHub**: [Ardan Labs Go Training](https://github.com/ardanlabs/gotraining) - Exemplos de código fonte completos e exercícios para todos os tópicos cobertos neste roadmap.
