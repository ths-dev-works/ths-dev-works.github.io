---
title: "Go: Métodos, Interfaces e Tipos Embutidos"
date: 2026-02-02
tags: ["Go", "Métodos", "Interfaces", "Tipos Embutidos", "Composição"]
---

Em Go, métodos, interfaces e tipos embutidos criam um sistema poderoso para organização de código usando composição em vez de herança. Resumo conciso desses fundamentos do artigo de William Kennedy da Ardan Labs.

## Métodos em Go

### O que é um Método?
Função com **receiver** - valor ou ponteiro de um tipo nomeado. Todos os métodos de um tipo pertencem ao seu **method set**.

```go
type User struct {
    Name  string
    Email string
}

// Receiver de valor
func (u User) Notify() error { ... }

// Receiver de ponteiro  
func (u *User) UpdateEmail(email string) { ... }
```

### Receiver de Valor vs Ponteiro

#### Receiver de Valor
- Opera em **cópia** do receiver
- Não pode modificar valor original
- Pode ser chamado com valores e ponteiros

#### Receiver de Ponteiro
- Opera no valor **original**
- Pode modificar valor original
- Pode ser chamado com valores e ponteiros (Go manipula dereferência)

## Interfaces em Go

### Declaração de Interface
```go
type Notifier interface {
    Notify() error
}
```

### Características
- **Implementação Implícita**: Tipos implementam interfaces automaticamente
- **Interfaces Pequenas**: maioria tem 1-2 métodos
- **Convenção**: Interfaces de método único usam sufixo `-er`

### Regras de Conformidade

#### Regra Crítica
> O method set do tipo T **não** inclui métodos com receiver *T

**Consequências:**
- Se método tem **receiver de ponteiro**, apenas **ponteiros** satisfazem interface
- Se método tem **receiver de valor**, ambos **valores e ponteiros** satisfazem interface

```go
// Receiver de ponteiro - apenas ponteiros funcionam
func (u *User) Notify() error { ... }

user := User{"bill", "bill@email.com"}
SendNotification(user)  // ERRO: User não implementa Notifier

pUser := &User{"jill", "jill@email.com"}  
SendNotification(pUser) // OK: *User implementa Notifier
```

## Tipos Embutidos (Composição)

### O que é Type Embedding?
Permite campos anônimos em structs, criando composição:

```go
type Admin struct {
    User  // Tipo embutido
    Level string
}
```

### Promoção de Métodos
Métodos do tipo embutido são **promovidos** para tipo externo:

```go
admin := &Admin{
    User: User{"john", "john@email.com"},
    Level: "super",
}

// Ambos funcionam - método é promovido
admin.Notify()        // Chama User.Notify()
admin.User.Notify()   // Chamada explícita ao tipo embutido
```

### Regras de Promoção

#### Regra 1: Promoção de Receiver de Valor
Se S contém campo anônimo T, ambos S e *S incluem métodos promovidos com receiver T.

#### Regra 2: Promoção de Receiver de Ponteiro  
Apenas *S inclui métodos promovidos com receiver *T.

#### Regra 3: Embedding de Ponteiro
Se S contém campo anônimo *T, ambos S e *S incluem métodos com receiver T ou *T.

### Regra Derivada
> Se S contém campo anônimo T, o method set de S **não** inclui métodos promovidos com receiver *T.

Consistente com regras de conformidade de interface.

## Implementação com Tipos Embutidos

### Múltiplas Implementações
Quando tipos externo e embutido implementam mesma interface:

```go
// User implementa Notifier
func (u *User) Notify() error { ... }

// Admin também implementa Notifier
func (a *Admin) Notify() error { ... }
```

### Regras de Resolução
1. **Prioridade do Tipo Externo**: Se implementa, é usado
2. **Fallback do Tipo Interno**: Se não implementa, métodos promovidos são usados
3. **Sem Conflitos**: Ambas implementações coexistem unicamente

```go
admin := &Admin{...}
SendNotification(admin)  // Chama Admin.Notify()
admin.Notify()           // Chama Admin.Notify()
admin.User.Notify()      // Chama User.Notify()
```

## Respondendo Perguntas Chave

### Q1: Erro do Compilador?
**Não**. Tipos embutidos usam nomes únicos para implementações internas/externas.

### Q2: Seleção de Implementação?
**Seleção por prioridade**: Implementação do tipo externo tem precedência, tipo interno disponível via acesso explícito.

## Boas Práticas

### 1. Design de Interface
- Mantenha interfaces pequenas e focadas
- Descubra interfaces do uso, não projete antecipadamente

### 2. Seleção de Receiver
- Use receiver de valor para operações imutáveis
- Use receiver de ponteiro para modificações ou structs grandes

### 3. Padrões de Composição
- Use embedding para reuso de código, não hierarquias de herança
- Prefira composição sobre herança

## Pontos Chave

1. **Implementação Implícita**: Tipos implementam interfaces automaticamente
2. **Regras de Method Set**: Receivers de valor funcionam com ambos; receivers de ponteiro apenas com ponteiros
3. **Composição sobre Herança**: Use embedding para reuso de código
4. **Interfaces Pequenas**: Mantenha interfaces focadas e mínimas
5. **Promoção de Métodos**: Governada por regras específicas
6. **Resolução de Prioridade**: Implementações externas têm precedência
7. **Sem Conflitos**: Múltiplas implementações podem coexistir

---

*Baseado no artigo ["Methods, Interfaces and Embedded Types in Go"](https://www.ardanlabs.com/blog/2014/05/methods-interfaces-and-embedded-types.html) de William Kennedy.*
