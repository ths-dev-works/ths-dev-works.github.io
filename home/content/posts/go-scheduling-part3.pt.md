---
title: "Go Scheduling: Parte III - Concorrência"
date: 2026-01-31
tags: ["Go", "Scheduling", "Concorrência", "Performance", "Workloads"]
description: "Compreender quando e como usar concorrência efetivamente em Go com diferentes tipos de workload."
---

Este post explora concorrência em Go, focando em quando usá-la e como diferentes tipos de workload afetam decisões de performance.

## O que é Concorrência

### Definição
- **Concorrência**: Execução "fora de ordem" de instruções que normalmente correriam sequencialmente
- **Paralelismo**: Executar duas ou mais instruções simultaneamente
- **Insight Chave**: Concorrência não é o mesmo que paralelismo

### Concorrência vs Paralelismo
```
┌─────────────────────────────────────────────────────────────┐
│                Concorrência vs Paralelismo                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Paralelismo (2 Cores):                                     │
│  ┌─────────────────────────┐       ┌──────────────────────┐ │
│  │ P1 (Core 1)             │       │ P2 (Core 2)          │ │
│  │ ┌─────┐                 │       │ ┌─────┐              │ │
│  │ │ G1* │ ← A Executar    │       │ │ G2* │ ← A Executar │ │
│  │ └─────┘                 │       │ └─────┘              │ │
│  └─────────────────────────┘       └──────────────────────┘ │
│                                                             │
│  Concorrência (Single Core):                                │
│  ┌─────────────────────────┐                                │
│  │ P1 (Core 1)             │                                │
│  │ ┌─────┐ ┌─────┐ ┌─────┐ │                                │
│  │ │ G1* │ │ G2  │ │ G3  │ │ ← A tomar turnos a partilhar   │
│  │ └─────┘ └─────┘ └─────┘ │   o mesmo core                 │
│  └─────────────────────────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Quando Usar Concorrência
- **Valor**: Deve fornecer ganho de performance suficiente para justificar custo de complexidade
- **Viabilidade**: Execução fora de ordem deve ser possível e fazer sentido
- **Teste**: Começar com solução sequencial, depois considerar concorrência

## Tipos de Workload

### Workloads CPU-Bound
**Definição**: Trabalho que nunca cria estados de espera - cálculos constantes
- **Exemplos**: Cálculos matemáticos, calcular Pi, processamento de dados
- **Características**: 
  - Sem estados de espera naturais
  - Requer paralelismo para concorrência efetiva
  - Mais goroutines que cores podem abrandar execução
  - Context switches criam eventos "Stop The World"

### Workloads IO-Bound
**Definição**: Trabalho que causa estados de espera naturais
- **Exemplos**: Requisições de rede, I/O de ficheiros, chamadas ao SO, eventos de sincronização
- **Características**:
  - Estados de espera naturais permitem partilha eficiente
  - Não requer paralelismo para concorrência efetiva
  - Mais goroutines que cores podem acelerar execução
  - Context switches não criam eventos "Stop The World"

### Matriz de Decisão de Workload
```
┌──────────────────────────────────────────────────────────────┐
│                    Análise de Workload                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CPU-Bound:                                                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ • Precisa de paralelismo (goroutines ≤ cores)           │ │
│  │ • Context switches são caros                            │ │
│  │ • Demasiadas goroutines = performance mais lenta        │ │
│  │ • Exemplo: Cálculos matemáticos                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  IO-Bound:                                                   │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ • Não precisa de paralelismo (goroutines > cores OK)    │ │
│  │ • Context switches são eficientes                       │ │
│  │ • Mais goroutines = melhor performance                  │ │
│  │ • Exemplo: Leitura de ficheiros, chamadas de rede       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Exemplos Práticos

### CPU-Bound: Adicionar Números
**Versão Sequencial** (5 linhas):
```go
func add(numbers []int) int {
    var v int
    for _, n := range numbers {
        v += n
    }
    return v
}
```

**Versão Concorrente** (26 linhas):
```go
func addConcurrent(goroutines int, numbers []int) int {
    var v int64
    totalNumbers := len(numbers)
    lastGoroutine := goroutines - 1
    stride := totalNumbers / goroutines

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for g := 0; g < goroutines; g++ {
        go func(g int) {
            start := g * stride
            end := start + stride
            if g == lastGoroutine {
                end = totalNumbers
            }

            var lv int
            for _, n := range numbers[start:end] {
                lv += n
            }

            atomic.AddInt64(&v, int64(lv))
            wg.Done()
        }(g)
    }

    wg.Wait()
    return int(v)
}
```

