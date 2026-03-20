# Relatório — Processamento Paralelo de Logs

---

| Campo      | Informação                        |
|------------|-----------------------------------|
| Disciplina | Computação Paralela e Distribuída |
| Aluno      | Samuel Leal de Araujo             |
| Professor  | Rafael Marconi Ramos              |
| Data       | 20/03/2026                        |

---

## 1. Descrição do Problema

O problema consiste no processamento de grandes volumes de arquivos de log em formato texto, com o objetivo de extrair informações relevantes para análise gerencial.

O programa realiza a leitura de múltiplos arquivos `.txt`, contabilizando:

- número total de linhas
- número total de palavras
- número total de caracteres
- frequência de palavras-chave (`erro`, `warning`, `info`)

O algoritmo utiliza o modelo **produtor-consumidor com buffer limitado**, implementado com `multiprocessing.Pool`. A lista de arquivos é distribuída automaticamente entre os processos via `pool.map`, e cada processo opera de forma independente com seu próprio interpretador Python.

- **Objetivo do programa:** processar e consolidar estatísticas de 1000 arquivos de log de forma paralela
- **Volume de dados:** 1000 arquivos, totalizando ~10 milhões de linhas e ~1,37 GB de caracteres
- **Algoritmo utilizado:** pool de processos com distribuição automática de tarefas (`multiprocessing.Pool.map`)
- **Complexidade aproximada:** `O(N)`, onde `N` é o número total de linhas processadas

---

## 2. Ambiente Experimental

| Item | Descrição |
|------|-----------|
| Processador | Intel Core i5-12500 (12ª Geração) |
| Número de núcleos | 12 núcleos lógicos (6 P-cores + 4 E-cores) |
| Memória RAM | 16 GB |
| Sistema Operacional | Windows 11 |
| Linguagem utilizada | Python 3.14 |
| Biblioteca de paralelização | `multiprocessing.Pool` |
| Versão do interpretador | Python 3.14 (CPython) |

---

## 3. Metodologia de Testes

O tempo de execução foi medido com a função `time.time()`, registrando o instante antes e após o processamento completo de todos os arquivos.

Para cada configuração, foi realizada **uma execução**. Os testes foram executados em ambiente local, sem controle rigoroso de carga do sistema.

**Configurações testadas:**

- 1 processo *(baseline)*
- 2 processos
- 4 processos
- 8 processos
- 12 processos

**Entrada utilizada:** pasta `log2`, contendo 1000 arquivos de log.

---

## 4. Resultados Experimentais

| Nº Processos | Tempo de Execução (s) |
|:------------:|:---------------------:|
| 1            | 85.8524               |
| 2            | 47.2071               |
| 4            | 27.5248               |
| 8            | 17.1923               |
| 12           | 15.7192               |

---

## 5. Cálculo de Speedup e Eficiência

**Speedup:**
```
Speedup(p) = T(1) / T(p)
```

Onde:
- `T(1)` = tempo da execução com 1 processo (baseline)
- `T(p)` = tempo com `p` processos

**Eficiência:**
```
Eficiência(p) = Speedup(p) / p
```

Onde:
- `p` = número de processos

---

## 6. Tabela de Resultados

| Processos | Tempo (s) | Speedup | Eficiência |
|:---------:|:---------:|:-------:|:----------:|
| 1         | 85.8524   | 1.000   | 1.000      |
| 2         | 47.2071   | 1.818   | 0.909      |
| 4         | 27.5248   | 3.119   | 0.780      |
| 8         | 17.1923   | 4.993   | 0.624      |
| 12        | 15.7192   | 5.462   | 0.455      |

---

## 7. Gráfico de Tempo de Execução

> *(Inserir gráfico)*

- **Eixo X:** número de processos
- **Eixo Y:** tempo de execução (segundos)

---

## 8. Gráfico de Speedup

> *(Inserir gráfico)*

- **Eixo X:** número de processos
- **Eixo Y:** speedup
- Incluir linha de speedup ideal (linear) para comparação

---

## 9. Gráfico de Eficiência

> *(Inserir gráfico)*

- **Eixo X:** número de processos
- **Eixo Y:** eficiência (valores entre 0 e 1)

---

## 10. Análise dos Resultados

Os resultados demonstram que o uso de `multiprocessing.Pool` trouxe **ganho real e expressivo de desempenho**. O tempo de execução caiu de **85.85 s** com 1 processo para **15.72 s** com 12 processos — uma redução de aproximadamente **81.7%**.

**O speedup obtido foi próximo do ideal?**
Parcialmente. O speedup cresceu de forma consistente, atingindo **5.46x** com 12 processos. Porém, o speedup ideal linear seria 12x, indicando que fatores como overhead de inicialização dos processos e contenção de I/O limitaram o ganho máximo.

**A aplicação apresentou escalabilidade?**
Sim. O tempo caiu a cada adição de processos, demonstrando boa escalabilidade até 12 processos — que é exatamente o número de núcleos lógicos disponíveis na máquina.

**Em qual ponto a eficiência começou a cair?**
A eficiência já apresenta queda gradual desde 2 processos (0.909), acentuando-se progressivamente até 12 processos (0.455). Isso é comportamento esperado em aplicações paralelas reais.

**O número de processos ultrapassa o número de núcleos físicos?**
Com 12 processos, o limite de núcleos lógicos da máquina é atingido exatamente. O ganho marginal entre 8 e 12 processos foi menor (de 17.19 s para 15.72 s), indicando saturação dos recursos de CPU.

**Houve overhead de paralelização?**
Sim, mas controlado. O overhead de criação dos processos e serialização dos dados via `pool.map` é responsável pela diferença entre o speedup obtido e o ideal.

**Por que multiprocessing funcionou e threading não?**
O `multiprocessing` cria processos independentes, cada um com seu próprio interpretador e **GIL (Global Interpreter Lock)** separado. Isso permite execução verdadeiramente simultânea em múltiplos núcleos. O `threading`, utilizado anteriormente, compartilha um único GIL, impedindo paralelismo real em tarefas CPU-intensivas.

**Outras causas de limitação do speedup:**

- overhead de criação e encerramento dos processos
- serialização e desserialização dos resultados entre processos
- contenção de acesso ao disco (I/O) com muitos processos simultâneos
- saturação dos núcleos físicos disponíveis

---

## 11. Conclusão

O uso de `multiprocessing.Pool` trouxe **ganho significativo e real de desempenho**, reduzindo o tempo de execução de **85.85 segundos para 15.72 segundos** com 12 processos — speedup de **5.46x**.

A migração de `threading` para `multiprocessing` foi determinante: processos independentes contornam o GIL do CPython e permitem paralelismo verdadeiro em tarefas CPU-intensivas.

A corretude foi mantida em todas as execuções, com resultados idênticos (10.000.000 linhas, 200.000.000 palavras, 1.366.663.305 caracteres).

O **melhor desempenho** foi obtido com **12 processos**, coincidindo com o número de núcleos lógicos da máquina.

**Melhorias futuras:**

- [ ] Realizar múltiplas execuções por configuração e calcular a média dos tempos
- [ ] Testar com `chunksize` no `pool.map` para reduzir overhead de comunicação entre processos
- [ ] Executar em ambiente Linux, onde `fork` reduz o custo de inicialização dos processos
- [ ] Remover ou reduzir o loop de simulação (`for _ in range(1000)`) para avaliar o comportamento real de I/O
