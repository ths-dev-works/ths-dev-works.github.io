---
title: "Go Scheduling: Parte II - Go Scheduler"
date: 2026-02-01
tags: ["Go", "Scheduling", "Concorrência", "Goroutines", "Performance"]
description: "Compreender a mecânica do scheduler do Go e como se baseia nos fundamentos do scheduler do SO."
---

Este post resume a mecânica do scheduler do Go e como fornece scheduling eficiente para goroutines.

## Arquitetura do Go Scheduler

### Componentes Core
- **Processadores Lógicos (P)**: Um por core virtual (lida com hyper-threading)
- **Threads do SO (M)**: Um por P, gerido pelo SO mas ligado ao P
- **Goroutines (G)**: Coroutines ao nível da aplicação, context-switched no M
- **Run Queues**: Local (LRQ) por P, Global (GRQ) para goroutines não atribuídas

### Visualização do Modelo G-M-P
```
┌─────────────────────────────────────────────────────────────┐
│                 Arquitetura do Go Scheduler                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  P1 (Processador Lógico)        P2 (Processador Lógico)     │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ LRQ (Fila Local)        │       │ LRQ (Fila Local)     │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐ ┌─────┐ ┌───┐│ │
│  │ │ G1  │ │ G2  │ │ G3  │ │       │ │ G4  │ │ G5  │ │G6 ││ │
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘ └─────┘ └───┘│ │
│  └─────────────────────────┘       └──────────────────────┘ │
│           │                               │                 │
│           ▼                               ▼                 │
│  ┌─────────────────────────┐       ┌─────────────────────┐  │
│  │ M1 (Thread do SO)       │       │ M2 (Thread do SO)   │  │
│  │ A Executar G1           │       │ A Executar G4       │  │
│  └─────────────────────────┘       └─────────────────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                   GRQ (Fila Global)                         │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐    │
│  │ G7  │ │ G8  │ │ G9  │ │ G10 │ │ G11 │ │ G12 │ │ G13 │    │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Inicialização
- Programa Go obtém P para cada core virtual (`runtime.NumCPU()`)
- Cada P obtém uma Thread do SO (M)
- Goroutine inicial começa a execução
- Todos os componentes trabalham juntos para scheduling eficiente

## Características do Scheduler

### Cooperativo vs Preemptivo
- **Scheduler do SO**: Preemptivo (ao nível do kernel, imprevisível)
- **Scheduler do Go**: Cooperativo (user-space, mas parece preemptivo)
- Scheduler do Go corre em user space acima do kernel
- Comportamento não determinístico como scheduler do SO
- Programador não controla decisões de scheduling

### Estados da Goroutine
Mesmos três estados que Threads do SO:
1. **Waiting**: Parada à espera de chamadas ao SO ou sincronização
2. **Runnable**: Quer tempo no M para executar instruções
3. **Executing**: Atualmente no M a executar instruções

## Context Switching

### Eventos de Scheduling
Quatro eventos permitem decisões do scheduler:
1. **Palavra-chave `go`**: Criar novas goroutines
2. **Garbage Collection**: Goroutines do GC precisam de tempo de execução
3. **Chamadas ao SO**: Podem causar que a goroutine bloqueie o M
4. **Sincronização**: Operações atómicas, mutex, canal

### Importância da Chamada de Função
- Context switching acontece em safe points de chamadas de função
- Loops apertadas sem chamadas de função causam latências no scheduler
- Go 1.12+ adiciona preempção não cooperativa para loops apertados

## Gestão de Chamadas ao SO

### Chamadas ao SO Assíncronas
- **Network Poller**: Lida com chamadas de rede assíncronas eficientemente
- Usa kqueue (MacOS), epoll (Linux), IOCP (Windows)
- Goroutine move para network poller, M permanece disponível
- M extra não necessário para operações de rede
- Reduz carga de scheduling do SO

### Fluxo de Chamada Assíncrona
```
┌─────────────────────────────────────────────────────────────┐
│              Fluxo de Chamada de Sistema Assíncrona         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Antes da Chamada de Rede:                                  │
│  ┌─────────────────────────┐                                │
│  │ P1                      │                                │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                                │
│  │ │ G1* │ │ G2  │ │ G3  │ │  ← G1 a executar no M1         │
│  │ └─────┘ └─────┘ └─────┘ │                                │
│  └─────────────────────────┘                                │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ M1 (Thread do SO)       │                                │
│  │ A Executar G1           │                                │
│  └─────────────────────────┘                                │
│                                                             │
│  Chamada de Rede Feita:                                     │
│  ┌─────────────────────────┐       ┌───────────────────────┐│
│  │ P1                      │       │ Network Poller        ││
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐               ││
│  │ │ G2* │ │ G3  │ │ G4  │ │       │ │ G1  │ ← G1 movida   ││
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘    para poller││
│  └─────────────────────────┘       └───────────────────────┘│
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ M1 (Thread do SO)       │                                │
│  │ Agora A Executar G2     │                                │
│  └─────────────────────────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Chamadas ao SO Síncronas
- **I/O de Ficheiros**: Não pode ser assíncrono, bloqueia M
- **CGO**: Pode bloquear M ao chamar funções C
- **Resposta do Scheduler**: Detacha M do P, traz novo M
- Goroutine bloqueada move de volta para LRQ quando chamada completa
- M original guardado para uso futuro