**Análise**:
- **Tipo de Workload**: CPU-Bound (matemática pura)
- **Benefício da Concorrência**: Sim (pode dividir trabalho)
- **Paralelismo Requerido**: Sim (goroutines ≤ cores)
- **Custo de Complexidade**: Alto (5→26 linhas)

### CPU-Bound: Bubble Sort
**Versão Sequencial**: Algoritmo bubble sort padrão
**Versão Concorrente**: Ordena chunks concorrentemente, depois reordena lista inteira

**Análise**:
- **Tipo de Workload**: CPU-Bound
- **Benefício da Concorrência**: Não (caro combinar resultados)
- **Problema**: Após ordenação concorrente, ainda precisa ordenar completamente
- **Resultado**: Adiciona complexidade sem benefício de performance

### IO-Bound: Ler Ficheiros
**Versão Sequencial**:
```go
func find(topic string, docs []string) int {
    var found int
    for _, doc := range docs {
        items, err := read(doc)
        if err != nil {
            continue
        }
        for _, item := range items {
            if strings.Contains(item.Description, topic) {
                found++
            }
        }
    }
    return found
}
```

**Versão Concorrente**: Usa worker pool com channel
```go
func findConcurrent(goroutines int, topic string, docs []string) int {
    var found int64

    ch := make(chan string, len(docs))
    for _, doc := range docs {
        ch <- doc
    }
    close(ch)

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for g := 0; g < goroutines; g++ {
        go func() {
            var lFound int64
            for doc := range ch {
                items, err := read(doc)
                if err != nil {
                    continue
                }
                for _, item := range items {
                    if strings.Contains(item.Description, topic) {
                        lFound++
                    }
                }
            }
            atomic.AddInt64(&found, lFound)
            wg.Done()
        }()
    }

    wg.Wait()
    return int(found)
}
```

**Resultados de Performance**:
- **Sem Paralelismo**: ~87-88% mais rápido que sequencial
- **Com Paralelismo**: Nenhum ganho adicional de performance
- **Insight Chave**: Concorrência sozinha fornece grandes benefícios para trabalho IO-bound

## Pontos Chave

### Framework de Decisão
1. **Começar Sequencial**: Sempre começar com solução sequencial funcional
2. **Identificar Workload**: Determinar se CPU-Bound ou IO-Bound
3. **Avaliar Concorrência**: O trabalho pode ser dividido e combinado eficientemente?
4. **Considerar Paralelismo**: Precisa de múltiplos cores?

### Diretrizes de Workload
- **CPU-Bound**: 
  - Usar concorrência apenas se trabalho pode ser dividido eficientemente
  - Limitar goroutines ao número de cores
  - Cuidado com overhead de context switching

- **IO-Bound**:
  - Concorrência quase sempre benéfica
  - Pode usar mais goroutines que cores
  - Context switches são eficientes devido à espera natural

### Sinais de Alerta
- **Combinação de Resultados Cara**: Como requisito de reordenação do bubble sort
- **Sem Estados de Espera Naturais**: Trabalho CPU-bound sem paralelismo
- **Complexidade vs Benefício**: 5→26 linhas de código para ganho mínimo

### Indicadores de Sucesso
- **Estados de Espera Naturais**: I/O de ficheiro, chamadas de rede, sincronização
- **Divisão Eficiente de Trabalho**: Limites claros para processamento concorrente
- **Combinação Simples de Resultados**: Fácil juntar resultados concorrentes

## Conclusão

Concorrência efetiva em Go requer compreender seu tipo de workload:

- **Workloads IO-Bound** beneficiam grandemente de concorrência sem precisar de paralelismo
- **Workloads CPU-Bound** precisam de paralelismo e gestão cuidadosa de goroutines
- **Alguns algoritmos** (como bubble sort) não são adequados para concorrência apesar de serem CPU-bound

A chave é identificar quando execução "fora de ordem" fornece valor real que justifica o custo de complexidade. Sempre comece sequencial, analise seu workload, e tome decisões informadas sobre concorrência e paralelismo.

---

*Baseado em ["Scheduling In Go : Part III - Concurrency"](https://www.ardanlabs.com/blog/2018/12/scheduling-in-go-part3.html) de William Kennedy da Ardan Labs.*
