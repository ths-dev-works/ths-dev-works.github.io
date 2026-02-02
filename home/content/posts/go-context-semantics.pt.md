---
title: "Go Mechanics: Semântica do Pacote Context"
date: 2026-02-02
tags: ["Go", "Context", "Concorrência", "Timeouts", "Cancelamento"]
description: "Compreender as semânticas essenciais do pacote context de Go para cancelamento adequado, timeouts e gestão de dados com escopo de requisição."
---

Em Go, o pacote context é essencial para gerir dados com escopo de requisição, sinais de cancelamento e prazos através de fronteiras de API e entre goroutines. Como Go não tem palavras-chave incorporadas para terminar goroutines, o pacote context fornece o mecanismo crítico para gerir a saúde e operação do serviço.

Este post resume as principais ideias de William Kennedy sobre semânticas do pacote context da Ardan Labs, focando nas regras e padrões estabelecidos para uso adequado.

## Introdução: O Problema do Context

Go fornece a palavra-chave `go` para criar goroutines mas não oferece suporte direto para terminá-las. Em serviços do mundo real, a capacidade de fazer timeout e terminar goroutines é crítica para manter a saúde do serviço. Nenhuma requisição deve correr para sempre - gerir latência é responsabilidade de cada programador.

O pacote context, introduzido por Sameer Ajmani em 2014, resolve este problema fundamental fornecendo:
- **Suporte de prazos**: Cancelamento automático em tempos específicos
- **Sinais de cancelamento**: Propagação de cancelamento manual
- **Valores com escopo de requisição**: Dados que viajam através de fronteiras de API

## A Interface Context

O tipo Context é uma interface com quatro métodos chave:

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

**Convenções chave**:
- Use o nome de variável `ctx` para todos os valores Context
- Como Context é uma interface, use semântica de valor (sem ponteiros)
- Cada função que aceita um Context obtém a sua própria cópia

## Regras Semânticas para Uso do Context

### Regra 1: Requisições recebidas devem criar um Context

Crie Context o mais cedo possível no processamento da requisição. Isto força o design da API a aceitar Context como primeiro parâmetro.

#### Criação de Context de Requisição HTTP
```go
// Go 1.7+ - Context já está na requisição
h := func(w http.ResponseWriter, r *http.Request, params map[string]string) {
    ctx, span := trace.StartSpan(r.Context(), "internal.platform.web")
    defer span.End()
    // ... lógica do handler
}

// Pré-Go 1.7 - Criar Context vazio manualmente
ctx := context.Background()
ctx, span := trace.StartSpan(ctx, "internal.platform.web")
defer span.End()
```

**Context de Background**:
- `context.Background()` retorna um Context não-nulo e vazio
- Nunca é cancelado, não tem valores, não tem prazo
- Usado pela função main, inicialização, testes e como Context de nível superior para requisições recebidas

### Regra 2: Chamadas de saída devem aceitar um Context

Chamadas de nível superior devem dizer a chamadas de nível inferior quanto tempo estão dispostas a esperar. Isto é essencial para propagação adequada de timeout.

#### Cliente HTTP com Context
```go
func main() {
    // Criar requisição
    req, err := http.NewRequest("GET", "https://api.example.com", nil)
    if err != nil {
        log.Println("ERRO:", err)
        return
    }

    // Criar context com timeout
    ctx, cancel := context.WithTimeout(req.Context(), 50*time.Millisecond)
    defer cancel()

    // Vincular context à requisição
    req = req.WithContext(ctx)

    // Fazer chamada - método Do respeita o timeout
    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        log.Println("ERRO:", err)
        return
    }
    defer resp.Body.Close()

    // Processar resposta
    io.Copy(os.Stdout, resp.Body)
}
```

**Pontos Chave**:
- Função de nível superior diz à função de nível inferior quanto tempo esperar
- `http.Do()` respeita o timeout no Context
- Context permite propagação adequada de timeout através de fronteiras de API

### Regra 3: Não armazene Contexts em structs; passe explicitamente

Funções que realizam I/O devem aceitar Context como primeiro parâmetro e respeitar configurações de timeout/prazo.

#### Padrão Correto
```go
// Função de base de dados aceitando Context
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    ctx, span := trace.StartSpan(ctx, "internal.user.List")
    defer span.End()

    users := []User{}
    const q = `SELECT * FROM users`

    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, errors.Wrap(err, "selecionando usuários")
    }

    return users, nil
}

// Handler chamando função de base de dados
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    ctx, span := trace.StartSpan(ctx, "handlers.User.List")
    defer span.End()

    users, err := user.List(ctx, u.db)  // Context propagado
    if err != nil {
        return err
    }

    return web.Respond(ctx, w, users, http.StatusOK)
}
```