### Fluxo de Chamada Síncrona
```
┌────────────────────────────────────────────────────────────┐
│               Fluxo de Chamada de Sistema Síncrona         │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Antes da Chamada de I/O de Ficheiro:                      │
│  ┌─────────────────────────┐                               │
│  │ P1                      │                               │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                               │
│  │ │ G1* │ │ G2  │ │ G3  │ │  ← G1 a executar no M1        │
│  │ └─────┘ └─────┘ └─────┘ │                               │
│  └─────────────────────────┘                               │
│           │                                                │
│           ▼                                                │
│  ┌─────────────────────────┐                               │
│  │ M1 (Thread do SO)       │                               │
│  │ A Executar G1           │                               │
│  └─────────────────────────┘                               │
│                                                            │
│  Chamada de I/O Bloqueia M1:                               │
│  ┌─────────────────────────┐                               │
│  │ P1                      │                               │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                               │
│  │ │ G2* │ │ G3  │ │ G4  │ │  ← G2 agora a executar no M2  │
│  │ └─────┘ └─────┘ └─────┘ │                               │
│  └─────────────────────────┐                               │
│           │                                                │
│           ▼                                                │
│  ┌─────────────────────────┐       ┌─────────────────────┐ │
│  │ M2 (Thread do SO)       │       │ M1 (Bloqueado)      │ │
│  │ Agora A Executar G2     │       │ G1 a fazer I/O      │ │
│  └─────────────────────────┘       └─────────────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## Work Stealing

### Propósito
- Impede que M fique idle (evita context switches do SO)
- Balanceia goroutines através de todos os P's
- Mantém trabalho eficientemente distribuído

### Visualização de Work Stealing
```
┌─────────────────────────────────────────────────────────────┐
│                    Exemplo de Work Stealing                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Estado Inicial:                                            │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ P1                      │       │ P2                   │ │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │       │ ┌─────┐ ┌─────┐ ┌───┐│ │
│  │ │ G1* │ │ G2  │ │ G3  │ │       │ │ G4* │ │ G5  │ │G6 ││ │
│  │ └─────┘ └─────┘ └─────┘ │       │ └─────┘ └─────┘ └───┘│ │
│  └─────────────────────────┘       └──────────────────────┘ │
│                                                             │
│  GRQ: ┌─────┐                                               │
│       │ G7  │                                               │
│       └─────┘                                               │
│                                                             │
│  P1 Termina Trabalho:                                       │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ P1 (Vazio)              │       │ P2                   │ │
│  │                         │       │ ┌─────┐ ┌─────┐ ┌───┐│ │
│  │                         │       │ │ G4* │ │ G5  │ │G6 ││ │
│  │                         │       │ └─────┘ └─────┘ └───┘│ │
│  └─────────────────────────┘       └──────────────────────┘ │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────┐                                │
│  │ P1 Rouba do P2:         │                                │
│  │ ┌─────┐ ┌─────┐         │                                │
│  │ │ G5* │ │ G6  │         │  ← Metade roubada do P2        │
│  │ └─────┘ └─────┘         │                                │
│  └─────────────────────────┘                                │
│                                                             │
│  Estado Final Balanceado:                                   │
│  ┌─────────────────────────┐       ┌─────────────────────┐  │
│  │ P1                      │       │ P2                  │  │
│  │ ┌─────┐ ┌─────┐         │       │ ┌─────┐             │  │
│  │ │ G5* │ │ G6  │         │       │ │ G4* │             │  │
│  │ └─────┘ └─────┘         │       │ └─────┘             │  │
│  └─────────────────────────┘       └─────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Regras de Stealing
1. Verificar fila runnable global (1/61 do tempo)
2. Verificar fila local
3. Tentar roubar de outros P's (pegar metade)
4. Verificar fila global novamente
5. Poll de rede

