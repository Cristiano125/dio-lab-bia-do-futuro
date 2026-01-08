# Base de Conhecimento

## Dados Utilizados

| Arquivo | Formato | Utilização da Laura |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores e dar um atendimento mais eficiente|
| `perfil_investidor.json` | JSON | Personalizar as recomendações sobre dúvidas e necessidades de aprendizagem para o cliente |
| `produtos_financeiros.json` | JSON | Sugerir produtos adequados ao perfil, apresentando os produtos disponíveis, afim que o cliente possa aprender mais sobre o assunto.|
| `transacoes.csv` | CSV | Analisar padrões de gastos do cliente e usar essas informações de forma didática |

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

Foi incrementada na base de dados de Produtos Financeiros mais 15 exemplos para que a agente tenha mais opções. 

---

## Estratégia de Integração

### Como os dados são carregados?
> Descreva como seu agente acessa a base de conhecimento.

Existe a possibilidade de carregar diretamente no propent (ctrl + c / ctrl + v ) ou carregamento por código, conforme abaixo.

```Python
import os
import pandas as pd
import json
import xml.etree.ElementTree as ET

# Caminho da pasta
PASTA = "dados/"

def ler_csv(caminho):
    return pd.read_csv(caminho)

def ler_excel(caminho):
    return pd.read_excel(caminho)

def ler_json(caminho):
    with open(caminho, "r", encoding="utf-8") as f:
        data = json.load(f)
    return pd.json_normalize(data)

def ler_txt(caminho):
    return pd.read_csv(caminho, sep="\n", header=None, names=["linha"])

def ler_xml(caminho):
    tree = ET.parse(caminho)
    root = tree.getroot()

    registros = []
    for item in root:
        registro = {child.tag: child.text for child in item}
        registros.append(registro)

    return pd.DataFrame(registros)

def tratar_dataframe(df):
    # Exemplo de tratamento
    df = df.copy()
    df.columns = [col.lower().strip() for col in df.columns]
    df = df.drop_duplicates()
    df = df.fillna("")
    return df

def importar_arquivos(pasta):
    dados = {}

    for arquivo in os.listdir(pasta):
        caminho = os.path.join(pasta, arquivo)

        if arquivo.endswith(".csv"):
            df = ler_csv(caminho)

        elif arquivo.endswith(".xlsx") or arquivo.endswith(".xls"):
            df = ler_excel(caminho)

        elif arquivo.endswith(".json"):
            df = ler_json(caminho)

        elif arquivo.endswith(".txt"):
            df = ler_txt(caminho)

        elif arquivo.endswith(".xml"):
            df = ler_xml(caminho)

        else:
            continue

        dados[arquivo] = tratar_dataframe(df)

    return dados


# Execução
if __name__ == "__main__":
    resultado = importar_arquivos(PASTA)

    for nome, df in resultado.items():
        print(f"\nArquivo: {nome}")
        print(df.head())

```

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Para simplificar, podemos "injetar" os dados no prompt, garantindo  que a Agente tenha o melhor contexto possível.
Lembrando que: Em soluções mais robustas, e recomendando que as informações sejam carregadas dinamicamente para que possamos ter flexibilidade.

