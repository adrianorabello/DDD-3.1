# 📚 Trabalho — Design Tático no DDD (Template para qualquer domínio)

> **Como usar:** copie este arquivo e substitua os **[colchetes]** com informações do **seu domínio** (e-commerce, marketplace, logística, educação, fintech, games, etc.).
> O objetivo é praticar Entidades, Value Objects, Agregados/AR, Repositórios e Eventos de Domínio — com foco em **invariantes** e **domínio rico**.

---

## 🚀 Quick start (5 passos)
1. Escolha um **domínio** que você conheça (ex.: **[Seu Domínio]**).
2. Liste 3–7 **invariantes** que devem estar corretas no **commit**.
3. Escolha 1–2 **Agregados principais** (comece por **[Agregado Principal]**).
4. Desenhe a **máquina de estados** e os **eventos** que surgem das transições.
5. Defina o **Repositório** da AR e como lidará com **consistência** entre agregados.

---

## 🩺 1) Sobre o Domínio Escolhido
**Nome do domínio:** **[Seu Domínio]**  
**Objetivo do sistema:** **[Frase curta que explica a proposta de valor]**  
**Principais atores:** **[Lista: Cliente, Vendedor, Motorista, Professor, etc.]**  
**Contextos (opcional):** **[Contextos/Bounded Contexts propostos]**

---

## 🧩 2) Entidades vs Value Objects
Preencha a tabela justificando cada tipo (identidade vs. imutabilidade).

| Elemento | Tipo (Entidade/VO) | Por quê? (identidade/imutável) |
|---|---|---|
| **[Elemento A]** | [Entidade/VO] | [Justificativa] |
| **[Elemento B]** | [Entidade/VO] | [Justificativa] |
| **[Elemento C]** | [Entidade/VO] | [Justificativa] |
| **[Elemento D]** | [Entidade/VO] | [Justificativa] |

> Dica: Promova tipos semânticos: `Email`, `CPF/CNPJ`, `Money`, `IntervaloDeTempo`, `Endereco`, `Percentual`, `Quantidade`, etc. **VOs devem ser imutáveis** e com **igualdade por valor**.

---

## 🏗️ 3) Agregados e Aggregate Root (AR)
**Agregado Principal:** **[Agregado Principal]**  
**AR:** **[Nome da AR]**  
**Conteúdo interno do agregado (apenas o necessário para consistência local):**  
- **[Entidade interna/VO]**
- **[Entidade interna/VO]**

**Referências a outros agregados (por ID):**  
- **[OutroAgregadoId]** (não conter dentro do agregado)
- **[OutroAgregadoId]**

**Boundary — Por que cada item está dentro/fora?**  
- **Dentro porque [precisa de consistência transacional por causa da invariante X]**  
- **Fora porque [pode esperar/eventual; pertence a outro BC; só precisa de referência por ID]**

---

## 🧭 4) Invariantes e Máquina de Estados
Liste invariantes (devem ser verdadeiras ao final de cada transação).

**Invariantes (exemplos):**
- **[Não aceitar pagamento acima do limite de crédito]**
- **[Não permitir slot de horário sobreposto para o mesmo recurso]**
- **[Não permitir alteração após estado X]**
- **[Preço Total = soma dos itens] (se aplicável)**

**Estados e transições da AR [Nome da AR]:**
```
[EstadoInicial] -> [Estado1] -> [Estado2] -> [EstadoFinal]
Regras:
- [Transição A] permitida se [condições/invariantes]
- [Transição B] bloqueada se [condições]
- [Transição C] exige [política/serviço]
```

---

## 🗃️ 5) Repositório do Agregado (interface)
> Repositório trabalha **apenas com a AR**, sem expor entidades internas do agregado. Consultas analíticas ficam fora (read models).

