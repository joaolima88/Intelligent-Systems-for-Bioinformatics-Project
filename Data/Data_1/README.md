# Descrição

## 1. Fontes de Dados

Para cumprir o objetivo de integrar dados biológicos de naturezas distintas, são utilizadas duas fontes primárias de dados:

### A. Dados Genómicos e Fenotípicos (Deep Mutational Scanning)
- Fonte: Starr et al. (2022) - https://github.com/jbloomlab/SARS-CoV-2-RBD_DMS_Omicron/tree/main/results/binding_Kd
- Descrição: Dataset gerado por Deep Mutational Scanning da variante ancestral (Wuhan-Hu-1). Este estudo mediu experimentalmente o impacto funcional de quase todas as mutações possíveis no complexo RBD.
- Utilidade: A medida real de quão bem cada variante se liga ao recetor ACE2.

### B. Dados Estruturais (Cristalografia de Raios-X)
- Fonte: RCSB Protein Data Bank - PDB ID: 6M0J - https://www.rcsb.org/structure/6M0J
- Descrição: A estrutura cristalina de referência do complexo SARS-CoV-2 RBD ligado ao recetor humano ACE2.
- Utilidade: Serve como "mapa espacial". Permite ao modelo compreender o contexto 3D de cada mutação (ex: se o resíduo mutado está na superfície de contacto com o ACE2 ou enterrado no núcleo da proteína), algo que a sequência linear sozinha não revela.

---

## 2. Variáveis do Modelo

O modelo de Machine Learning é treinado como um problema de Regressão Supervisionada, mapeando características moleculares para um fenótipo funcional.

### Variáveis de Entrada ($X$)
O modelo recebe dois tipos de inputs para cada variante viral:
1.  Sequência Completa do RBD: A string completa de 211 aminoácidos contendo a mutação específica. Esta sequência é processada para extrair propriedades bioquímicas e evolutivas (ex: via -Embeddings- do ProtBERT).
2.  Contexto Estrutural: Informação extraída do PDB 6M0J correspondente à posição onde a mutação ocorreu. Inclui:
    - Distância euclidiana ao recetor ACE2.
    - Acessibilidade ao solvente (SASA).
    - Variação de volume e carga atómica.

### Variável de Saída ($Y$)
- Alvo: Afinidade de Ligação ($\Delta \log_{10} K_a$).
- Interpretação:
    - Valores mais altos indicam uma ligação mais forte ao recetor humano (maior infeciosidade potencial).

    - Valores mais baixos indicam perda de função.

