---
title: "Go Mechanics: Valores de Interface São Sem Valor"
date: 2026-02-02
tags: ["Go", "Interfaces", "Polimorfismo", "Design Orientado a Dados", "Comportamento"]
description: "Compreender como as interfaces Go são sem valor e como permitem polimorfismo através do comportamento de dados concretos."
---

Em Go, as interfaces representam uma mudança fundamental do pensamento focado em implementação para o design focado em comportamento. A ideia crítica é que os valores de interface são "sem valor" - não têm significado concreto sem os dados armazenados dentro deles.

Este post resume as principais ideias de William Kennedy sobre a ausência de valor de interfaces da Ardan Labs, focando como este conceito permite polimorfismo adequado e design orientado a dados.

## Introdução: Foque no Comportamento, Não na Implementação

Ao desenhar com interfaces, a maioria dos programadores foca nos detalhes de implementação. No entanto, o poder real vem de compreender a relação que as interfaces têm com os dados concretos.

**Ideia principal**: Valores de interface são sem valor - só se tornam significativos quando dados concretos são armazenados dentro deles.

## Princípios de Design Orientado a Dados

### Primeira Lei do Design Orientado a Dados
> "Se não compreender os dados com que está a trabalhar, não compreende o problema que está a tentar resolver."

Cada problema que resolve é um problema de transformação de dados:
- **Entrada** → **Transformação** → **Saída**
- Funções são transformações de dados menores
- Algoritmos são baseados em dados concretos
- Simpatia mecânica vem de compreender dados concretos

### Segunda Lei do Design Orientado a Dados
> "Quando os dados estão a mudar, o seu problema está a mudar. Quando o problema está a mudar, então os algoritmos que escreveu precisam de mudar."

**O desafio**: Como permitir que os algoritmos permaneçam pequenos e precisos enquanto lidam com mudanças nos dados sem causar mudanças em cascata por toda a base de código?

**A solução**: Interfaces fornecem o mecanismo para desacoplamento comportamental.

## Dados Concretos: A Fundação

### Tipos Concretos com `struct`

```go
type file struct {
    name string
}

type pipe struct {
    name string
}
```

Estes são **tipos concretos** - definem dados reais que:
- Podem ser armazenados em memória
- Podem ser enviados através de redes
- Podem ser escritos em ficheiros
- Podem ser manipulados diretamente

```go
func main() {
    var f file  // Valor concreto de tipo file
    var p pipe  // Valor concreto de tipo pipe
    
    fmt.Println(f, p)  // { } { }
}
```

Tanto `f` como `p` são **valores reais** com estado concreto.

## Interfaces São Sem Valor

### Tipos de Interface Definem Comportamento

```go
type reader interface {
    read(b []byte) (int, error)
}
```

Um tipo de interface:
- Apenas declara um conjunto de métodos de comportamento
- Não tem nada concreto sobre ele
- Cria valores **sem valor**

```go
var r reader  // r é sem valor!
```

**Conceitos críticos**:
- Não há nada real sobre a variável `r`
- Não há nada concreto sobre a variável `r`
- A variável `r` é **sem valor**

### Figura 1: Tipos Concretos vs de Interface

