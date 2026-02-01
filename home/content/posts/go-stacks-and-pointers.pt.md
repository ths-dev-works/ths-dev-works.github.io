---
title: "Mecânica Go: Stacks, Ponteiros e Limites de Frame"
date: 2026-01-31
tags: ["Go", "Internos", "Memória"]
---

Em Go, "Simpatia Mecânica" começa com a compreensão de onde os seus dados vivem. Aqui está um resumo abrangente dos princípios fundamentais de Bill Kennedy sobre Stacks e Ponteiros da série Ardan Labs.

## Limites de Frame: A Fundação

Cada função executa dentro de **stack frames** - espaços de memória privados que fornecem isolamento e contexto. Quando uma função é chamada, Go faz a transição entre frames, copiando dados "por valor" através do limite.

### Conceitos Chave:
- **Stack Frames**: Cada função obtém a sua própria sandbox de memória
- **Pass-by-Value**: Dados são copiados entre frames (WYSIWYG - O Que Vê É O Que Obtém)
- **Limites de Frame**: Funções têm acesso direto apenas à sua própria memória de frame

## Semântica de Valor vs. Semântica de Ponteiro

### 1. Semântica de Valor (Isolamento)
Quando se passa dados por valor, Go cria uma **cópia**. A função opera na sua própria versão, garantindo isolamento completo.

```go
func increment(inc int) {
    inc++ // Afeta apenas a cópia
}
```

### 2. Semântica de Ponteiro (Partilha)
Um ponteiro permite que uma função "atravesse" limites de frame para o frame de outra função. Isto é **partilha**.

```go
func increment(inc *int) {
    *inc++ // Afeta o valor original
}
```

## A Mecânica do Stack

### Alocação do Stack
- Cada goroutine começa com ~2KB de espaço de stack
- Frames são tomados para baixo no stack (detalhe de implementação)
- Memória abaixo do frame ativo é inválida
- Memória do frame ativo e acima é válida

### Processo de Chamada de Função
1. **Criação de Frame**: Novo frame de stack é alocado
2. **Transferência de Dados**: Valores são copiados através do limite de frame
3. **Execução**: Função opera dentro do seu frame
4. **Retorno**: Frame torna-se inválido (mas a memória não é limpa)

### Autolimpeza do Stack
- Quando uma função retorna, o seu frame torna-se inválido
- Memória é deixada intacta (sem custo de limpeza)
- Memória do stack é limpa a cada nova chamada de função
- Todos os valores são inicializados com zero values

## Visualizações do Stack

### Figura 1: Frame Único (função main)
```
┌─────────────────────┐ ← Memória válida
│    frame main       │
│                     │
│  count = 10         │
│  (end: 0x10429fa4)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

### Figura 2: Chamada de Função (main chama increment)
```
┌─────────────────────┐ ← Memória válida
│    frame main       │
│                     │
│  count = 10         │
│  (end: 0x10429fa4)  │
├─────────────────────┤
│  frame increment    │
│                     │
│  inc = 10           │
│  (end: 0x10429f98)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

### Figura 3: Após execução do increment
```
┌─────────────────────┐ ← Memória válida
│    frame main       │
│                     │
│  count = 10         │
│  (end: 0x10429fa4)  │
├─────────────────────┤
│  frame increment    │
│                     │
│  inc = 11           │
│  (end: 0x10429f98)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

### Figura 4: Após retorno da função
```
┌─────────────────────┐ ← Memória válida (main está ativo)
│    frame main       │
│                     │
│  count = 10         │
│  (end: 0x10429fa4)  │
├─────────────────────┤ ← Memória inválida (frame increment)
│  frame increment    │  (deixado intacto)
│                     │
│  inc = 11           │
│  (end: 0x10429f98)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

