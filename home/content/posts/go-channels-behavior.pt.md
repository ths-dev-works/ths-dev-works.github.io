---
title: "Go Mechanics: O Comportamento dos Canais"
date: 2026-02-02
tags: ["Go", "Concorrência", "Canais", "Goroutines", "Sinalização"]
description: "Compreender os canais Go como mecanismos de sinalização para orquestrar goroutines e escrever melhor código concorrente."
---

Em Go, os canais são uma das funcionalidades mais poderosas para programação concorrente. No entanto, muitos desenvolvedores cometem o erro de pensar nos canais como estruturas de dados ou filas. A chave para dominar os canais é pensá-los como **mecanismos de sinalização** que permitem que as goroutines comuniquem-se sobre eventos.

Este post resume as principais ideias de William Kennedy sobre o comportamento dos canais da Ardan Labs, focando nos três atributos fundamentais da sinalização: Garantia de Entrega, Estado e Com/Sem Dados.

## A Filosofia da Sinalização

### De Estrutura de Dados para Mecanismo de Sinalização
Ao trabalhar com canais, esqueça a sua estrutura interna e foque no seu comportamento. Um canal permite que uma goroutine sinalize outra goroutine sobre um evento específico. Esta mentalidade de sinalização ajudá-lo-á a escrever código concorrente melhor e mais preciso.

### Os Três Atributos de Sinalização
1. **Garantia de Entrega**: Precisa de confirmação de que um sinal foi recebido?
2. **Estado**: Qual é o estado atual do canal (nil, aberto, fechado)?
3. **Com ou Sem Dados**: Está a sinalizar com dados ou apenas com o sinal?

## Garantia de Entrega

A questão fundamental: "Preciso de uma garantia de que o sinal enviado por uma determinada goroutine foi recebido?"

Considere este exemplo básico:
```go
go func() {
    p := <-ch // Receber
}()

ch <- "papel" // Enviar
```

A goroutine que envia precisa de saber que "papel" foi recebido antes de continuar? A sua resposta determina que tipo de canal usar.

### Figura 1: Garantia de Entrega

```
┌─────────────────────────────────────────────────────────┐
│                 GARANTIA DE ENTREGA                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  O remetente precisa saber que o sinal foi recebido?    │
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐             │
│  │   SIM           │    │   NÃO           │             │
│  │                 │    │                 │             │
│  │ Canal Não       │    │ Canal           │             │
│  │ Bufferizado     │    │ Bufferizado (>1)│             │
│  │                 │    │                 │             │
│  │ Receber         │    │ Enviar          │             │
│  │ acontece ANTES  │    │ acontece ANTES  │             │
│  │ do Envio        │    │ do Receber      │             │
│  │ completar       │    │                 │             │
│  │                 │    │                 │             │
│  │ 100% Garantia   │    │ Sem Garantia    │             │
│  │                 │    │                 │             │
│  └─────────────────┘    └─────────────────┘             │
│                                                         │
│  ┌─────────────────┐                                    │
│  │ BUFFERIZADO = 1 │                                    │
│  │                 │                                    │
│  │ Garantia        │                                    │
│  │ Atrasada        │                                    │
│  │                 │                                    │
│  │ Sinal anterior  │                                    │
│  │ garantido       │                                    │
│  │ recebido        │                                    │
│  │                 │                                    │
│  └─────────────────┘                                    │
└─────────────────────────────────────────────────────────┘
```

### Tipos de Canais e Garantias

#### Canais Não Bufferizados - Entrega Garantida
```go
ch := make(chan string)
```
- **Garantia**: Sinal enviado foi recebido
- **Mecanismo**: Receber acontece ANTES do Envio completar
- **Caso de uso**: Quando precisa saber que o sinal foi recebido

#### Canais Bufferizados (>1) - Sem Garantia
```go
ch := make(chan string, 10)
```
- **Garantia**: Nenhuma garantia de receção
- **Mecanismo**: Enviar acontece ANTES do Receber
- **Caso de uso**: Quando não precisa de confirmação imediata

#### Canais Bufferizados (=1) - Garantia Atrasada
```go
ch := make(chan string, 1)
```
- **Garantia**: Sinal anterior foi recebido
- **Mecanismo**: Receber do primeiro sinal acontece ANTES do segundo Envio completar
- **Caso de uso**: Quando precisa de garantia com um sinal de atraso

## Estados dos Canais

