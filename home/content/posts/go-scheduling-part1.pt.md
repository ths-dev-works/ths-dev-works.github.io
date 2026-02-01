---
title: "Go Scheduling: Parte I - Fundamentos do OS Scheduler"
date: 2026-01-31
tags: ["Go", "Scheduling", "Concorrência", "OS", "Performance"]
description: "Compreender os fundamentos do scheduler do sistema operacional que formam a base para o sistema de scheduling do Go."
---

Este post resume os fundamentos do scheduler do sistema operacional que fornecem a base para compreender o sistema de scheduling do Go.

## Básicos do OS Scheduler

### Threads e Execução
- **Threads** são "caminhos de execução" que correm independentemente com o seu próprio estado
- Cada programa cria um Processo com uma Thread inicial
- Decisões de scheduling acontecem ao nível da Thread, não do Processo
- Threads podem correr concorrentemente (à vez) ou em paralelo (simultaneamente em diferentes cores)

### Estados da Thread
1. **Waiting**: Parada à espera de hardware, chamadas ao SO ou sincronização
2. **Runnable**: Quer tempo de CPU para executar instruções
3. **Executing**: Atualmente num core a executar instruções de máquina

### Tipos de Trabalho
- **CPU-Bound**: Nunca entra no estado Waiting (cálculos constantes)
- **IO-Bound**: Causa estados Waiting (rede, base de dados, chamadas ao SO)

## Context Switching

### Scheduling Preemptivo
- Schedulers modernos são preemptivos (seleção de Threads imprevisível)
- Nunca escrever código baseado em comportamento percebido
- Controlar sincronização explicitamente para determinismo

### Impacto na Performance
- **Context switches são caros**: ~1000-1500 nanossegundos
- Custo: ~12k-18k instruções perdidas por switch
- **Trabalho IO-Bound**: Context switches ajudam (cores permanecem ocupados)
- **Trabalho CPU-Bound**: Context switches prejudicam (custo de latência pura)

## Princípios de Performance

### "Menos é Mais"
- Menos Threads Runnable = menos overhead, mais tempo por Thread
- Mais Threads Runnable = menos tempo por Thread, menos trabalho feito
- Equilibrar cores vs. Threads para throughput ótimo

### Coerência de Cache
- **Cache lines**: Blocos de 64 bytes trocados entre memória e caches
- **False sharing**: Múltiplas threads a aceder à mesma cache line causa problemas de performance
- Cada core obtém a sua própria cópia da cache line (semântica de valor no hardware)
- Invalidação de cache cria latência de acesso à memória (~100-300 ciclos de clock)

### Contexto Histórico
- Abordagem tradicional: Thread pools com ~3 threads por core
- Go elimina a necessidade de gestão manual de thread pools
- Schedulers do SO tomam decisões complexas de equilíbrio constantemente

## Pontos Chave

1. **Threads são caminhos de execução independentes** com o seu próprio estado
2. **Context switches são caros** e impactam a performance diferentemente baseado no tipo de trabalho
3. **Menos threads frequentemente performam melhor** que muitas threads
4. **Coerência de cache é crítica** para performance multithreaded
5. **Schedulers do SO tomam decisões complexas** equilibrando múltiplos fatores

Esta fundação é essencial para compreender como o scheduler do Go se baseia nestes fundamentos do SO na Parte II.

---

*Baseado em ["Scheduling In Go : Part I - OS Scheduler"](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html) de William Kennedy da Ardan Labs.*