**Por que não armazenar em structs?**
- Context tem escopo de requisição, não de struct
- Passagem explícita torna dependências claras
- Evita dependências ocultas e problemas de tempo de vida

### Regra 4: Propague Context através de cadeias de chamada

Context e quaisquer alterações feitas durante o processamento da requisição devem ser propagadas e respeitadas por toda a cadeia de chamada.

#### Padrão de Propagação
```go
// Nível do handler
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    ctx, span := trace.StartSpan(ctx, "handlers.User.List")
    defer span.End()

    users, err := user.List(ctx, u.db)  // Propagar Context
    if err != nil {
        return err
    }

    return web.Respond(ctx, w, users, http.StatusOK)  // Propagar Context
}

// Nível da lógica de negócio
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    ctx, span := trace.StartSpan(ctx, "internal.user.List")
    defer span.End()

    users := []User{}
    const q = `SELECT * FROM users`

    if err := db.SelectContext(ctx, &users, q); err != nil {  // Propagar Context
        return nil, errors.Wrap(err, "selecionando usuários")
    }

    return users, nil
}
```

**Importante**: Não crie novos valores Context de nível superior em funções do meio - isto perde informações Context existentes de chamadas de nível superior.

### Regra 5: Substitua Context usando funções With

Context usa semântica de valor - qualquer alteração cria um novo valor Context. Use a função With apropriada para modificações.

#### Padrões de Criação de Context
```go
func main() {
    // Criar context com timeout
    duration := 150 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), duration)
    defer cancel()  // CRÍTICO: Sempre chame cancel

    // Criar canal para resultado do trabalho
    ch := make(chan data, 1)

    // Iniciar trabalho da goroutine
    go func() {
        time.Sleep(50 * time.Millisecond)  // Simular trabalho
        ch <- data{"123"}
    }()

    // Esperar pelo trabalho com timeout
    select {
    case d := <-ch:
        fmt.Println("trabalho completo", d)
    case <-ctx.Done():
        fmt.Println("trabalho cancelado")
    }
}
```

**Regras Críticas**:
- Sempre chame a função cancel (use `defer`)
- Use função With apropriada para sua necessidade:
  - `WithCancel`: Cancelamento manual
  - `WithDeadline`: Cancelamento em tempo específico
  - `WithTimeout`: Cancelamento após duração
  - `WithValue`: Armazenar valores com escopo de requisição

### Regra 6: Cancelamento propaga para Contexts derivados

Quando um Context pai é cancelado, todos os Contexts derivados dele também são cancelados. Isto permite limpeza eficiente de árvores de requisição inteiras.

#### Propagação de Cancelamento
```go
func main() {
    // Criar Context cancelável
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    var wg sync.WaitGroup
    wg.Add(10)

    // Criar 10 goroutines com Contexts derivados
    for i := 0; i < 10; i++ {
        go func(id int) {
            defer wg.Done()

            // Derivar Context com valor único
            ctx := context.WithValue(ctx, key, id)

            // Esperar pelo cancelamento
            <-ctx.Done()
            fmt.Println("Cancelado:", id)
        }(i)
    }

    // Cancelar todos os Contexts derivados com uma chamada
    cancel()
    wg.Wait()
}
```

**Benefícios Chave**:
- Uma chamada cancel cancela todos os Contexts derivados
- Context é seguro para uso simultâneo por múltiplas goroutines
- Limpeza eficiente de árvores de requisição inteiras

### Regra 7: Nunca passe nil Context; use TODO quando incerto

Sempre passe um Context válido. Se não tiver certeza de qual Context usar, use `context.TODO()` em vez de `nil`.

#### Uso de TODO
```go
// Quando sabe que precisa de um Context mas não tem certeza de onde vem
func processData(data []byte) error {
    ctx := context.TODO()  // Temporário até descobrir a fonte
    return database.Save(ctx, data)
}

// Mais tarde, quando sabe a fonte do Context
func processData(ctx context.Context, data []byte) error {
    return database.Save(ctx, data)
}
```

**Background vs TODO**:
- `Background`: Para Context de nível superior (main, init, testes)
- `TODO`: Quando precisa de um Context temporariamente durante desenvolvimento