Os canais existem em três estados possíveis, cada um com comportamentos distintos:

### Figura 2: Estados dos Canais

```
┌──────────────────────────────────────────────────────────┐
│                    ESTADOS DOS CANAIS                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐  ┌───────────┐ │
│  │      NIL        │  │      ABERTO     │  │  FECHADO  │ │
│  │                 │  │                 │  │           │ │
│  │ var ch chan T   │  │ ch := make()    │  │ close(ch) │ │
│  │ ch = nil        │  │                 │  │           │ │
│  │                 │  │                 │  │           │ │
│  │ Envio & Receber │  │ Envio & Receber │  │ Não mais  │ │
│  │ BLOQUEIAM para  │  │ Normais         │  │ Envios    │ │
│  │ sempre          │  │                 │  │           │ │
│  │                 │  │                 │  │ Recebes   │ │
│  │                 │  │                 │  │ ainda     │ │
│  │                 │  │                 │  │ funcionam │ │
│  │                 │  │                 │  │           │ │
│  └─────────────────┘  └─────────────────┘  └───────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Canal Nil
```go
var ch chan string  // Valor zero
ch = nil            // Nil explícito
```
- **Comportamento**: Qualquer Envio ou Recebe bloqueia para sempre
- **Caso de uso**: Desativar sinalização, limitação de taxa

### Canal Aberto
```go
ch := make(chan string)
```
- **Comportamento**: Operações normais de Envio e Receber
- **Caso de uso**: Comunicação ativa entre goroutines

### Canal Fechado
```go
close(ch)
```
- **Comportamento**: Não são permitidos mais Envios, Recebes ainda funcionam
- **Caso de uso**: Sinalizar conclusão, não há mais dados a chegar

## Sinalização Com Dados

Quando sinaliza com dados, normalmente está:
- Pedindo a uma goroutine para iniciar uma nova tarefa
- Obtendo um resultado de volta de uma goroutine

### Figura 3: Sinalização Com Dados

```
┌───────────────────────────────────────────────────────────┐
│                SINALIZAÇÃO COM DADOS                      │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────┐ │
│  │  NÃO            │  │ BUFFERIZADO (>1) │  │ BUFFERIZ  │ │
│  │  BUFFERIZADO    │  │                  │  │  ADO (=1) │ │
│  │                 │  │                  │  │           │ │
│  │ ch <- "papel"   │  │ ch <- "papel"    │  │ ch <- "p" │ │
│  │                 │  │                  │  │           │ │
│  │ Receber         │  │ Enviar           │  │ Envio do  │ │
│  │ acontece ANTES  │  │ acontece ANTES   │  │ primeiro  │ │
│  │ do Envio        │  │ do Receber       │  │ sinal     │ │
│  │ completar       │  │                  │  │ acontece  │ │
│  │                 │  │                  │  │ ANTES do  │ │
│  │ GARANTIA        │  │ SEM GARANTIA     │  │ segundo   │ │
│  │                 │  │                  │  │ Envio     │ │
│  │                 │  │                  │  │           │ │
│  │                 │  │                  │  │ GARANTIA  │ │
│  │                 │  │                  │  │ ATRASADA  │ │
│  └─────────────────┘  └──────────────────┘  └───────────┘ │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Cenário 1: Esperar por Tarefa (Entrega Garantida)
Gestor contrata funcionário e espera até estar pronto para lhe dar trabalho:

```go
func waitForTask() {
    ch := make(chan string)

    go func() {
        p := <-ch  // Funcionário espera pelo papel
        
        // Funcionário realiza o trabalho aqui
        // Funcionário terminou e pode ir embora
    }()

    time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
    ch <- "papel"  // Gestor envia trabalho quando está pronto
}
```

**Pontos Chave**:
- Funcionário bloqueia à espera de trabalho
- Gestor tem garantia que o funcionário recebeu o papel
- Latência desconhecida para ambas as partes

### Cenário 2: Esperar por Resultado (Entrega Garantida)
Gestor contrata funcionário para trabalhar imediatamente e espera pelo resultado:

```go
func waitForResult() {
    ch := make(chan string)

    go func() {
        time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
        ch <- "papel"  // Funcionário envia resultado
        
        // Funcionário terminou e pode ir embora
    }()

    p := <-ch  // Gestor espera pelo resultado
}
```

**Pontos Chave**:
- Funcionário começa a trabalhar imediatamente
- Gestor bloqueia à espera do resultado
- Funcionário tem garantia que o gestor recebeu o resultado

