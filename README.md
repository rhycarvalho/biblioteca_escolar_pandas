# Biblioteca escolar: empréstimos e atrasos

Projeto pequeno de análise de dados. Peguei um CSV com os empréstimos de uma biblioteca escolar, tratei no Python (Pandas) e organizei o resultado em uma planilha de Excel.

A pergunta que guiou o trabalho: quais livros ainda não voltaram, há quanto tempo estão fora e como os empréstimos se dividem por turma, livro e gênero?

## Dados

`dados/biblioteca_escolar.csv` tem 20 empréstimos de outubro de 2026, um por linha, com ponto e vírgula como separador.

| Coluna | O que é |
|---|---|
| aluno | Nome do aluno |
| turma | 1A, 1B, 2A, 2B, 3A ou 3B |
| livro | Título emprestado (6 títulos) |
| genero | Aventura, Conto, Fantasia ou Ficção |
| data_emprestimo | Dia em que o livro saiu |
| data_devolucao | Dia da devolução (vazia se ainda não voltou) |
| status | Devolvido ou Em andamento |

## Como foi feito

Li o arquivo com `sep=";"` e olhei tipos e valores vazios. As datas vieram como texto e `data_devolucao` tinha 7 vazios. Comparei com a coluna `status` e eram exatamente os empréstimos em andamento, então mantive os vazios: o livro ainda não voltou, e esses são os registros que mais interessam.

Converti as duas datas com `pd.to_datetime` e criei duas colunas: `dias_emprestado`, para quem já devolveu, e `dias_pendente`, para quem ainda está com o livro. Os dias pendentes são contados até 01/11/2026, uma data fixa para o resultado não mudar a cada vez que o notebook roda. Cheguei a fazer uma coluna única para os dois casos, mas ela misturava situações diferentes, então separei.

Depois contei por turma, livro e gênero com `value_counts` e `groupby`, fiz dois gráficos com Matplotlib e montei um resumo por turma com os nomes dos alunos. Por fim, exportei os CSVs e organizei tudo no Excel.

```python
df = pd.read_csv("dados/biblioteca_escolar.csv", sep=";")

df["data_emprestimo"] = pd.to_datetime(df["data_emprestimo"])
df["data_devolucao"] = pd.to_datetime(df["data_devolucao"])

df["dias_emprestado"] = (df["data_devolucao"] - df["data_emprestimo"]).dt.days

data_ref = pd.Timestamp("2026-11-01")
df["dias_pendente"] = (
    (data_ref - df["data_emprestimo"]).dt.days.where(df["status"] == "Em andamento")
)
```

## Resultados

- Dos 20 empréstimos, 13 foram devolvidos e 7 continuam em andamento (35%).
- Todos os devolvidos ficaram exatamente 7 dias com o livro.
- Se o prazo for de 7 dias, os 7 pendentes estão atrasados, de 5 a 21 dias. O maior atraso é do Pedro (2B), com "O Pequeno Príncipe". Esse prazo é suposição minha, tirada dos devolvidos: não aparece no arquivo.
- Aventura é o gênero mais emprestado, com 10 dos 20 empréstimos e 5 dos 7 pendentes. Com 20 linhas, serve só como ponto de atenção.
- 1A e 1B têm 2 pendentes cada, e a 2A não tem nenhum.
- "O Pequeno Príncipe" e "O Sítio das Descobertas" têm 2 pendentes cada.

## Excel

`biblioteca_escolar_formatado.xlsx` tem cinco abas:

| Aba | Conteúdo |
|---|---|
| Resumo | Números gerais e tabelas por turma, gênero e livro, com um gráfico por turma |
| Resumo por turma | Devolvidos e em andamento por turma, com os nomes |
| Empréstimos | Base tratada, com os dias calculados por fórmula |
| Pendentes | Livros que ainda não voltaram, com dias fora e atraso |
| Gráficos | Os dois gráficos feitos no Python |

Na aba Resumo, a data de hoje e o prazo são células editáveis (em amarelo). Mudando esses valores, os atrasos das abas Empréstimos e Pendentes se atualizam.

## Estrutura

```
biblioteca-escolar/
├── README.md
├── exploracao_biblioteca.ipynb
├── biblioteca_escolar_formatado.xlsx
├── requirements.txt
├── dados/
│   ├── biblioteca_escolar.csv
│   ├── biblioteca_escolar_modificado.csv
│   └── resumo_por_turma.csv
└── graficos/
    ├── grafico_genero.png
    └── grafico_pendentes_turma.png
```

## Ferramentas

Python (Pandas, Matplotlib) e Excel.
