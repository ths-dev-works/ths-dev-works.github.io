---
title: "Go: Tipos de Erro Personalizados - Parte II"
date: 2026-02-02
tags: ["Go", "Tratamento de Erros", "Tipos Personalizados", "Boas Práticas"]
---

Em Go, tipos de erro personalizados fornecem contexto adicional quando mensagens simples não são suficientes. Resumo conciso de tipos de erro personalizados do artigo de William Kennedy da Ardan Labs.

## Quando Usar Tipos de Erro Personalizados

### Use Erros Simples Quando
- Mensagem de erro fornece contexto suficiente
- Não há estado adicional necessário
- Chamador não precisa tomar decisões baseadas no tipo de erro

### Use Tipos de Erro Personalizados Quando
- Chamador precisa de contexto extra para decisões informadas
- Erro requer estado associado
- Você precisa envolver erros originais com contexto

## Exemplo do Pacote net: OpError

### Declaração de Tipo de Erro Personalizado
```go
type OpError struct {
    Op   string // Operação que causou o erro ("read", "write")
    Net  string // Tipo de rede ("tcp", "udp6")
    Addr Addr   // Endereço de rede onde ocorreu o erro
    Err  error  // Erro real que ocorreu
}
```

### Implementação da Interface Error
```go
func (e *OpError) Error() string {
    if e == nil {
        return "<nil>"
    }
    s := e.Op
    if e.Net != "" {
        s += " " + e.Net
    }
    if e.Addr != nil {
        s += " " + e.Addr.String()
    }
    s += ": " + e.Err.Error()
    return s
}
```

### Padrão de Uso
```go
func Listen(net, laddr string) (Listener, error) {
    la, err := resolveAddr("listen", net, laddr, noDeadline)
    if err != nil {
        return nil, &OpError{
            Op:   "listen", 
            Net:  net, 
            Addr: nil, 
            Err:  err
        }
    }
    // ... resto da implementação
}
```

## Exemplo do Pacote json: Múltiplos Tipos de Erro

### UnmarshalTypeError
```go
type UnmarshalTypeError struct {
    Value string       // Descrição do valor JSON
    Type  reflect.Type // Tipo Go que não pôde ser atribuído
}

func (e *UnmarshalTypeError) Error() string {
    return "json: cannot unmarshal " + e.Value + 
           " into Go value of type " + e.Type.String()
}
```

### InvalidUnmarshalError
```go
type InvalidUnmarshalError struct {
    Type reflect.Type
}

func (e *InvalidUnmarshalError) Error() string {
    if e.Type == nil {
        return "json: Unmarshal(nil)"
    }
    if e.Type.Kind() != reflect.Ptr {
        return "json: Unmarshal(non-pointer " + e.Type.String() + ")"
    }
    return "json: Unmarshal(nil " + e.Type.String() + ")"
}
```

## Identificando Tipos de Erro Concretos

### Padrão de Type Switch
```go
err := json.Unmarshal(data, &v)
if err != nil {
    switch e := err.(type) {
    case *json.UnmarshalTypeError:
        log.Printf("Erro de Tipo: Value[%s] Type[%v]", e.Value, e.Type)
        // Manipula incompatibilidade de tipo
    case *json.InvalidUnmarshalError:
        log.Printf("Unmarshal Inválido: Type[%v]", e.Type)
        // Manipula argumento inválido
    default:
        log.Println(err)
        // Manipula outros erros
    }
    return
}
```

### Benefícios Chave do Type Switch
- **Identificação de Tipo**: Determinar tipo de erro concreto
- **Acesso ao Estado**: Acessar campos específicos do erro
- **Manipulação Direcionada**: Manipular diferentes erros adequadamente

## Padrões de Design

### 1. Padrão Wrapper (pacote net)
- Envolve erro original com contexto
- Adiciona informações de operação, rede, endereço
- Preserva erro original no campo `Err`

### 2. Padrão Contexto (pacote json)
- Nome do tipo fornece contexto do erro
- Inclui estado relevante nos campos do struct
- Gera mensagens de erro descritivas

### 3. Convenção de Nomenclatura
- Sufixe tipos de erro personalizados com "Error"
- Exemplos: `OpError`, `UnmarshalTypeError`, `InvalidUnmarshalError`

## Boas Práticas

### 1. Comece Simples
```go
// Use isto quando possível
return errors.New("mypackage: operação falhou")
```

### 2. Adicione Contexto Quando Necessário
```go
// Use isto quando o contexto importa
return &MyError{
    Operation: "save",
    Context:   userData,
    Err:       originalErr,
}
```

### 3. Implemente Método Error()
```go
func (e *MyError) Error() string {
    return fmt.Sprintf("mypackage: operação %s falhou: %v", 
        e.Operation, e.Err)
}
```

### 4. Exporte Tipos de Erro
- Exporte tipos de erro personalizados como parte da sua API
- Permita que chamadores usem type switches
- Documente quando cada tipo de erro ocorre

## Framework de Decisão

### Perguntas a Fazer
1. **O chamador precisa tomar decisões baseadas no tipo de erro?**
   - Sim → Considere tipo de erro personalizado
   - Não → Use erro simples

2. **Você precisa fornecer estado adicional?**
   - Sim → Tipo de erro personalizado com campos
   - Não → Mensagem de erro simples

3. **Você está envolvendo outro erro com contexto?**
   - Sim → Padrão wrapper como OpError
   - Não → Padrão contexto como erros json

## Pontos Chave

1. **Comece Simples**: Use `errors.New()` e `fmt.Errorf()` primeiro
2. **Adicione Contexto**: Crie tipos personalizados quando chamadores precisam de mais informações
3. **Siga Padrões**: Use padrões wrapper ou contexto da biblioteca padrão
4. **Type Switches**: Permita que chamadores identifiquem e manipulem erros específicos
5. **Convenção de Nomes**: Termine tipos de erro personalizados com "Error"
6. **Exporte Tipos**: Faça tipos de erro parte da sua API pública
7. **Preserve Original**: Envolva erros originais, não os substitua

---

*Baseado no artigo ["Error Handling In Go, Part II"](https://www.ardanlabs.com/blog/2014/11/error-handling-in-go-part-ii.html) de William Kennedy.*