### Benefícios
- Ms permanecem ocupados ("spinning")
- Melhor localidade de cache
- Reduzido overhead de scheduling do SO

## Vantagens de Performance

### Eficiência do Context Switch
- **Context switch de Thread do SO**: ~1000-1500 nanossegundos (~12k-18k instruções)
- **Context switch de Goroutine**: ~200 nanossegundos (~2.4k instruções)
- **Melhoria de 5-6x** no custo de context switching

### Visualização de Comparação de Performance
```
┌─────────────────────────────────────────────────────────────┐
│                Comparação de Performance                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Context Switch de Thread do SO:                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Tempo: ~1000-1500 nanossegundos                        │ │
│  │ Instruções Perdidas: ~12k-18k                          │ │
│  │ Cache Misses: Alto (a saltar entre cores)              │ │
│  │ Overhead do SO: Alto                                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  Context Switch de Goroutine:                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Tempo: ~200 nanossegundos                              │ │
│  │ Instruções Perdidas: ~2.4k                             │ │
│  │ Cache Misses: Baixo (mesmo core/thread)                │ │
│  │ Overhead do SO: Mínimo                                 │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  Ganho de Performance: 5-6x mais rápido em context switches │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Inovação Chave
Go transforma **Trabalho IO/Blocking em trabalho CPU-bound** ao nível do SO:
- Thread do SO nunca entra no estado Waiting
- Todo o context switching acontece ao nível da aplicação
- Melhor eficiência de cache-line
- Latência NUMA reduzida

### Impacto Prático
- Mesma Thread do SO e Core usadas para todo o processamento
- Sem perda de instruções para context switches do SO
- Mais trabalho feito com menos threads
- Carga reduzida no SO e hardware

## Pontos Chave

1. **Scheduler do Go é cooperativo mas parece preemptivo** - não determinístico como scheduler do SO
2. **Goroutines são threads ao nível da aplicação** com mesmos três estados que threads do SO
3. **Context switching é muito mais barato** que switching de threads do SO (~200ns vs ~1000ns)
4. **Network poller lida com chamadas assíncronas eficientemente** sem bloquear threads
5. **Work stealing impede cores idle** e balanceia carga através de processadores
6. **Go transforma trabalho blocking em trabalho CPU-bound** ao nível do SO para ganhos de eficiência maiores

Este design permite ao Go executar mais trabalho ao longo do tempo usando menos threads mais eficientemente, reduzindo carga de scheduling do SO mantendo alta concorrência.

---

*Baseado em ["Scheduling In Go : Part II - Go Scheduler"](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html) de William Kennedy da Ardan Labs.*
