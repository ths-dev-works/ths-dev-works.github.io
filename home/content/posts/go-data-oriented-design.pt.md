---
title: "Go Mechanics: Design Orientado a Dados e Semântica"
date: 2026-02-01
tags: ["Go", "Design", "Semântica", "Arquitetura"]
---

Em Go, entender design orientado a dados e escolher entre semântica de valor e de ponteiro é fundamental para escrever software manutenível e performático. Aqui está um resumo abrangente da filosofia de design de Bill Kennedy sobre dados e semântica da Ardan Labs.

## Filosofias de Design: A Fundação

### Semântica de Valor vs. Semântica de Ponteiro
> "Semântica de valor mantém valores na stack, o que reduz pressão no Garbage Collector (GC). No entanto, semântica de valor requer várias cópias de qualquer valor dado para serem armazenadas, rastreadas e mantidas. Semântica de ponteiro coloca valores no heap, o que pode colocar pressão no GC. No entanto, semântica de ponteiro é eficiente porque apenas um valor precisa ser armazenado, rastreado e mantido." - Bill Kennedy

O uso consistente de semântica de valor/ponteiro para um tipo de dado específico é crítico para manter integridade e legibilidade em todo seu software.

### Modelos Mentais e Escala de Código
> "Vamos imaginar um projeto que vai acabar com um milhão de linhas de código ou mais. A probabilidade desses projetos serem bem-sucedidos nos Estados Unidos hoje em dia é muito baixa - bem abaixo de 50%." - Tom Love (inventor do Objective C)

Considere que uma caixa de papel de cópia comporta ~100k linhas de código. Para que porcentagem desse código você poderia manter um modelo mental? Realisticamente, um único desenvolvedor pode manter cerca de 10k linhas de código. Na escala de um milhão de linhas, você precisa de 100 desenvolvedores coordenados e em comunicação constante.

### Depuração e Modelos Mentais
> "Os bugs mais difíceis são aqueles onde seu modelo mental da situação está simplesmente errado, então você não consegue ver o problema de forma alguma" - Brian Kernighan

Depuradores devem ser usados quando você perdeu seu modelo mental, não como a primeira reação a bugs. Depuração em produção depende de logs, que requerem entender as transformações de dados do código.

### Legibilidade Através da Previsibilidade
> "C é o melhor equilíbrio que já vi entre poder e expressividade. Você pode fazer quase qualquer coisa que queiser programando de forma bastante direta e terá um modelo mental muito bom do que vai acontecer na máquina" - Brian Kernighan

Isso se aplica igualmente a Go. Manter um modelo mental claro impulsiona integridade, legibilidade e simplicidade—os pilares do software manutenível.

## Princípios de Design Orientado a Dados

### Dados é Tudo
> "Se você não entende os dados, você não entende o problema. Isso porque todos os problemas são únicos e específicos para os dados com os quais você está trabalhando." - Bill Kennedy

Todo problema é um problema de transformação de dados. Toda função recebe dados de entrada e produz dados de saída. Seu modelo mental de software é entender essas transformações de dados.

### Menos é Mais
Uma atitude de "menos é mais" é crítica para resolver problemas com:
- Menos camadas
- Menos declarações
- Menos generalização
- Menos complexidade
- Menos esforço

Isso torna tudo mais fácil tanto para sua equipe quanto para o hardware executando as transformações.

### Tipo é Vida
> "Integridade significa que cada alocação, cada leitura de memória e cada escrita de memória é precisa, consistente e eficiente. O sistema de tipos é crítico para garantir que tenhamos esse nível micro de integridade." - William Kennedy

Se dados impulsionam tudo, então os tipos que representam esses dados são críticos. Tipos fornecem ao compilador a capacidade de garantir integridade de dados e ditam regras semânticas.

### Dados com Capacidade
> "Métodos são válidos quando é prático ou razoável para um pedaço de dado ter uma capacidade." - William Kennedy

