# SME0211 — Otimização Linear: Balanceamento de Carga

Trabalho final da disciplina **SME0211 - Otimização Linear (2024)** do ICMC-USP São Carlos.

**Autores:**
- Francisco Maian — 14570890
- Gustavo Moreira — 5244057
- Julia Pravato — 14615054
- Murilo Lirani — 11234673

---

## Descrição do Problema

O projeto modela um **problema de balanceamento de carga** como um programa linear. Dado um conjunto de tarefas e servidores, o objetivo é distribuir as tarefas entre os servidores de forma a **maximizar a relevância total** de alocação, respeitando as capacidades de cada servidor e as quantidades disponíveis de cada tarefa.

A formulação matemática resulta em:

```
max  r^T x
s.a. A x ≤ b
     x ≥ 0
```

que é convertida para a **forma padrão** com variáveis de folga antes de ser resolvida.

---

## Estrutura do Repositório

```
.
├── CódigoOL.ipynb      # Notebook principal com implementação e experimentos
├── RelatórioOL.pdf     # Relatório completo do projeto
└── README.md
```

---

## Dependências

- Python 3.x
- NumPy
- Pandas
- SciPy
- Matplotlib
- Plotly
- Seaborn

Instale todas com:

```bash
pip install numpy pandas scipy matplotlib plotly seaborn
```

---

## Como Executar

Abra o notebook `CódigoOL.ipynb` no Jupyter ou Google Colab e execute as células em ordem. Os experimentos foram originalmente rodados no **Google Colaboratory**.

---

## Organização do Código

### Geração do Problema

**`calc_beta_params(mean, std)`**
Converte média e desvio padrão em parâmetros α e β de uma distribuição Beta, usada para gerar os limites de capacidade dos servidores.

**`gerar_tensor_aleatorio(size, B_params, sparsity)`**
Gera uma matriz (ou tensor) de relevâncias aleatórias e os vetores de limites correspondentes. O parâmetro `sparsity` controla a fração de elementos zerados.

**`traducao_variaveis(relevancia, limites)`**
Transforma o tensor de relevâncias em um vetor linearizado (`ravel`) e constrói a matriz de restrições `A` e o vetor de limites `b` no formato `Ax ≤ b`.

**`forma_padrao(A, b, r)`**
Converte o problema para a forma padrão de minimização (`min c^T x`, `Ax = b`, `x ≥ 0`), adicionando variáveis de folga e negando os custos para transformar a maximização em minimização. Variáveis com custo nulo são removidas automaticamente.

### Algoritmos de Resolução

**`simplex(c, A, b)`** — Simplex com Regra de Bland
Implementação manual do algoritmo Simplex com a **Regra de Bland** para evitar ciclagem. A regra garante que, em caso de empate, sempre é escolhida a variável de menor índice. Retorna o custo ótimo, a solução e o número de iterações.

**`linprog` (SciPy)** — Simplex Revisado
Utiliza `scipy.optimize.linprog` com `method='simplex'`, que internamente implementa o **Simplex Revisado** usando bibliotecas C (BLAS/LAPACK) para maior eficiência numérica.

### Experimentos

**`teste_singular(size, B_params, sparsity)`**
Gera um problema aleatório e resolve com ambos os métodos, retornando custo ótimo, número de iterações e tempo de execução para cada um.

**`amostragem(sample_size, size, B_params, sparsity)`**
Executa `teste_singular` múltiplas vezes e agrega os resultados em um DataFrame.

**`experimentos(sample_size, sizeS, beta_paramsS, sparsityS)`**
Varre listas de parâmetros (`size`, `B_params`, `sparsity`) chamando `amostragem` para cada configuração e consolida todos os resultados.

---

## Resultados Principais

- Ambos os algoritmos atingem o **mesmo custo ótimo**.
- O **SciPy** é consistentemente mais rápido para problemas grandes, graças à implementação em C.
- O **Simplex com Bland** apresenta maior variabilidade nas iterações, mas pode ser competitivo em problemas pequenos.
- A **esparsidade** reduz o tempo de execução de forma aproximadamente linear em ambos os métodos.
- Os parâmetros da distribuição Beta (média e desvio padrão das restrições) não afetam significativamente o tempo de execução.
- Para `R₅₀ₓ₅₀`, o Simplex Bland convergiu em ~10 minutos (4449 iterações), enquanto o SciPy atingiu o limite de 1000 iterações sem convergir.

---

## Tamanho do Problema

Para uma matriz de relevâncias `R(n×m)`, o problema na forma padrão tem dimensões:

- **Variáveis:** `nm(1-s) + n + m`
- **Restrições:** `n + m`

onde `s` é a esparsidade de R. A complexidade esperada no pior caso é:

```
O((n+m) · (nm(1-s) + n+m))
```

---

## Referências

- Marina Andretta (2024). *Slides de Otimização Linear*. USP São Carlos.
- SciPy Documentation. [`scipy.optimize.linprog`](https://docs.scipy.org/doc/scipy/reference/optimize.linprog-simplex.html)
- Bertsimas, D. & Tsitsiklis, J. N. (1997). *Introduction to Linear Optimization*. Athena Scientific.
- Pan, V. (1985). *On the Complexity of a Pivot Step of the Revised Simplex Algorithm*. Computers & Mathematics with Applications.
