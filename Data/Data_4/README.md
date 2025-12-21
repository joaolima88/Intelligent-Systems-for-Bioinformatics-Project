# Dataset: Previsão de Neutralização SARS-CoV-2 (CoV-AbDab Curated)

## 1. Visão Geral

Dataset da CoV-AbDab (Oxford Protein Informatics Group) para o desenvolvimento de modelos de Deep Learning (GNNs e Transformers) capazes de prever a neutralização viral. O foco é a interação entre anticorpos humanos e variantes do vírus SARS-CoV-2.

Arquitetura Proposta:

Para lidar com a complexidade do problema, e cumprir os requisitos de integração de dados heterogéneos, adotamos uma estratégia híbrida:

 1 . Anticorpo (Estrutura 3D): Processado via Graph Neural Networks (GNNs) para capturar a geometria 3D do anticorpo.

 2. Variante Viral (Sequência/Semântica): Processada via modelos de Sequência (Embeddings ou One-Hot).

Objetivo: Prever a probabilidade de neutralização da nova variante com base nos anticorpos conhecidos

$$f(\text{Estrutura do Anticorpo}, \text{Sequência da Variante}) = P(\text{Neutralização})$$


---

## 2. Estrutura do Dataset de treino

Cada linha representa uma interação única (Um Anticorpo vs. Uma Variante Viral).


| pdb_id | chain_heavy      | chain_light      | variant_target         | label |
|--------|------------------|------------------|------------------------|-------|
| 7BWJ   | EVOLVESGGGL...   | DIOMTOSPSS...    | SARS-CoV2_Omicron-BA1  | 1     |
| 7BWJ   | EVOLVESGGGL...   | DIOMTOSPSS...    | SARS-CoV2_Delta        | 0     |
| 7K8M   | QVQLVQSGAEV...   | EIVLTQSPAT...    | SARS-CoV2_WT           | 1     |


| Variáveis | Tipo | Descrição | Uso no Modelo |
| --------- | ---- | --------- | ------------- |
| `pdb_id` | String | Identificador único da estrutura no PDB (ex: `7BWJ`). | Input GNN: Chave para carregar o grafo 3D do anticorpo. |
| `chain_heavy` | String | Sequência de aminoácidos da Cadeia Pesada (VH). | Input Sequência: Atributos dos nós ou Embedding. |
| `chain_light` | String | Sequência de aminoácidos da Cadeia Leve (VL). | Input Sequência: Atributos dos nós ou Embedding. |
| `variant_target` | String | Nome da variante do SARS-CoV-2 testada (ex: `Omicron`, `Delta`, `WT`). | Input Contexto: Define o alvo da neutralização. |
| `label` | Int | Classificação binária do fenótipo. | Target: 1 (Neutraliza) vs 0 (Não Neutraliza). |

Tipo 1: Dados Ómicos / Sequenciais (1D)

Fonte: Sequenciação de aminoácidos das regiões variáveis dos anticorpos.
Variáveis: Sequências da Cadeia Pesada ($V_H$) e Cadeia Leve ($V_L$).
Aplicação no Modelo: Utilizados como node features nos grafos ou como input para modelos de NLP (ex: ProtBERT) para capturar a composição bioquímica dos resíduos.

Tipo 2: Dados Estruturais / Geométricos (3D)

Fonte: Cristalografia de Raios-X (Ficheiros PDB).
Variáveis: Coordenadas espaciais ($x, y, z$) dos átomos (Carbonos-Alfa) do anticorpo.
Aplicação no Modelo: Utilizados para construir as Matrizes de Adjacência e arestas das Graph Neural Networks (GNNs). Isto permite ao modelo aprender padrões espaciais que não são visíveis apenas na sequência linear.

---

## 3. Limpeza preliminar

Dados originais: ~12.000 entradas

1. Removidas todas as entradas sem sequência completa (VH + VL) OU sem estrutura 3D associada.
2. Removidas todas as entradas de SARS-CoV-1 e MERS focando apenas no SARS-CoV-2
3. Remoção de entradas duplicadas (Mesmo PDB + Mesma Variante + Mesmo Resultado).

---

## 4. Estatísticas do Dataset

* Total de Amostras (Interações): ~4.199
* Estruturas Únicas (PDBs): ~473, Cada estrutura PDB aparece, em média, em 8.8 interações diferentes (testada contra variantes diferentes).
* Equilíbrio de Classes: Neutraliza (1) ~64%,  Não Neutraliza (0) ~36%

---

## 5. Desafios e Limitações


### 1. Risco de Data Leakage

Problema: Como temos 473 PDBs para 4.199 amostras, a mesma estrutura 3D repete-se em múltiplas linhas. Se fizermos um random split simples, o modelo pode ver o mesmo anticorpo no treino e novamente no teste. A validação cruzada deve usar GroupKFold ou Cluster-based Split, agrupando pelo `pdb_id`, para garantir que o modelo nunca viu a estrutura do conjunto de teste.


### 2. Representação da Variante Viral

Atualmente, a variante viral é representada pelo nome (`variant_target`). O modelo pode ter dificuldade em generalizar para variantes novas se apenas memorizar os nomes, para resolver isto podemos substituir o nome da variante pela sequência da proteína Spike correspondente ou pelas mutações específicas que a definem.


## 6. Link

https://opig.stats.ox.ac.uk/webapps/covabdab/

Procurar por "Database (CSV)" e "PDB Structures (.tar.gz)"

