# TP01 — Integração de marcações e pagamentos

Autora: Thifany Antoni  
Número de aluno: 16077  
Curso: Engenharia de Sistemas Informáticos — IPCA  
Unidade curricular: Integração de Sistemas de Informação — 2026/2027  
Docente: Óscar Ribeiro

## Vídeo e repositório

Vídeo de demonstração: [Abrir vídeo](https://drive.google.com/file/d/16tJznyzWdmZY6JnCvfCfpeTh9yyD8EKp/view?usp=sharing)  
Repositório Git: [tp01-16077_ISI](https://github.com/ThifanyAntoni/tp01-16077_ISI)

## Objetivo

Integrar dados fictícios de um estúdio de estética provenientes de CSV, JSON, XML e SQLite, limpar e validar os registos e exportar detalhe integrado, resumo mensal e marcações para revisão.

## Organização dos ficheiros

- `doc/16077_doc.pdf`: relatório com link e QR Code do vídeo.
- `dataintegration/16077_TP1_Estudio_ETL.knwf`: workflow KNIME exportado, incluindo dados internos e resultados executados.
- `data/input/`: cópias de consulta das quatro fontes fictícias: marcacoes.csv, pagamentos.json, servicos.xml e clientes.sqlite.
- `data/output/`: exemplos dos três CSV gerados na execução exportada.

O processamento foi implementado nos nós KNIME; não existe código de aplicação separado em `src/`.

## Requisitos e execução

1. Usar KNIME Analytics Platform 5.12.0, versão usada no desenvolvimento, com suporte aos nós JSON, XML e de base de dados. Se a importação indicar extensões em falta, instalar as extensões correspondentes.
2. Importar `dataintegration/16077_TP1_Estudio_ETL.knwf` para o espaço local do KNIME, usando a opção de importar workflow.
3. Abrir o workflow importado. As quatro fontes estão incluídas no KNWF em `data/input`, e os caminhos referem a própria área do workflow, por exemplo `knime://knime.workflow/data/input/marcacoes.csv`.
4. Para reprocessar todas as fontes, usar Reset no workflow e executar os três CSV Writer, permitindo a execução dos antecessores. Os ramos independentes têm de terminar para obter as três saídas.
5. Consultar os ficheiros na área de dados do workflow importado, em `data/output`. Os CSV Writer estão configurados para substituir os ficheiros anteriores, com separador `;`, UTF-8 e cabeçalhos.

As pastas externas `data/input` e `data/output` são cópias de consulta. Executar o workflow importado atualiza as suas saídas internas, não estas cópias externas. Para mudar as fontes, substituir as cópias internas ou alterar as configurações dos leitores.

## Regras do tratamento

- Comparar as cinco colunas das marcações para remover cópias exatamente iguais.
- Separar marcações sem valor para revisão, sem lhes atribuir preço zero ou preço base automaticamente.
- Somar pagamentos por id_marcacao antes da junção, pois uma marcação pode ter vários pagamentos.
- Preservar as marcações nas junções; substituir apenas a soma de pagamento ausente por zero, assumindo que o ficheiro de pagamentos está completo.
- Calcular valor_em_falta = valor_servico - total_pago.
- Normalizar espaços e maiúsculas dos emails e validar um formato básico com Regex; isto não verifica a existência do endereço.
- Retirar nome e emails da exportação integrada. Os identificadores continuam relacionáveis com a origem; não se afirma anonimização completa.
- Agrupar os totais pelo mês da marcação, não pela data do pagamento.

## Resultados de controlo

125 linhas na origem; 5 cópias exatas removidas; 120 marcações distintas; 5 sem preço separadas; 115 marcações analisadas. Os 118 pagamentos produzem 88 grupos. Há 27 marcações analisadas sem pagamento associado.

| Saída | Linhas | Colunas |
| --- | ---: | ---: |
| marcacoes_integradas.csv | 115 | 14 |
| resumo_mensal.csv | 3 | 6 |
| marcacoes_para_revisao.csv | 5 | 5 |

| Mês de 2026 | Marcações | Serviços (€) | Pago (€) | Em falta (€) |
| --- | ---: | ---: | ---: | ---: |
| Agosto | 35 | 1000 | 820 | 180 |
| Setembro | 38 | 1045 | 765 | 280 |
| Outubro | 42 | 2030 | 1297,50 | 732,50 |
| Total | 115 | 4075 | 2882,50 | 1192,50 |

Em cada linha da tabela integrada verifica-se valor_servico = total_pago + valor_em_falta. Existem 115 IDs distintos e nenhum campo vazio nessa saída. O estado de email é Rever em 8 marcações de 2 clientes distintos. Os registos para revisão são M004, M030, M060, M090 e M120.

## Limitações

O workflow foi executado no computador de desenvolvimento. A cópia exportada foi importada e reexecutada após Reset no mesmo computador, terminando com todos os nós a verde. Não foi testada a execução em outro computador.
