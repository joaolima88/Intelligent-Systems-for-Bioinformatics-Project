# Projeto SIB 2025: Modelo Computacional para Previsão de Aptidão Viral

Modelar a Virulência/Replicabilidade, usando a Carga Viral (Ct Value) como proxy para a aptidão da variante. Hipótese: variantes mais "aptas" geram cargas virais mais altas e assinaturas transcriptómicas específicas no hospedeiro.

## 1. Dados
Múltiplas fontes públicas: dados clínicos do GEO + dados de sequenciação do NCBI.

### Dataset Principal
* Fonte: GEO GSE152075 (Lieberman et al., 2020). https://pubmed.ncbi.nlm.nih.gov/32898168/ https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE152075
* Tipo: RNA-seq de zaragatoas nasofaríngeas (in situ).
* Conteúdo: Pacientes infetados com a variante Ancestral (Wuhan/D614G).
* Target: Carga Viral (medida por PCR `N1_Ct`).
* Volume: 413 Pacientes com dados completos (após limpeza) -> é um bocado arriscado porque podemos ter overfitting se usarmos redes neuronais ou deep learning.

### Dataset Complementar (É preciso extrair as variantes e fazer uma limpeza)
* Fonte: GEO GSE198449 (CHARM Study). https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE198449
* Conteúdo: Pacientes infetados com variantes Delta e Omicron.
* Objetivo: 1) Permitir ao modelo aprender diferenças entre variantes. 2) Potencial de Expansão: +100 a 200 amostras (Delta/Omicron - dependente do sucesso na extração do GSE198449).
* Status: Ficheiros brutos identificados; fase de extração de metadados em curso para mapear IDs às variantes.

### Fonte Genómica (Externa)
* Fonte: NCBI Virus / GenBank.
* Dados: Sequências de referência da proteína Spike para as variantes Ancestral, Delta e Omicron.

### Dataset final
(Dataset Principal + Dataset Complementar) + Fonte Genómica

## 2. Variáveis (Architecture)

O modelo recebe dois inputs paralelos (Multi-Modal) para prever uma saída contínua.

### Inputs ($X$)
1.  Hospedeiro (Transcriptómica):
    * *Dados:* Matriz de contagens de genes (~35.000 genes).

2.  Vírus (Genómica):
    * *Dados:* Sequência de aminoácidos da proteína Spike.
    * *Processamento:* Geração de Embeddings numéricos usando NLP (ProtBERT). Isto permite representar a biologia da mutação matematicamente.

### Output ($Y$)
* Target: Carga Viral (Cycle Threshold - Ct).


## 3. Problemas

* Complexidade de Integração: Unir datasets de estudos diferentes (GSE152075 + GSE198449) exige lidar com "Batch Effects" (ruído técnico entre laboratórios).
* Disponibilidade de Metadados: problemas na extração de etiquetas (Delta e Omicron) no segundo dataset devido à formatação dos ficheiros do GEO.
* Desequilíbrio: Temos mais dados da variante Ancestral (413) do que das variantes novas.