### Cenário 3: Fan Out (Sem Garantia)
Gestor contrata equipa, cada um trabalha independentemente:

```go
func fanOut() {
    emps := 20
    ch := make(chan string, emps)  // Bufferizado para todos os resultados

    for e := 0; e < emps; e++ {
        go func() {
            time.Sleep(time.Duration(rand.Intn(200)) * time.Millisecond)
            ch <- "papel"  // Envio não bloqueia
        }()
    }

    for emps > 0 {
        p := <-ch
        fmt.Println(p)
        emps--
    }
}
```

**Pontos Chave**:
- Nenhuma garantia de quando os resultados serão recebidos
- Funcionários não bloqueiam ao enviar resultados
- Tamanho do buffer deve ser calculado com base nas restrições

### Cenário 4: Drop (Sem Garantia)
Gestor descarta trabalho quando o funcionário está na capacidade:

```go
func selectDrop() {
    const cap = 5
    ch := make(chan string, cap)

    go func() {
        for p := range ch {
            fmt.Println("funcionário : recebeu :", p)
        }
    }()

    const work = 20
    for w := 0; w < work; w++ {
        select {
        case ch <- "papel":
            fmt.Println("gestor : envio confirmado")
        default:
            fmt.Println("gestor : descartado")
        }
    }

    close(ch)
}
```

**Pontos Chave**:
- Trabalho é descartado quando o buffer está cheio
- Sem pressão de retrocesso na submissão de trabalho
- Usa `select` com `default` para envios não bloqueantes

### Cenário 5: Esperar por Tarefas (Garantia Atrasada)
Gestor envia múltiplas tarefas para funcionário único com buffer de 1:

```go
func waitForTasks() {
    ch := make(chan string, 1)

    go func() {
        for p := range ch {
            fmt.Println("funcionário : trabalhando :", p)
        }
    }()

    const work = 10
    for w := 0; w < work; w++ {
        ch <- "papel"
    }

    close(ch)
}
```

**Pontos Chave**:
- Buffer de 1 fornece garantia atrasada
- Tarefa anterior garantida de ser recebida antes do próximo envio
- Reduz latência mantendo garantias

## Sinalização Sem Dados

Quando sinaliza sem dados, normalmente está:
- Dizendo a uma goroutine para parar o que está a fazer
- Reportando conclusão sem resultado
- Sinalizando encerramento

### Figura 4: Sinalização Sem Dados

