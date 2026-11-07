# FIA-P2 — Desempenho em Matemática no Ensino Médio (SAEB 2021 → 2023)

2º Trabalho de **Fundamentos de Inteligência Artificial** — Instituto de Computação (IComp), Universidade Federal do Amazonas (UFAM). Professor: Edjard Mota.

**Autores:** Ana S. B. Silva, Davi T. de Moraes, Gabriel J. L. dos Santos, Sebastian L. Guedes e Vinicius O. de Souza.

O trabalho investiga se a média de Matemática da última série do Ensino Médio (`MEDIA_12_MT`) de cada município **melhorou** entre o SAEB 2021 e o SAEB 2023, usando o IDHM-Renda de 2010 (Censo/Atlas) como contexto socioeconômico. Os métodos cobrem seis etapas: árvores de decisão, modelos lineares, probabilidade, Naive Bayes com EM, redes bayesianas e lógica fuzzy.

O texto completo, com metodologia, tabelas e discussão, está em [`Relatório 2 Avaliação de FIA.pdf`](Relat%C3%B3rio%202%20Avalia%C3%A7%C3%A3o%20de%20FIA.pdf).

---

## 1. Estrutura do repositório

```
FIA-P2/
├── Dados/
│   ├── saeb_2021_brasil_estados_municipios.xlsx                  # SAEB 2021 (INEP)
│   ├── Resultados_Saeb_2023_Brasil_Estados_Municipios.xlsb       # SAEB 2023 (INEP)
│   ├── censo_total_1991_2010(MUN 91-00-10).csv                   # Atlas/Censo (IDHM_R de 2010)
│   └── saeb_2025_brasil_estados_municipios_censitario.xlsx       # NÃO é usado por nenhum notebook
├── Notebooks/
│   ├── etapa1.ipynb   # Árvore de decisão (ID3 do zero) + monotonicidade
│   ├── etapa2.ipynb   # Regressão linear/multivariada, Ridge e logística
│   ├── etapa3.ipynb   # Probabilidade: conjunta, Bayes, independência
│   ├── etapa4.ipynb   # MLE/MAP, Naive Bayes e EM (variável oculta)
│   ├── etapa5.ipynb   # Rede Bayesiana (inferência por enumeração vs. pgmpy)
│   └── etapa6.ipynb   # Sistema fuzzy Mamdani + pipeline integrado
├── Relatório 2 Avaliação de FIA.pdf
└── README.md
```

## 2. Como os dados chegam aos notebooks (leia antes de rodar)

Cada notebook carrega o conjunto de análise de uma de duas formas:

| Modo | Quando ocorre | Etapas |
|---|---|---|
| **Embutido** (padrão) | O conjunto já filtrado e cruzado (SAEB 2021 + SAEB 2023 + IDHM_R) está salvo dentro do notebook (gzip + base64). Não precisa de nenhum arquivo de `Dados/`. | Todas (1 a 6) |
| **Brutos** (opcional) | Apenas a **Etapa 1** sabe reconstruir o conjunto a partir dos arquivos originais de `Dados/`. | Somente 1 |

Ou seja, **para reproduzir os resultados das Etapas 1 a 6 basta ter os notebooks**. O modo "brutos" serve para auditar a construção do conjunto de dados.

## 3. Requisitos

- **Python 3.12 ou 3.13** (os notebooks foram salvos com 3.12.3 e 3.13).
- Jupyter (JupyterLab, Notebook, VS Code ou Google Colab).
- Pacotes:

| Pacote | Usado em | Observação |
|---|---|---|
| `numpy`, `pandas`, `matplotlib` | todas | |
| `scipy` | 4, 5 | |
| `scikit-learn` | 1, 2, 4, 6 | usado só para comparar com as implementações próprias e para métricas |
| `openpyxl` | 1 (modo brutos) | leitura dos `.xlsx` |
| `pyxlsb` | 1 (modo brutos) | leitura do `.xlsb` do SAEB 2023 |
| `pgmpy` | 5 | precisa de versão recente (os notebooks importam `DiscreteBayesianNetwork`) |
| `scikit-fuzzy` | 6 | importado como `skfuzzy` |