**Linguagem livre** (ex.: C#, Java, Kotlin, TS). Exemplo (C# assíncrono, adapte nomes):
```csharp
public interface I[Agregado]Repository
{
    Task<[Agregado]?> ObterPorIdAsync(Guid id, CancellationToken ct = default);
    Task AdicionarAsync([Agregado] entidade, CancellationToken ct = default);
    Task SalvarAsync([Agregado] entidade, CancellationToken ct = default);
}
```

---

## 🔄 6) Estratégia para Consistência entre Agregados
Escolha **uma** (ou combine) e **justifique trade-offs**:

- **A. Índice Único** no armazenamento (ex.: `(RecursoId, Inicio, Fim)`): simples, performático; requer _retry_ com **concorrência otimista**.
- **B. Agregado Guardião** (ex.: `AgendaDoRecurso`): concentra regras de overlap; invariantes fortes; risco de contenção; pode exigir particionamento por período.
- **C. Serviço de Domínio + Outbox/Projeções**: validação orquestrada, consistência eventual; complexidade operacional (idempotência, retries).

**Decisão escolhida:** **[A/B/C]**  
**Justificativa curta:** **[Por quê essa? Quando pode falhar? Plano de mitigação?]**

---

## 📣 7) Eventos de Domínio
Defina **2–4 eventos** com **payload mínimo** e **momento de publicação** (preferir **pós-commit**). Diferencie **evento interno** vs **evento de integração**.

| Evento | Quando ocorre | Payload mínimo | Interno/Integração | Observações |
|---|---|---|---|---|
| **[EventoXOcorrido]** | [ao confirmar/remarcar/etc.] | [ids, valores necessários] | [Interno/Integração] | [idempotência, consumidor] |
| **[EventoYOcorrida]** | [...] | [...] | [...] | [...] |
| **[EventoZOcorrida]** | [...] | [...] | [...] | [...] |

---

## 🗺️ 8) Diagrama (Mermaid ou ferramenta à sua escolha)
> Mostre **Agregados/AR**, **VOs** e **relacionamentos por ID** entre agregados (não “contenha” outros agregados).

**Exemplo de esqueleto Mermaid:**
```mermaid
classDiagram
  class AgregadoPrincipal {
    +Guid Id
    +Guid OutroAgregadoId
    +VOImportante Valor
    +Status Estado
    +Operacao1(args)
    +Operacao2(args)
  }

  class VOImportante {
    +Atributo1
    +Atributo2
    +OperacaoVO()
  }

  class OutroAgregado {
    +Guid Id
  }

  AgregadoPrincipal --> OutroAgregado : por Id
  AgregadoPrincipal --> VOImportante
```

---

## ✅ Checklist de Aceitação
- [ ] **VOs imutáveis** e com **igualdade por valor** (nada de “string de CPF/Email”).
- [ ] **Boundary do agregado** pequeno e com **invariantes claras**.
- [ ] **Domínio rico**: operações do negócio como métodos (evitar `set` aberto).
- [ ] **Repositório** focado na **AR** (sem `IQueryable`/detalhes de ORM no domínio).
- [ ] **Eventos** definidos, publicados **pós-commit**, handlers idempotentes.
- [ ] Estratégia de **consistência entre agregados** escolhida e **defendida**.

---

## 📝 Rubrica de Avaliação (sugestão)
| Critério | Peso |
|---|---|
| Entidades vs VOs (imutabilidade, igualdade por valor, justificativas) | **2.0** |
| Boundary do agregado principal (coerência, tamanho, IDs externos) | **2.0** |
| Invariantes e máquina de estados | **2.0** |
| Repositório focado na AR (sem vazamentos de infra) | **1.5** |
| Eventos de Domínio (nomes, payload mínimo, pós-commit, idempotência) | **1.5** |
| Estratégia de consistência (trade-offs, riscos e mitigação) | **1.0** |

**Nota máxima: 10,0**

---

## 📤 Entrega
- **Arquivo**: `trabalho-ddd-design-tatico-{seu-nome}.md`
- **Inclua**: link/imagem do **diagrama** + todas as seções acima preenchidas.
- **Prazo**: **[definido pelo professor]**

---

## 💡 Dicas Rápidas
- “**Igual** não é **o mesmo**” → VO ≠ Entidade.  
- “**Uma transação, um agregado**” → consistência local.  
- “**Regra perto dos dados**” → domínio rico.  
- “**O que pode esperar**?” → evento + consistência eventual.



> **Dicas para Mermaid no GitHub**
> - Não use **colchetes `[]`** nos nomes das classes/tipos.
> - Evite espaços: use **CamelCase** ou **snake_case**.
> - Mantenha o bloco cercado por crases com a linguagem **mermaid**.
> - Alternativa se preferir um diagrama mais simples:
>
> ```mermaid
> flowchart TB
>   AR[AgregadoPrincipal (AR)]
>   OA[OutroAgregado]
>   VO[VOImportante]
>   AR -->|ref por Id| OA
>   AR --> VO
> ```
>
> Ou modelo ER:
>
> ```mermaid
> erDiagram
>   AGREGADOPRINCIPAL ||--o{ VOIMPORTANTE : "contém"
>   AGREGADOPRINCIPAL }o--|| OUTROAGREGADO : "ref por Id"
>   AGREGADOPRINCIPAL {
>     GUID Id
>     GUID OutroAgregadoId
>     STRING Estado
>   }
>   VOIMPORTANTE {
>     STRING Atributo1
>     STRING Atributo2
>   }
>   OUTROAGREGADO {
>     GUID Id
>   }
> ```