### Regra 8: Use valores Context apenas para dados com escopo de requisição

Esta é a regra semântica mais crítica. Não use Context para passar parâmetros de função necessários.

#### O que NÃO fazer - Dados Necessários no Context
```go
// ANTI-PADRÃO: Conexão de base de dados no Context
func List(ctx context.Context) ([]User, error) {
    db := ctx.Value("database").(*sqlx.DB)  // Dependência oculta
    
    users := []User{}
    const q = `SELECT * FROM users`
    
    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, err
    }
    
    return users, nil
}
```

#### O que FAZER - Dependências Explícitas
```go
// BOM PADRÃO: Conexão de base de dados como parâmetro
func List(ctx context.Context, db *sqlx.DB) ([]User, error) {
    users := []User{}
    const q = `SELECT * FROM users`
    
    if err := db.SelectContext(ctx, &users, q); err != nil {
        return nil, err
    }
    
    return users, nil
}

// Quando a assinatura não pode ser alterada (como handlers HTTP)
func (u *User) List(ctx context.Context, w http.ResponseWriter, r *http.Request, params map[string]string) error {
    users, err := user.List(ctx, u.db)  // db do receiver
    if err != nil {
        return err
    }
    return web.Respond(ctx, w, users, http.StatusOK)
}

type User struct {
    db *sqlx.DB  // Dependência explícita
    authenticator *auth.Authenticator
}
```

#### Prioridade de Passagem de Dados
1. **Parâmetros de função** - Mais claro, sem dependências ocultas
2. **Campos do receiver** - Quando a assinatura não pode ser alterada
3. **Valores Context** - Apenas para dados com escopo de requisição

#### Valores Context Seguros
Dados com escopo de requisição seguros para Context:
- **Trace IDs**: Para rastreamento distribuído
- **Tempo de início da requisição**: Para monitoramento de performance
- **Tokens de autenticação**: Quando transitam fronteiras de API
- **IDs de correlação**: Para rastreamento de requisição

```go
// BOM: Dados de rastreamento com escopo de requisição
type Values struct {
    TraceID    string
    StartTime  time.Time
    StatusCode int
}

// Armazenar no Context para middleware e logging
v := Values{
    TraceID:   span.SpanContext().TraceID.String(),
    StartTime: time.Now(),
}
ctx = context.WithValue(ctx, KeyValues, &v)
```

## Segurança de Valores Context

Ao usar valores Context, verifique sempre a integridade:

```go
// Recuperação segura de valor Context
func logRequest(ctx context.Context, r *http.Request) {
    v, ok := ctx.Value(KeyValues).(*Values)
    if !ok {
        // Problema grave de integridade - desligar serviço
        return web.NewShutdownError("valor web ausente do context")
    }
    
    log.Printf("%s : %s %s -> %s (%s)",
        v.TraceID,
        r.Method, r.URL.Path,
        r.RemoteAddr,
        time.Since(v.StartTime),
    )
}
```

**Consequências do mau uso**:
- Necessidade de verificação de integridade e mecanismos de desligamento
- Testes e depuração tornam-se mais difíceis
- Clareza e legibilidade de código reduzidas

## Pontos Chave

1. **Crie Context cedo** no processamento da requisição
2. **Aceite Context explicitamente** como primeiro parâmetro
3. **Não armazene Context em structs** - passe explicitamente
4. **Propague Context** através de toda a cadeia de chamada
5. **Use funções With** para criar Contexts derivados
6. **Sempre chame funções cancel** com defer
7. **Cancelamento propaga** para todos os Contexts derivados
8. **Nunca passe nil Context** - use TODO quando incerto
9. **Use valores Context apenas para dados com escopo de requisição**, não parâmetros necessários
10. **Priorize dependências explícitas** sobre dependências Context ocultas

## Em Resumo

- **Context gere ciclo de vida da requisição** através de prazos, cancelamento e valores
- **Semântica de valor garante** que alterações não afetam referências Context existentes
- **Padrões de propagação permitem** comportamento adequado de timeout e cancelamento
- **Dependências explícitas** fornecem melhor clareza que dependências Context ocultas
- **Dados com escopo de requisição** como trace IDs são apropriados para valores Context
- **Padrões de uso adequados** são essenciais para serviços Go confiáveis e mantíveis

---

*Baseado em ["Context Package Semantics In Go"](https://www.ardanlabs.com/blog/2019/09/context-package-semantics-in-go.html) de William Kennedy da Ardan Labs.*
