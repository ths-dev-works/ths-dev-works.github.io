---
title: "Go Mechanics: Entendendo Slices e Internals"
date: 2026-02-01
tags: ["Go", "Internals", "Memória", "Estruturas de Dados"]
---

Em Go, slices são uma das estruturas de dados mais poderosas e frequentemente utilizadas. Elas fornecem uma forma flexível e eficiente de trabalhar com sequências de dados. Aqui está um resumo abrangente da mecânica de slices baseado no blog oficial do Go e insights da Ardan Labs.

## Arrays: A Fundação

Antes de entender slices, devemos entender arrays. Arrays em Go são coleções de tamanho fixo com o tamanho sendo parte do tipo.

### Propriedades dos Arrays
- **Tamanho Fixo**: `[4]int` e `[5]int` serem tipos diferentes e incompatíveis
- **Tipo de Valor**: Arrays serem valores, não ponteiros para o primeiro elemento
- **Layout de Memória**: Elementos dispostos sequencialmente na memória
- **Semântica de Cópia**: Atribuição criar uma cópia completa do array

```go
// Declarações de array
var a [4]int
a[0] = 1
i := a[0] // i == 1

// Literais de array
b := [2]string{"Penn", "Teller"}
c := [...]string{"Penn", "Teller"} // Compilador conta elementos
```

## Slices: A Abstração

Slices constroem sobre arrays para fornecer flexibilidade e poder. Uma slice é um **descritor** de um segmento de array.

### Especificação de Tipo de Slice
```go
[]T // Onde T é o tipo do elemento
```

### Métodos de Criação de Slices

#### 1. Literais de Slice
```go
letters := []string{"a", "b", "c", "d"}
```

#### 2. Usando make()
```go
// make([]T, comprimento, capacidade)
s := make([]byte, 5, 5) // len=5, cap=5
s := make([]byte, 5)    // len=5, cap=5 (capacidade padrão é comprimento)
```

#### 3. Fatiando Arrays/Slices Existentes
```go
// De array
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // slice referenciando armazenamento do array

// De slice
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
slice := b[1:4] // []byte{'o', 'l', 'a'}, compartilha armazenamento
```

## Internals de Slices: A Estrutura de Três Partes

Uma slice consiste em três componentes:

1. **Ponteiro**: Referência ao array subjacente
2. **Comprimento**: Número de elementos na slice
3. **Capacidade**: Número máximo de elementos no array subjacente

### Visualização de Memória

```
┌─────────────────────────────────────────────────────────┐
│                    Cabeçalho da Slice                   │
├─────────────────┬─────────────────┬─────────────────────┤
│     Ponteiro    │   Comprimento   │     Capacidade      │
│   (0x10430000)  │       5         │         8           │
└─────────────────┴─────────────────┴─────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────┐
│                   Array Subjacente                      │
├─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┤
│ [0] │ [1] │ [2] │ [3] │ [4] │ [5] │ [6] │ [7] │ ... │
│ 10  │ 20  │ 30  │ 40  │ 50  │  0  │  0  │  0  │     │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
      ↑─────────────────↑─────────────────────────────────┐
      Elementos Acessíveis   Capacidade Reservada         │
      (Comprimento = 5)     (Capacidade = 8)              │
```

## Comprimento vs. Capacidade

### Comprimento (`len()`)
- Número de elementos atualmente acessíveis
- Sempre ≤ capacidade
- Usado para verificação de limites

### Capacidade (`cap()`)
- Espaço total disponível no array subjacente
- Determina quando realocação é necessária
- Pode ser maior que o comprimento

```go
s := make([]int, 3, 8) // len=3, cap=8
fmt.Printf("len=%d, cap=%d\n", len(s), cap(s))
// Saída: len=3, cap=8
```

## Operações e Comportamento de Slices

### Operações de Fatiamento
```go
// Slice original
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}

// Várias operações de fatiamento
b[:2]  // []byte{'g', 'o'}
b[2:]  // []byte{'l', 'a', 'n', 'g'}
b[:]   // []byte{'g', 'o', 'l', 'a', 'n', 'g'} (cópia do cabeçalho)
b[1:4] // []byte{'o', 'l', 'a'}
```

### Aviso de Armazenamento Compartilhado
Fatiamento **não** copia dados - cria um novo cabeçalho de slice apontando para o mesmo array:

```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:] // e == []byte{'a', 'd'}
e[1] = 'm' // e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'} - original é modificado!
```

## Crescendo Slices: Append e Realocação

### A Função Append
A função embutida `append` do Go gerencia crescimento de slices automaticamente:

```go
func append(s []T, x ...T) []T

// Uso básico
a := make([]int, 1) // []int{0}
a = append(a, 1, 2, 3) // []int{0, 1, 2, 3}

// Anexar outra slice
b := []string{"John", "Paul"}
c := []string{"George", "Ringo"}
b = append(b, c...) // []string{"John", "Paul", "George", "Ringo"}
```