```txt
Dados do Cliente e Perfil (data/perfil_investidor.json):
{
  "nome": "João Silva",
  "idade": 32,
  "profissao": "Analista de Sistemas",
  "renda_mensal": 5000.00,
  "perfil_investidor": "moderado",
  "objetivo_principal": "Construir reserva de emergência",
  "patrimonio_total": 15000.00,
  "reserva_emergencia_atual": 10000.00,
  "aceita_risco": false,
  "metas": [
    {
      "meta": "Completar reserva de emergência",
      "valor_necessario": 15000.00,
      "prazo": "2026-06"
    },
    {
      "meta": "Entrada do apartamento",
      "valor_necessario": 50000.00,
      "prazo": "2027-12"
    }
  ]
}

Tranzações do Cliente (data/transacoes.csv):
data,descricao,categoria,valor,tipo
2025-10-01,Salário,receita,5000.00,entrada
2025-10-02,Aluguel,moradia,1200.00,saida
2025-10-03,Supermercado,alimentacao,450.00,saida
2025-10-05,Netflix,lazer,55.90,saida
2025-10-07,Farmácia,saude,89.00,saida
2025-10-10,Restaurante,alimentacao,120.00,saida
2025-10-12,Uber,transporte,45.00,saida
2025-10-15,Conta de Luz,moradia,180.00,saida
2025-10-20,Academia,saude,99.00,saida
2025-10-25,Combustível,transporte,250.00,saida

Histórico de Atendimento do Cliente(data/historico_atendimento.csv):

data,canal,tema,resumo,resolvido
2025-09-15,chat,CDB,Cliente perguntou sobre rentabilidade e prazos,sim
2025-09-22,telefone,Problema no app,Erro ao visualizar extrato foi corrigido,sim
2025-10-01,chat,Tesouro Selic,Cliente pediu explicação sobre o funcionamento do Tesouro Direto,sim
2025-10-12,chat,Metas financeiras,Cliente acompanhou o progresso da reserva de emergência,sim
2025-10-25,email,Atualização cadastral,Cliente atualizou e-mail e telefone,sim

Produtos disponíveis para Ensino(data/produtos_financeiros.json):
[
  {
    "nome": "Tesouro Selic",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "100% da Selic",
    "aporte_minimo": 30.00,
    "indicado_para": "Reserva de emergência e iniciantes"
  },
  {
    "nome": "CDB Liquidez Diária",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "102% do CDI",
    "aporte_minimo": 100.00,
    "indicado_para": "Quem busca segurança com rendimento diário"
  },
  {
    "nome": "LCI/LCA",
    "categoria": "renda_fixa",
    "risco": "baixo",
    "rentabilidade": "95% do CDI",
    "aporte_minimo": 1000.00,
    "indicado_para": "Quem pode esperar 90 dias (isento de IR)"
  },
  {
    "nome": "Fundo Multimercado",
    "categoria": "fundo",
    "risco": "medio",
    "rentabilidade": "CDI + 2%",
    "aporte_minimo": 500.00,
    "indicado_para": "Perfil moderado que busca diversificação"
  },
  {
    "nome": "Fundo de Ações",
    "categoria": "fundo",
    "risco": "alto",
    "rentabilidade": "Variável",
    "aporte_minimo": 100.00,
    "indicado_para": "Perfil arrojado com foco no longo prazo"
  },
  {
    "nome": "CDB Prefixado",
    "categoria": "Renda fixa",
    "risco": "Baixo (FGC)",
    "rentabilidade": "14% ao ano",
    "aporte_minimo": 100.00,
    "indicado_para": "Investidores que querem previsibilidade"
  },
  {
    "nome": "CDB Pós-fixado 115% do CDI",
    "categoria": "Renda fixa",
    "risco": "Baixo (FGC)",
    "rentabilidade": "115% do CDI (~16,1% a.a. no exemplo)",
    "aporte_minimo": 100.00,
    "indicado_para": "Quem busca rendimento acima do CDI"
  },
  {
    "nome": "LCI IPCA + 7,5% a.a.",
    "categoria": "Renda fixa isenta",
    "risco": "Baixo (FGC)",
    "rentabilidade": "IPCA + 7,5% ao ano",
    "aporte_minimo": 1000.00,
    "indicado_para": "Proteção contra inflação"
  },
  {
    "nome": "Tesouro IPCA+",
    "categoria": "Renda fixa pública",
    "risco": "Muito baixo",
    "rentabilidade": "IPCA + taxa fixa (ex.: 5% a.a.)",
    "aporte_minimo": 30.00,
    "indicado_para": "Objetivos de longo prazo"
  },
  {
    "nome": "Tesouro Prefixado",
    "categoria": "Renda fixa pública",
    "risco": "Muito baixo",
    "rentabilidade": "Taxa fixa anual (ex.: 10% a.a.)",
    "aporte_minimo": 30.00,
    "indicado_para": "Quem acredita na queda dos juros"
  },
  {
    "nome": "Tesouro Selic",
    "categoria": "Renda fixa pública",
    "risco": "Muito baixo",
    "rentabilidade": "Selic anual (ex.: 13,25% a.a.)",
    "aporte_minimo": 30.00,
    "indicado_para": "Reserva de emergência"
  },
  {
    "nome": "Debênture Incentivada",
    "categoria": "Renda fixa privada",
    "risco": "Médio",
    "rentabilidade": "IPCA + taxa fixa (ex.: IPCA + 6% a.a.)",
    "aporte_minimo": 1000.00,
    "indicado_para": "Longo prazo com isenção de IR"
  },
  {
    "nome": "Fundo DI",
    "categoria": "Fundos de investimento",
    "risco": "Baixo",
    "rentabilidade": "Próxima ao CDI (ex.: 98% a 105% do CDI)",
    "aporte_minimo": 100.00,
    "indicado_para": "Perfil conservador"
  },
  {
    "nome": "Fundo de Renda Fixa",
    "categoria": "Fundos de investimento",
    "risco": "Baixo a médio",
    "rentabilidade": "Taxa variável (ex.: 8% a 12% a.a.)",
    "aporte_minimo": 100.00,
    "indicado_para": "Diversificação simples"
  },
  {
    "nome": "Fundo Multimercado",
    "categoria": "Fundos de investimento",
    "risco": "Médio a alto",
    "rentabilidade": "Variável (ex.: 10% a 20% a.a.)",
    "aporte_minimo": 500.00,
    "indicado_para": "Investidores moderados"
  },
  {
    "nome": "Fundo de Ações",
    "categoria": "Fundos de investimento",
    "risco": "Alto",
    "rentabilidade": "Variável (ex.: 15% a 30% a.a.)",
    "aporte_minimo": 100.00,
    "indicado_para": "Longo prazo e maior risco"
  },
  {
    "nome": "ETF de Renda Variável",
    "categoria": "Bolsa de valores",
    "risco": "Alto",
    "rentabilidade": "Varia conforme o índice (ex.: 10% a 25% a.a.)",
    "aporte_minimo": 50.00,
    "indicado_para": "Diversificação na bolsa"
  },
  {
    "nome": "Ações",
    "categoria": "Renda variável",
    "risco": "Alto",
    "rentabilidade": "Variável (ex.: 5% a 40% a.a.)",
    "aporte_minimo": 1.00,
    "indicado_para": "Investidores experientes"
  },
  {
    "nome": "Fundo Imobiliário (FII)",
    "categoria": "Renda variável",
    "risco": "Médio",
    "rentabilidade": "Dividend yield (ex.: 8% a 12% a.a.)",
    "aporte_minimo": 10.00,
    "indicado_para": "Renda mensal"
  },
  {
    "nome": "CRI/CRA",
    "categoria": "Renda fixa estruturada",
    "risco": "Médio a alto",
    "rentabilidade": "IPCA + taxa fixa (ex.: IPCA + 7% a.a.)",
    "aporte_minimo": 1000.00,
    "indicado_para": "Investidores qualificados"
  }
]

```

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

O exemplo montado abaixo, se baseia nos dados originais da base de conhecimento, mas sitentiza deixando apenas as informações relevantes; desta maneira otimiza o uso de tokens. Resaltamos que o mais importante e ter todas as informações relevanntes disponiveis, ale de economizar o uso de tokens. 

```
Dados do Cliente:
- Nome: João Silva
- Perfil: Moderado
- Saldo disponível: R$ 5.000

Últimas transações:
- 01/11: Supermercado - R$ 450
- 03/11: Streaming - R$ 55
...

Produtos disponíveis para explicar:
- Tesouro IPCA+ (Muito baixo)
- Tesouro Prefixado ("Muito baixo)
- Tesouro Selic (Muito baixo)
- Debênture Incentivada (Médio)
- Fundo DI (Baixo)
- Fundo de Renda Fixa (Baixo a médio)
- Fundo Multimercado (Médio a alto)
- Fundo de Ações (Alto)
- ETF de Renda Variável (Alto)
- Ações (Alto)
- Fundo Imobiliário (FII) (Médio)
- CRI/CRA (Médio a alto)
....

```
