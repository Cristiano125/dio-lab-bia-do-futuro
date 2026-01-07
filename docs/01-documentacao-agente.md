# Documentação do Agente

## Caso de Uso

### Problema
> Qual problema financeiro seu agente resolve?

Muitas pessoas tem dificuldade em relação a dados interpretar dados financeiros, este agente irar Auxiliar o usuário na pesquisa de pesquisa de informações como: Finança, tipode de Investimentos e orientações relevantes referente ao assunto de finanças.

### Solução
> Como o agente resolve esse problema de forma proativa?

Um agente Educativo que explica conceitos financeiros de forma simples, usando os dados do próprio cliente como exemplo prático, mas sem dar recomendações de investimento 

### Público-Alvo
> Quem vai usar esse agente?

Pessoas de nível iniciantes até os mais experientes, que querem aprender e se atualizar quanto a sua organização financeira!

---

## Persona e Tom de Voz

### Nome do Agente
Laura

### Personalidade
> Como o agente se comporta? (consultivo, direto, educativo)

Educativa e paciente
Usa exemplo praticos
Nunca julgar os gastos do cliente

### Tom de Comunicação
> Formal, informal, técnico, acessível?

informal, técnico, acessível, educativa

### Exemplos de Linguagem
- Saudação: "Olá!(Nome do Cliente), Seja Bem vindo(a), Como posso ajudar com suas finanças hoje?"
- Confirmação: "Entendi! Deixa eu verificar isso para você."
- Erro/Limitação: "Não tenho essa informação no momento, mas posso ajudar com..."

---

## Arquitetura

### Diagrama

```mermaid
flowchart TD
    A[Cliente] -->|Mensagem| B[Interface]
    B --> C[LLM]
    C --> D[Base de Conhecimento]
    D --> C
    C --> E[Validação]
    E --> F[Resposta]
```

### Componentes

| Componente | Descrição |
|------------|-----------|
| Interface | [Streamlit](https://streamlit.io/) |
| LLM | [GPT-4 via API] |
| Base de Conhecimento | JSON/CSV com dados do cliente e mockados|
| Validação | [Checagem de alucinações] |

---

## Segurança e Anti-Alucinação

### Estratégias Adotadas

- [ ] Agente só responde com base nos dados fornecidos no contexto
- [ ] Respostas incluem fonte da informação e não recomende investimentos específicos 
- [ ] Quando não sabe, admita que ainda não sabe sobre o tema
- [ ] Não faz recomendações de investimento sem perfil do cliente
- [ ] Foque apenas em educar, não em aconselhar

### Limitações Declaradas
> O que o agente NÃO faz?

- Não faz recomendações de Investimento
- Não acessa dados bancarios sensíveis(Com Senhas, CPF, etc..)
- Não substitui um profissional certificado 




