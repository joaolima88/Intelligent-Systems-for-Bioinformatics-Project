# Documentação do Dataset: Previsão de Aptidão Antigénica (H3N2)

**Projeto:** Sistemas Inteligentes para a Bioinformática (2025/2026)
**Foco:** Modelação Molecular e Evolução Viral
**Subtipo Viral:** Influenza A (H3N2)

## 1. Visão Geral e Objetivo

O objetivo deste dataset é treinar modelos de Machine Learning (Deep Learning) para prever a **Aptidão Viral (Fitness)**, definida aqui como a capacidade de escape imune. O modelo deverá aprender a mapear a sequência de aminoácidos da proteína Hemaglutinina (HA) para um valor de distância antigénica (ou título HI), sem depender de alinhamentos filogenéticos prévios.

## 2. Conformidade com os Requisitos do Projeto

Para cumprir a exigência de "integrar inputs de pelo menos dois tipos de dados biológicos distintos", este projeto articula:

1.  **Dados Genómicos (Input $X$):** Sequências lineares de aminoácidos provenientes do NCBI, representando a variabilidade genética das estirpes.
2.  **Dados Bioquímicos/Serológicos (Target $y$):** Títulos de Inibição da Hemaglutinação (HI Assays), que quantificam a interação funcional (afinidade) entre o vírus e anticorpos.
3.  **Dados Físico-Químicos (Feature Engineering):** Enriquecimento do input com propriedades moleculares (hidrofobicidade, carga, volume - via AAIndex) e anotações estruturais (epitopos conhecidos via PDB), satisfazendo a integração de dados estruturais na pipeline.

## 3. Arquitetura do Dataset Final (Target)

Após o processamento e limpeza, o ficheiro final (`dataset_final_h3n2.csv`) terá a seguinte estrutura tabular:

| Coluna (Nome) | Tipo de Dado | Descrição (Papel no Modelo) |
| :--- | :--- | :--- |
| `virus_strain` | String | Identificador único da variante viral (Ex: `A/Wisconsin/67/2005`). Usado para rastreio. |
| `antiserum_strain` | String | Identificador da estirpe usada para gerar o antissoro/vacina. |
| **`ha_sequence`** | **String** | **Variável de Entrada ($X$)**. A sequência completa ou parcial (domínio HA1) de aminoácidos da Hemaglutinina. |
| **`hi_titer`** | **Integer** | **Variável Alvo ($y$)**. O Título de Inibição da Hemaglutinação (Ground Truth Experimental). |
| `collection_year` | Integer | Metadado temporal (1968-2011). Essencial para validação "Time-Series Split". |

> **Nota:** O problema pode ser modelado como Regressão (prever o valor exato do título) ou Classificação Binária (prever se Título < 40, indicando escape imune).

## 4. Fonte dos Dados (Data Provenance)

Estamos a utilizar uma abordagem híbrida para contornar restrições de licenciamento:

1.  **Fenótipos (Rótulos $y$):**
    * **Fonte:** Bedford et al. (2014), repositório Dryad.
    * **Ficheiro:** `H3N2_HI_data.txt`.
    * **Conteúdo:** Matriz esparsa de interações vírus-soro (Dados Bioquímicos).

2.  **Genótipos (Sequências $X$):**
    * **Fonte:** NCBI GenBank (Influenza Virus Resource).
    * **Método:** Script Python (`Biopython`) que consulta o NCBI usando os nomes das estirpes.
    * **Motivo:** O dataset original fornece apenas IDs GISAID (acesso restrito).

## 5. Volume e Amostragem

* **Total de Registos:** Estimado entre 5.000 e 10.000 pares de interação.
* **Período Temporal:** 1968 a 2011.
* **Diversidade:** Foco exclusivo no subtipo **H3N2** devido à sua elevada taxa de mutação (*Antigenic Drift*), proporcionando um sinal evolutivo forte.

## 6. Desafios Técnicos e Estratégias de Mitigação

### A. A Barreira GISAID vs. NCBI (Missing Data)
* **Problema:** Estirpes listadas no dataset de fenótipos podem existir apenas na base privada GISAID.
* **Mitigação:** Remoção dessas linhas (<15% de perda estimada).

### B. Adaptação ao Ovo ("Egg-Adaptation")
* **Problema:** Vírus cultivados em ovos ganham mutações artificiais. O NCBI pode conter a versão "adaptada" enquanto o título HI corresponde ao vírus clínico.
* **Mitigação:** Aceitação do ruído estatístico. Onde possível, priorizar sequências marcadas como "clinical isolate", mas assumir a mais prevalente como *proxy* válido.

### C. Redundância e Ambiguidade
* **Problema:** Múltiplas sequências para a mesma estirpe no NCBI.
* **Mitigação:** Algoritmo de seleção: Priorizar "Full Length CDS" e sequências mais longas (>300aa).

### D. Inconsistências Tipográficas
* **Problema:** `A/Wisconsin/67/05` (TXT) vs `A/Wisconsin/67/2005` (NCBI).
* **Mitigação:** Normalização de strings (Regex) antes da consulta à API.

## 7. Próximos Passos (Pipeline)

1.  Desenvolver script Python para ler `H3N2_HI_data.txt`.
2.  Extrair lista de estirpes únicas.
3.  Executar *batch query* ao NCBI Entrez API.
4.  Validar qualidade das sequências.
5.  Unir (Merge) as sequências com os títulos HI.
6.  Exportar `dataset_final_h3n2.csv`.