### Estratégia de Crescimento
Quando a capacidade é insuficiente, `append`:
1. Aloca um novo array maior
2. Copia elementos existentes para o novo array
3. Adiciona novos elementos
4. Retorna um novo cabeçalho de slice

### Controle Manual de Crescimento
Para controle preciso sobre crescimento:

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    
    if n > cap(slice) {
        // Realocar: duplicar o necessário para crescimento futuro
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

## A Função Copy

A função embutida `copy` copia dados eficientemente entre slices:

```go
func copy(dst, src []T) int

// Uso
t := make([]byte, len(s), (cap(s)+1)*2)
copied := copy(t, s) // retorna número de elementos copiados
```

### Propriedades da Copy
- Copia até o mínimo de len(dst) e len(src)
- Lida corretamente com slices sobrepostas
- Mais eficiente que cópia manual elemento por elemento

## Padrões Comuns e Armadilhas

### 1. Slice Nula vs. Slice Vazia
```go
var s []byte     // slice nula, len=0, cap=0
t := []byte{}    // slice vazia, len=0, cap=0
u := make([]byte, 0) // slice vazia, len=0, cap=0

// Todas se comportam similarmente com append
s = append(s, 1) // funciona bem
```

### 2. Truncamento de Slice
```go
s := []int{1, 2, 3, 4, 5}
s = s[:3] // truncar para primeiros 3 elementos
// Array subjacente ainda contém todos 5 elementos
```

### 3. Slices Multidimensionais
```go
// Slice de slices
board := [][]int{
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9},
}
```

## Considerações de Performance

### Eficiência de Memória
- **Slices compartilham armazenamento**: Múltiplas slices podem referenciar o mesmo array
- **Copy-on-grow**: Apenas quando a capacidade é excedida
- **Cópia de cabeçalho**: Cabeçalhos de slice são pequenos (3 palavras)

### Performance de Cache
- **Memória contígua**: Melhor localidade de cache que listas ligadas
- **Acesso previsível**: Padrões de acesso sequencial são rápidos

### Padrões de Alocação
```go
// Bom: Pré-alocar capacidade conhecida
s := make([]int, 0, 1000) // Evita múltiplas realocações

// Menos eficiente: Deixar append crescer automaticamente
var s []int
for i := 0; i < 1000; i++ {
    s = append(s, i) // Pode causar múltiplas realocações
}
```

## Técnicas Avançadas de Slices

### 1. Filtragem com Append
```go
func Filter(s []int, fn func(int) bool) []int {
    var p []int // slice nula
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

### 2. Modificação In-place
```go
// Remover elementos preservando ordem
func Remove(slice []int, i int) []int {
    return append(slice[:i], slice[i+1:]...)
}

// Remover elementos sem preservar ordem (mais rápido)
func RemoveUnordered(slice []int, i int) []int {
    slice[i] = slice[len(slice)-1]
    return slice[:len(slice)-1]
}
```

### 3. Implementação de Stack
```go
type Stack []int

func (s *Stack) Push(v int) {
    *s = append(*s, v)
}

func (s *Stack) Pop() (int, bool) {
    if len(*s) == 0 {
        return 0, false
    }
    index := len(*s) - 1
    element := (*s)[index]
    *s = (*s)[:index]
    return element, true
}
```

## Pontos-Chave

1. **Slices são cabeçalhos**: Ponteiro + comprimento + capacidade, não os dados em si
2. **Armazenamento compartilhado**: Múltiplas slices podem referenciar o mesmo array subjacente
3. **Crescimento aciona realocação**: Quando comprimento excede capacidade
4. **Append é seu amigo**: Lida com a maioria dos cenários de crescimento automaticamente
5. **Copy para transferências explícitas**: Use quando precisa controle preciso
6. **Slices nulas são úteis**: Comece com nil e deixe append gerenciar alocação
7. **Planejamento de capacidade**: Pré-aloque quando conhece o tamanho para melhor performance

## Em Resumo

- **Slices fornecem uma janela** para um array subjacente através de um cabeçalho de três partes
- **Comprimento vs. Capacidade**: Comprimento é o que você pode acessar, capacidade é o que está disponível
- **Armazenamento compartilhado significa** modificações através de uma slice afetam todas as slices que compartilham aquele array
- **Append gerencia crescimento** alocando novos arrays quando necessário
- **Copy fornece eficiente** transferência de elementos entre slices
- **Performance vem de** memória contígua e cópia mínima
- **Pré-alocação previne** múltiplas realocações em cenários de crescimento

---

*Este resumo é baseado no post oficial do blog Go ["Go Slices: usage and internals"](https://go.dev/blog/slices-intro) e ["Understanding Slices in Go Programming"](https://www.ardanlabs.com/blog/2013/08/understanding-slices-in-go-programming.html) de William Kennedy da Ardan Labs.*