```
┌────────────────────────────────────────────────────────┐
│               SINALIZAÇÃO SEM DADOS                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌─────────────────┐  ┌──────────────────┐             │
│  │   NÃO           │  │   BUFFERIZADO    │             │
│  │   BUFFERIZADO   │  │                  │             │
│  │                 │  │                  │             │
│  │ close(ch)       │  │ close(ch)        │             │
│  │                 │  │                  │             │
│  │ Close acontece  │  │ Close acontece   │             │
│  │ ANTES do Receber│  │ ANTES do Receber │             │
│  │                 │  │                  │             │
│  │ Perfeito para   │  │ Cheiro de código │             │
│  │ cancelamento    │  │ para cancelamento│             │
│  │                 │  │                  │             │
│  └─────────────────┘  └──────────────────┘             │
│                                                        │
│  Use pacote context para cancelamento                  │
│  Usa canal não bufferizado internamente                │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### A Operação Close
```go
close(ch)
```

**Benefícios**:
- Uma única goroutine pode sinalizar múltiplas goroutines de uma vez
- Perfeito para cenários de cancelamento e encerramento
- Qualquer receive em canal fechado não bloqueia

### Melhor Prática: Usar Pacote Context
```go
// Abordagem preferida
ctx, cancel := context.WithCancel(context.Background())
go func() {
    select {
    case <-ctx.Done():
        // Lidar com cancelamento
    }
}()
cancel()  // Sinalizar sem dados
```

### Canal de Cancelamento Manual
Se implementar o seu próprio cancelamento:
```go
ch := make(chan struct{})  // Canal de sinalização de espaço zero
close(ch)  // Sinalizar cancelamento
```

**Importante**: Use `chan struct{}` para canais apenas de sinalização - é a escolha idiomática de espaço zero.

### Cenário 6: Cancelamento Baseado em Context
Usando pacote context para cancelamento baseado em timeout:

```go
func withTimeout() {
    duration := 50 * time.Millisecond

    ctx, cancel := context.WithTimeout(context.Background(), duration)
    defer cancel()

    ch := make(chan string, 1)

    go func() {
        time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
        ch <- "papel"
    }()

    select {
    case p := <-ch:
        fmt.Println("trabalho completo", p)

    case <-ctx.Done():
        fmt.Println("a continuar")
    }
}
```

**Pontos Chave**:
- Context usa canal não bufferizado internamente para cancelamento
- Canal bufferizado de 1 previne fugas de goroutines
- `select` espera por conclusão de trabalho ou timeout
- Chame sempre `cancel()` em defer para limpeza

## Diretrizes de Tamanho do Buffer

### A Regra "Sem Números Aleatórios"
Os tamanhos dos buffers nunca devem ser arbitrários. Devem ser calculados com base em restrições bem definidas.

### Questões Chave para Dimensionamento de Buffer
1. **Tenho uma quantidade bem definida de trabalho a ser completado?**
2. **Quanto trabalho em excesso me coloca na capacidade?**
3. **Se a minha goroutine não conseguir acompanhar, posso descartar novo trabalho?**
4. **Que nível de risco estou disposto a aceitar se o meu programa terminar inesperadamente?**

Se estas questões não fazem sentido para o seu caso de uso, usar um buffer maior que 1 provavelmente está errado.

## Resumo da Mecânica da Linguagem

### Canais Não Bufferizados
- **Comportamento**: Receber acontece antes do Envio
- **Benefício**: 100% de garantia que o sinal foi recebido
- **Custo**: Latência desconhecida de quando o sinal será recebido

### Canais Bufferizados
- **Comportamento**: Enviar acontece antes do Receber
- **Benefício**: Reduzir latência de bloqueio entre sinalizações
- **Custo**: Nenhuma garantia de quando o sinal foi recebido
- **Nota**: Buffer maior = menos garantia

### Fechar Canais
- **Comportamento**: Close acontece antes do Receber (como Bufferizado)
- **Uso**: Sinalizar sem dados, perfeito para cancelamentos

### Canais Nil
- **Comportamento**: Envio e Receber bloqueiam
- **Uso**: Desativar sinalização, limitação de taxa, paragens temporárias

## Filosofia de Design

### Quando Usar Canais Bufferizados > 1
Permitido apenas se:
- Consegue responder claramente às questões de dimensionamento de buffer
- Tem medições que mostram a necessidade
- Sabe exatamente o que acontece quando a goroutine que envia bloqueia

### Cheiros de Código a Evitar
- Tamanhos de buffer aleatórios
- Canais bufferizados para cancelamento (use `chan struct{}` em vez disso)
- Usar canais para sincronização simples estilo mutex
- Pensar nos canais como estruturas de dados em vez de sinais

## Considerações de Performance

### Análise Custo/Benefício
- **Entrega garantida** = custo de latência desconhecida
- **Sem garantia** = potencial perda de dados no término do programa
- **Tamanho do buffer** = troca entre throughput e uso de memória

### Impacto de Memória
- Cada elemento bufferizado consome memória
- Buffers maiores = mais pressão de memória
- Considere o que acontece no crash do programa

## Pontos Chave

1. **Pense em sinais, não dados**: Canais são para sinalização entre goroutines
2. **Escolha garantias sabiamente**: Não bufferizados para garantias, bufferizados para throughput
3. **O estado importa**: nil, aberto e fechado têm comportamentos diferentes
4. **Buffer com propósito**: Nunca use tamanhos de buffer aleatórios
5. **Sinalize sem dados**: Use `close()` ou `context` para cancelamento
6. **Questione o seu design**: Está realmente a usar canais pelas razões certas?

## Em Resumo

- **Canais orquestram goroutines** através de sinalização, não partilha de dados
- **Três atributos definem comportamento**: Garantia, Estado e Com/Sem-Dados
- **Canais não bufferizados fornecem garantias** ao custo de latência desconhecida
- **Canais bufferizados fornecem throughput** ao custo de garantias
- **Fechar canais sinaliza conclusão** e permite padrões de cancelamento
- **Tamanhos de buffer devem ser calculados** com base em restrições bem definidas
- **Pense em termos de sinalização** para escrever melhores programas concorrentes em Go

---

*Baseado em ["The Behavior Of Channels"](https://www.ardanlabs.com/blog/2017/10/the-behavior-of-channels.html) de William Kennedy da Ardan Labs.*