```
┌─────────────────────────────────────────────────────────┐
│           TIPOS CONCRETOS vs DE INTERFACE               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │              TIPOS CONCRETOS                       │ │
│  │                                                    │ │
│  │  type file struct {                                │ │
│  │      name string                                   │ │
│  │  }                                                 │ │
│  │                                                    │ │
│  │  var f file  // f é REAL                           │ │
│  │  ┌─────────┐                                       │ │
│  │  │ name: ""│                                       │ │
│  │  └─────────┘                                       │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │              TIPOS DE INTERFACE                    │ │
│  │                                                    │ │
│  │  type reader interface {                           │ │
│  │      read(b []byte) (int, error)                   │ │
│  │  }                                                 │ │
│  │                                                    │ │
│  │  var r reader  // r é SEM VALOR                    │ │
│  │  ┌─────────┐                                       │ │
│  │  │   ?     │  ← Sem dados concretos ainda          │ │
│  │  └─────────┘                                       │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Polimorfismo Através do Comportamento

### O Poder de Interfaces Sem Valor

```go
func retrieve(r reader) error {
    data := make([]byte, 100)
    
    len, err := r.read(data)
    if err != nil {
        return err
    }
    
    fmt.Println(string(data[:len]))
    return nil
}
```

**Definição de polimorfismo de Tom Kurtz**:
> "Polimorfismo significa que escreve um certo programa e ele comporta-se de forma diferente dependendo dos dados em que opera."

**Ideia principal**: A declaração da função NÃO está a dizer:
- "Passe-me um valor de tipo reader" (impossível - valores reader não existem)

**Está SIM a dizer**:
- "Passe-me qualquer dado concreto (valor ou ponteiro) que implemente o contrato reader"
- "Passe-me dados concretos que exibam o comportamento de leitura"

### Figura 2: Comportamento de Função Polimórfica

```
┌──────────────────────────────────────────────────────────┐
│              FUNÇÃO POLIMÓRFICA                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  func retrieve(r reader) error {                         │
│      // r é sem valor - aceita QUALQUER dado concreto    │ 
│      // que implemente a interface reader                │ 
│  }                                                       │
│                                                          │
│  ┌─────────────────┐    ┌─────────────────┐              │
│  │   VALOR file    │    │   VALOR pipe    │              │
│  │                 │    │                 │              │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │              │
│  │ │ name: "data"│ │    │ │ name: "cfg" │ │              │
│  │ └─────────────┘ │    │ └─────────────┘ │              │
│  └─────────────────┘    └─────────────────┘              │
│           │                       │                      │
│           │ retrieve(f)           │ retrieve(p)          │
│           ▼                       ▼                      │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              FUNÇÃO retrieve()                      │ │
│  │                                                     │ │
│  │  r.read() comporta-se de forma diferente baseado em:│ │
│  │  • file.read()  → retorna dados RSS                 │ │
│  │  • pipe.read()  → retorna dados JSON                │ │
│  │                                                     │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## Dar Comportamento aos Dados

### Métodos Permitem Comportamento

```go
type reader interface {
    read(b []byte) (int, error)
}

type file struct {
    name string
}

func (file) read(b []byte) (int, error) {
    s := "<rss><channel><title>Going Go</title></channel></rss>"
    copy(b, s)
    return len(s), nil
}

type pipe struct {
    name string
}

func (pipe) read(b []byte) (int, error) {
    s := `{"name": "bill", "title": "developer"}`
    copy(b, s)
    return len(s), nil
}
```

**Pontos chave**:
- Métodos dão comportamento aos dados concretos
- Receivers de valor permitem que valores e ponteiros sejam passados
- Cada tipo agora implementa a interface `reader`
- Comportamento está ligado ao tipo de dado concreto

### Exemplo Polimórfico Completo

```go
func main() {
    f := file{"data.json"}
    p := pipe{"cfg_service"}
    
    retrieve(f)  // Chama file.read() - dados RSS
    retrieve(p)  // Chama pipe.read() - dados JSON
}
```

## Atribuições de Valores de Interface

### Atribuições Interface-a-Interface

```go
type Reader interface {
    Read()
}

type Writer interface {
    Write()
}

type ReadWriter interface {
    Reader
    Writer
}

type system struct {
    Host string
}

func (*system) Read()  { /* ... */ }
func (*system) Write() { /* ... */ }
```

### Figura 3: Atribuições de Valores de Interface