Métodos dão capacidade aos dados. O foco deve estar nos dados porque eles impulsionam:
- Os algoritmos que você escreve
- As encapsulações que você coloca em prática
- A performance que você pode alcançar

### Polimorfismo Através de Dados
> "Polimorfismo significa que você escreve um certo programa e ele se comporta de forma diferente dependendo dos dados nos quais opera." - Tom Kurtz (inventor do BASIC)

Uma função pode se comportar de forma diferente baseada nos dados nos quais opera. Isso desacopla funções dos tipos de dados concretos e é chave para arquitetar sistemas adaptáveis.

### Abordagem Protótipo Primeiro
> "A menos que o desenvolvedor tenha uma ideia realmente boa do que o software vai ser usado, há uma probabilidade muito alta de que o software vai dar errado." - Brian Kernighan

Foque primeiro em entender dados concretos e algoritmos. Escreva implementações concretas que poderiam ser implantadas em produção. Uma vez funcionando, refatore para desacoplar implementação dos dados concretos dando capacidade aos dados.

## Diretrizes Semânticas

### Regras Principais
Você deve decidir qual semântica (valor ou ponteiro) usar para um tipo de dado específico no momento em que o tipo é declarado. APIs devem respeitar a semântica escolhida para o tipo.

**Diretrizes Básicas:**
- No momento da declaração do tipo, decidir a semântica
- Funções e métodos devem respeitar a escolha semântica
- Evitar receptores de método com semânticas diferentes do tipo
- Evitar funções que aceitam/retornam dados com semânticas diferentes
- Evitar mudar semânticas para um tipo dado

**Exceção:** Unmarshaling sempre requer semântica de ponteiro.

### Tipos Embutidos: Semântica de Valor

Os tipos embutidos do Go (numéricos, textuais, booleanos) devem usar semântica de valor. Não usar ponteiros a menos que tenha um motivo muito bom.

```go
// Do pacote strings - todos usam semântica de valor
func Replace(s, old, new string, n int) string
func LastIndex(s, sep string) int
func ContainsRune(s string, r rune) bool
```

### Tipos de Referência: Semântica de Valor

Tipos de referência (slices, maps, interfaces, funções, channels) devem usar semântica de valor porque são projetados para ficar na stack e minimizar pressão de heap.

```go
// Do pacote net
type IP []byte
type IPMask []byte

// Design de API com semântica de valor
func (ip IP) Mask(mask IPMask) IP {
    // Cria e retorna um novo valor IP
    n := len(ip)
    if n != len(mask) {
        return nil
    }
    out := make(IP, n)
    for i := 0; i < n; i++ {
        out[i] = ip[i] & mask[i]
    }
    return out
}

// append usa semântica de valor
var data []string
data = append(data, "string")  // Retorna novo valor de slice
```

### Tipos Definidos pelo Usuário: O Ponto de Decisão

É aqui que você deve tomar a maioria das decisões. A função factory para um tipo revela qual semântica foi escolhida.

#### Exemplo: Tipo Time (Semântica de Valor)

```go
type Time struct {
    sec int64
    nsec int32
    loc *Location
}

// Função factory mostra semântica de valor
func Now() Time {
    sec, nsec := now()
    return Time{sec + unixToInternal, nsec, Local}
}

// Método respeita semântica de valor
func (t Time) Add(d Duration) Time {
    t.sec += int64(d / 1e9)
    nsec := t.nsec + int32(d%1e9)
    if nsec >= 1e9 {
        t.sec++
        nsec -= 1e9
    } else if nsec < 0 {
        t.sec--
        nsec += 1e9
    }
    t.nsec = nsec
    return t
}

// Função aceita semântica de valor
func div(t Time, d Duration) (qmod2 int, r Duration) {
    // Implementação...
}
```

Apenas funções relacionadas a unmarshal usam semântica de ponteiro:

```go
func (t *Time) UnmarshalBinary(data []byte) error
func (t *Time) GobDecode(data []byte) error
func (t *Time) UnmarshalJSON(data []byte) error
func (t *Time) UnmarshalText(data []byte) error
```

#### Exemplo: Tipo File (Semântica de Ponteiro)

```go
// Função factory mostra semântica de ponteiro
func Open(name string) (file *File, err error) {
    return OpenFile(name, O_RDONLY, 0)
}

// Métodos respeitam semântica de ponteiro
func (f *File) Chdir() error {
    if f == nil {
        return ErrInvalid
    }
    if e := syscall.Fchdir(f.fd); e != nil {
        return &PathError{"chdir", f.name, e}
    }
    return nil
}

// Funções aceitam semântica de ponteiro
func epipecheck(file *File, e error) {
    if e == syscall.EPIPE {
        if atomic.AddInt32(&file.nepipe, 1) >= 10 {
            sigpipe()
        }
    } else {
        atomic.StoreInt32(&file.nepipe, 0)
    }
}
```

## Framework de Decisão

### Quando Usar Semântica de Valor
- Tipos embutidos (int, string, bool, etc.)
- Tipos de referência (slice, map, channel, interface, function)
- Structs pequenas que podem ser razoavelmente copiadas
- Quando copiar é correto e razoável
- Quando você quer reduzir pressão do GC
- Quando isolamento é benéfico

### Quando Usar Semântica de Ponteiro
- Structs grandes onde copiar é caro
- Quando mudanças precisam ser compartilhadas entre funções
- Quando você não tem 100% de certeza que copiar é razoável
- Handles de recursos (arquivos, conexões de rede)
- Quando fazendo unmarshaling de dados
- Quando estado deve ser compartilhado

### A Regra de Ouro
> Se você não tem 100% de certeza que é correto e razoável fazer cópias, então use semântica de ponteiro.

## Consistência e Manutenibilidade

### Foco em Code Review
O uso consistente de semântica de valor/ponteiro deve ser um foco em code reviews porque:
- Mantém o código consistente e previsível
- Permite a todos manter um modelo mental claro
- Torna-se mais importante conforme a base de código e equipe crescem

### Impacto em Toda a Linguagem
A escolha entre semântica de ponteiro e valor estende-se além de receptores e parâmetros de função. Aparece por toda a linguagem:
- Como `for range` funciona
- Mecânicas de interfaces
- Valores de função
- Operações de slice

## Pontos-Chave

1. **Decidir na Declaração**: Escolher semântica quando declara o tipo
2. **Ser Consistente**: APIs devem respeitar a semântica escolhida
3. **Padrão para Valor**: Usar semântica de valor a menos que tenha um bom motivo
4. **Dados Impulsionam Design**: Focar nos dados e suas transformações
5. **Protótipo Primeiro**: Começar com implementações concretas
6. **Menos é Mais**: Simplicidade auxilia manutenibilidade
7. **Modelos Mentais Importam**: Consistência ajuda a manter modelos mentais

## Em Resumo

- **Design orientado a dados** começa com entender os dados e suas transformações
- **Semântica de valor** reduz pressão do GC mas requer cópia
- **Semântica de ponteiro** é eficiente mas pode aumentar pressão do GC
- **Consistência** é crítica para manutenibilidade e modelos mentais
- **Tipos embutidos e de referência** devem usar semântica de valor
- **Tipos definidos pelo usuário** requerem escolhas semânticas deliberadas
- **Funções factory** revelam a semântica pretendida
- **Unmarshaling** é a principal exceção às regras semânticas

---

*Este resumo é baseado em ["Design Philosophy On Data And Semantics"](https://www.ardanlabs.com/blog/2017/06/design-philosophy-on-data-and-semantics.html) de Bill Kennedy da Ardan Labs, o post final de uma série de quatro partes cobrindo a mecânica e filosofia de design do Go.*