### Figura 5: Partilha de Ponteiro (passar endereço)
```
┌─────────────────────┐ ← Memória válida
│    frame main       │
│                     │
│  count = 10         │
│  (end: 0x10429fa4)  │
├─────────────────────┤
│  frame increment    │
│                     │
│  inc = 0x10429fa4   │ ← Aponta para count
│  (end: 0x10429f98)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

### Figura 6: Após dereferência do ponteiro
```
┌─────────────────────┐ ← Memória válida
│    frame main       │
│                     │
│  count = 11         │ ← Modificado via ponteiro
│  (end: 0x10429fa4)  │
├─────────────────────┤
│  frame increment    │
│                     │
│  inc = 0x10429fa4   │ ← Continua a apontar para count
│  (end: 0x10429f98)  │
└─────────────────────┘
┌─────────────────────┐ ← Memória inválida
│                     │
└─────────────────────┘
```

## Tipos de Ponteiro e Mecânica

### Regras de Tipo de Ponteiro
Para cada tipo declarado, obtém-se um tipo de ponteiro gratuito:
- `int` → `*int`
- `User` → `*User`
- Todos os ponteiros são 4-8 bytes (tamanho de endereço)

### Variáveis de Ponteiro Não São Especiais
- São variáveis regulares com alocação de memória
- Armazenam valores de endereço
- O caractere `*` serve duplo propósito: declaração de tipo e operador de dereferência

## A Regra de Ouro dos Ponteiros

> **Se a palavra "partilhar" não sair da sua boca, não precisa de um ponteiro.**

Ponteiros servem a UM propósito: partilhar um valor com uma função para que possa ler/escrever nesse valor mesmo que não exista diretamente dentro do seu próprio frame.

## Padrões de Acesso à Memória

### Acesso Direto (Semântica de Valor)
```go
count := 10
increment(count)      // Passar valor (cópia)
// count permanece 10
```

### Acesso Indireto (Semântica de Ponteiro)
```go
count := 10
increment(&count)     // Passar endereço (partilha)
// count torna-se 11
```

## Pontos Chave

1. **Go é 100% Pass-by-Value**: Até ponteiros são copiados (copia-se o endereço)
2. **O Stack é Autolimpante**: Sem envolvimento de GC para memória de stack
3. **Ponteiros São para Partilha**: Se não está a partilhar dados através de limites, fique com valores
4. **Limites de Frame Impõem Isolamento**: Funções só podem aceder diretamente ao seu próprio frame
5. **Acesso Indireto à Memória**: Ponteiros permitem acesso à memória entre frames
6. **Segurança de Tipos**: Tipos de ponteiro garantem que apenas endereços compatíveis podem ser partilhados

## Em Resumo

- **Funções executam dentro do âmbito de limites de frame** que fornecem um espaço de memória individual para cada função respetiva
- **Quando uma função é chamada**, ocorre uma transição entre dois frames
- **O benefício de passar dados "por valor" é a legibilidade**
- **O stack é importante** porque fornece o espaço de memória físico para os limites de frame dados a cada função individual
- **Toda a memória de stack abaixo do frame ativo é inválida** mas a memória do frame ativo e acima é válida
- **Fazer** uma chamada de função significa que a goroutine precisa de enquadrar uma nova secção de memória no stack
- **É durante cada chamada de função, quando o frame é tomado**, que a memória do stack para esse frame é limpa
- **Ponteiros servem um propósito**, partilhar um valor com uma função para que a função possa ler e escrever nesse valor mesmo que o valor não exista diretamente dentro do seu próprio frame
- **Para cada tipo declarado**, seja por si ou pela própria linguagem, obtém gratuitamente um tipo de ponteiro complementar que pode usar para partilha
- **A variável de ponteiro permite acesso indireto à memória** fora do frame da função que a está a usar
- **Variáveis de ponteiro não são especiais** porque são variáveis como qualquer outra variável. Têm uma alocação de memória e detêm um valor

---

*Este resumo baseia-se em ["Language Mechanics On Stacks And Pointers"](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html) de Bill Kennedy da Ardan Labs, parte de uma série de quatro artigos cobrindo ponteiros, stacks, heaps, análise de escape e semântica de valor/ponteiro em Go.*
