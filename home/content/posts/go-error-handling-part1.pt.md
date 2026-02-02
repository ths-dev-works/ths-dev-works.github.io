---
title: "Go: Fundamentos de Tratamento de Erros - Parte I"
date: 2026-02-02
tags: ["Go", "Tratamento de Erros", "Interface", "Boas Práticas"]
---

Resumo conciso dos fundamentos de tratamento de erros de Go do artigo de William Kennedy da Ardan Labs.

## Interface Error

```go
type error interface {
    Error() string
}
```

- **Método Único**: Apenas `Error()` retornando string
- **Universal**: Usada em toda biblioteca padrão
- **Flexível**: Qualquer tipo implementando `Error()` pode ser erro

## Criando Erros

### `errors.New()` para mensagens simples
```go
var ErrInvalidParam = errors.New("mypackage: parâmetro inválido")
```

### `fmt.Errorf()` para mensagens formatadas
```go
var ErrInvalidParam = fmt.Errorf("parâmetro inválido [%s]", param)
```

## Padrão de Tratamento

```go
resp, err := c.Get(url)
if err != nil {
    log.Println(err)
    return
}
```

### Princípios
- **Verificação Imediata**: Sempre verifique erros após chamadas
- **Retorno Antecipado**: Use retornos antecipados
- **Propagação**: Deixe erros subirem até manipuladores

## Variáveis de Erro da Biblioteca Padrão

```go
// bufio
var (
    ErrInvalidUnreadByte = errors.New("bufio: uso inválido de UnreadByte")
    ErrBufferFull        = errors.New("bufio: buffer cheio")
)

// io
var (
    EOF              = errors.New("EOF")
    ErrUnexpectedEOF = errors.New("EOF inesperado")
)
```

## Comparando Erros

### Use Variáveis de Pacote (Recomendado)
```go
switch err {
case bufio.ErrNegativeCount:
    // Manipula erro específico
}
```

### Evite Comparação de String
```go
// ANTI-PADRÃO: Frágil e lento!
switch err.Error() {
case "bufio: negative count":  // Quebra se mensagem mudar
```

## Filosofia de Design

### Por Que Receivers de Ponteiro?
- **Unicidade**: Cada chamada cria valor único
- **Identidade**: Baseada em ponteiro, não conteúdo
- **Estabilidade**: Previne correspondências acidentais

## Boas Práticas

### 1. Variáveis de Erro do Pacote
```go
var (
    ErrInvalidInput = errors.New("mypackage: entrada inválida")
    ErrTimeout      = errors.New("mypackage: timeout")
)
```

### 2. Convenções
- **Prefixo**: Use nome do pacote como prefixo
- **Prefixo Err**: Comece variáveis com `Err`
- **Exportação**: Exporte variáveis como parte da API

## Pontos Chave

1. **Interface Simples**: Design elegante e flexível
2. **Semântica de Ponteiro**: Garante unicidade
3. **Variáveis de Pacote**: Exporte erros predefinidos
4. **Retornos Antecipados**: Manipule erros imediatamente
5. **Mensagens Descritivas**: Inclua contexto do pacote
6. **Evite String Comparison**: Use variáveis de erro
7. **Performance**: Comparação baseada em ponteiro é rápida

---

*Baseado no artigo ["Error Handling In Go, Part I"](https://www.ardanlabs.com/blog/2014/10/error-handling-in-go-part-i.html) de William Kennedy.*