```
┌─────────────────────────────────────────────────────────┐
│           ATRIBUIÇÕES DE VALORES DE INTERFACE           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  var rw ReadWriter = &system{"127.0.0.1"}               │
│  var r Reader = rw                                      │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │    rw (ReadWriter)      │    r (Reader)            │ │
│  │                         │                          │ │
│  │ ┌──────────────────┐    │ ┌─────────────────┐      │ │
│  │ │ Valor de         │    │ │ Valor de        │      │ │
│  │ │ Interface        │    │ │ Interface       │      │ │
│  │ │                  │    │ │                 │      │ │
│  │ │ ┌─────────────┐  │    │ │ ┌─────────────┐ │      │ │
│  │ │ │Tipo:        │  │    │ │ │Tipo:        │ │      │ │
│  │ │ │ReadWriter   │  │    │ │ │Reader       │ │      │ │
│  │ │ └─────────────┘  │    │ │ └─────────────┘ │      │ │
│  │ │ ┌─────────────┐  │    │ │ ┌─────────────┐ │      │ │
│  │ │ │Dados:       │  │    │ │ │Dados:       │ │      │ │
│  │ │ │&system{...} │◄─┼────┼─┤ │&system{...} │ │      │ │
│  │ │ └─────────────┘  │    │ │ └─────────────┘ │      │ │
│  │ └──────────────────┘    │ └─────────────────┘      │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  A atribuição copia os DADOS CONCRETOS, não os valores  │
│  de interface. Tanto rw como r armazenam os mesmos      │
│  dados concretos.                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Ideia principal**: Atribuições de interface copiam os dados concretos, não os valores de interface. Tanto `rw` como `r` armazenam os mesmos dados concretos (`&system{...}`).

```go
func main() {
    var rw ReadWriter = &system{"127.0.0.1"}
    var r Reader = rw
    fmt.Println(rw, r)  // &{127.0.0.1} &{127.0.0.1}
}
```

## Pontos Chave

1. **Valores de interface são sem valor** - não têm significado sem dados concretos
2. **Polimorfismo é orientado por dados** - dados concretos mudam o comportamento do código
3. **Interfaces definem contratos de comportamento** - não implementações concretas
4. **Métodos dão comportamento aos dados** - permitindo polimorfismo
5. **Atribuições de interface copiam dados concretos** - não valores de interface
6. **Foque no comportamento, não na implementação** - para melhor design
7. **Design orientado a dados** - compreenda os seus dados para compreender o seu problema

## Princípios de Design

### Quando Usar Interfaces

- **Desacoplamento**: Quando precisa de desacoplar de implementações concretas
- **Testes**: Quando precisa de simular comportamento
- **Extensibilidade**: Quando múltiplos tipos precisam exibir o mesmo comportamento
- **Design de API**: Quando quer definir contratos, não implementações

### Diretrizes de Design de Interfaces

1. **Foque no comportamento**: O que os dados deveriam poder fazer?
2. **Mantenha interfaces pequenas**: Princípio da responsabilidade única
3. **Desenhe em torno de dados concretos**: Comece com os dados, depois defina comportamento
4. **Aceite interfaces, retorne structs**: Para máxima flexibilidade

### Armadilhas Comuns a Evitar

1. **Sobre-abstração**: Não crie interfaces prematuramente
2. **Poluição de interfaces**: Não crie interfaces para cada tipo concreto
3. **Ignorar semântica**: Seja consistente com semântica de valor vs ponteiro
4. **Foco na implementação**: Não desenhe em torno de detalhes de implementação

## Em Resumo

- **Valores de interface são sem valor** contentores para dados concretos
- **Polimorfismo emerge** de dados concretos exibindo comportamento
- **Design orientado a dados** foca em compreender dados concretos primeiro
- **Contratos de comportamento** permitem desacoplamento preciso sem generalização
- **Atribuições de interface** copiam dados concretos, não valores de interface
- **Métodos fornecem** o mecanismo para dados exibirem comportamento
- **Foco de design** deve ser em relações comportamentais, não detalhes de implementação

---

*Baseado em ["Interface Values Are Valueless"](https://www.ardanlabs.com/blog/2018/03/interface-values-are-valueless.html) de William Kennedy da Ardan Labs.*