Instalação em ambiente virtual:

```bash
git clone https://github.com/davitel2025/FIA-P2.git
cd FIA-P2

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install numpy pandas matplotlib scipy scikit-learn openpyxl pyxlsb \
            pgmpy scikit-fuzzy jupyterlab
```

As Etapas 1, 5 e 6 tentam instalar `pyxlsb`, `pgmpy` e `scikit-fuzzy` sozinhas se não os encontrarem, mas isso exige internet. Instalar tudo antes evita surpresas.

## 4. Reprodução padrão (modo embutido)

Todos os notebooks são independentes entre si e podem ser rodados em qualquer ordem. A ordem 1 → 6 segue o relatório.

**Interativo:**

```bash
jupyter lab
# abra Notebooks/etapa1.ipynb ... etapa6.ipynb e use "Run All"
```

**Linha de comando (sem abrir o navegador):**

```bash
mkdir -p saida
jupyter nbconvert --to notebook --execute Notebooks/etapa1.ipynb --output-dir saida
# repita para etapa2 ... etapa6
```

**Google Colab:** envie o notebook desejado e execute tudo. Não é preciso enviar `Dados/`.

Todas as sementes são fixas (`SEED = 42`), e a validação cruzada é um **5-fold agrupado por município**, com as mesmas partições em todas as etapas. Por isso os números devem coincidir com os abaixo.

### Resultados esperados

| Etapa | Métrica | Valor esperado |
|---|---|---|
| Conjunto | Exemplos / proporção de melhora | 3.519 / 0,631 |
| 1 | Entropia na raiz; atributo da raiz | 0,950 bit; `FAIXA_MEDIA_2021` (IG = 0,169) |
| 1 | Acurácia 5-fold, ID3 próprio / scikit-learn (podada) | 0,739 / 0,738 |
| 1 | Acurácia de treino sem poda; profundidade do ID3 | 0,776; 5 |
| 2 | R² 5-fold: univariada / multivariada / Ridge (λ ≈ 14) | 0,254 / 0,287 / 0,287 |
| 2 | RMSE: univariada / multivariada | 11,10 / 10,86 |
| 2 | Acurácia da regressão logística | 0,741 |
| 3 | P(M=1), P(B=1), P(M=1 \| B=1), P(M=1 \| B=0) | 0,631; 0,483; 0,846; 0,430 |
| 4 | Naive Bayes puro, 5-fold | 0,712 |
| 4 | Naive Bayes + EM, K=2 / K=3 | 0,726 / 0,734 |
| 5 | Rede Bayesiana como classificador (17 parâmetros) | 0,697 |
| 6 | Fuzzy vs. limiar rígido (tabela do relatório) | AUC 0,880 (fuzzy) vs. 0,178 (limiar da rede) para "média 2023 < 265" |
| Baseline | Classe majoritária | 0,631 |

Ao rodar, as células que imprimem tabelas em LaTeX (`... \\`) reproduzem diretamente as tabelas do relatório.

## 5. Fontes dos dados

- **SAEB 2021 e 2023:** INEP, resultados por município. <https://www.gov.br/inep/pt-br/areas-de-atuacao/avaliacao-e-exames-educacionais/saeb/resultados>
- **IDHM-Renda 2010 (`IDHM_R`):** Atlas do Desenvolvimento Humano no Brasil (PNUD, Ipea, Fundação João Pinheiro). <https://www.atlasbrasil.org.br/consulta/planilha>

## 6. Uso de LLMs

Modelos de linguagem (GPT, Claude e Gemini) foram usados como apoio à formatação do relatório em LaTeX e à implementação dos notebooks. Os resultados das implementações próprias foram conferidos contra `scikit-learn` e `pgmpy`.
