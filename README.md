<div align="center">

# 🩸 HemoTrack — Threads II

**Projeto Integrador — Processos e Threads**

Aplicação em **Java 21 + Spring Boot** para comparar processamento sequencial, threads de plataforma e threads virtuais em uma operação CPU-bound com grande volume de requisições hospitalares.

</div>

---

## 🎯 Objetivo

O projeto avalia a **validação em lote de requisições hospitalares**. Cada registro tem seus campos validados e um código de integridade **SHA-256** recalculado.

A mesma operação é executada de três formas:

| Modo | Estratégia |
|---|---|
| 🐢 **Sequencial** | Um único fluxo processa todos os registros |
| ⚡ **Threads de plataforma** | O vetor é dividido entre threads de um `ExecutorService` |
| 🪶 **Threads virtuais** | As fatias são executadas com virtual threads do Java 21 |

> Todas as versões devem produzir exatamente o mesmo resultado. O que muda é o tempo de processamento.

---

## 🧠 Por que essa operação?

A validação foi escolhida porque o gargalo está principalmente na **CPU**, e não em espera de banco ou rede. O cálculo SHA-256 é repetido para cada requisição, tornando o custo mensurável em entradas de **100 mil** e **1 milhão** de registros.

Os dados são particionáveis porque cada requisição pode ser validada de forma independente. Cada tarefa mantém seu próprio resultado parcial e, ao final, os resultados são agregados. Isso reduz o risco de **race condition**.

---

## ⚙️ Fluxo do projeto

```text
Interface Web
     ↓
API Spring Boot
     ↓
Geração/recuperação das requisições
     ↓
Validação dos campos + SHA-256
     ↓
Sequencial / Plataforma / Virtual
     ↓
Agregação dos resultados
     ↓
Tempo + Speedup + Gráfico na interface
```

A interface permite escolher a quantidade de registros e de threads/tarefas, executar cada versão separadamente ou comparar as três.

---

## 📈 Big-O

Se `n` é o número de requisições:

```text
Sequencial: O(n)
```

Cada registro é visitado uma vez e o trabalho por registro é constante em relação a `n`.

Com `p` trabalhadores, a parte paralelizável pode se aproximar de:

```text
O(n / p) + overhead
```

O **trabalho total continua O(n)**, pois todos os registros ainda precisam ser processados. O paralelismo reduz o tempo de parede, mas não altera a ordem de crescimento assintótico quando `p` é limitado pelo hardware.

---

## 📊 Medições

O speedup é calculado por:

```text
Speedup = T_sequencial / T_versão
```

| Entrada | Configuração | Tempo | Speedup |
|---:|---|---:|---:|
| 100.000 | Sequencial — 1 | 31,193 ms | 1,000x |
| 100.000 | Plataforma — 2 | 17,275 ms | 1,806x |
| 100.000 | Plataforma — 4 | **8,893 ms** | **3,507x** |
| 100.000 | Plataforma — 8 | 12,116 ms | 2,574x |
| 100.000 | Plataforma — 16 | 15,260 ms | 2,044x |
| 100.000 | Virtual — 8 | 9,983 ms | 3,125x |
| 1.000.000 | Sequencial — 1 | 304,817 ms | 1,000x |
| 1.000.000 | Plataforma — 2 | 166,003 ms | 1,836x |
| 1.000.000 | Plataforma — 4 | 90,734 ms | 3,359x |
| 1.000.000 | Plataforma — 8 | 93,209 ms | 3,270x |
| 1.000.000 | Plataforma — 16 | 81,884 ms | 3,723x |
| 1.000.000 | Virtual — 8 | **79,833 ms** | **3,818x** |

### Tempo de resposta

![Tempo de resposta](grafico_tempo_resposta(2).png)

### Speedup observado

![Speedup](grafico_speedup(2).png)

---

## 🔍 Resumo da análise

- O ganho **não foi linear**: mais threads também geram custos de agendamento, sincronização, cache e agregação.
- Em **100 mil registros**, 4 threads de plataforma tiveram o melhor resultado registrado: **8,893 ms e 3,507x**.
- Em **1 milhão de registros**, a carga maior aproveitou melhor o paralelismo; 8 tarefas virtuais chegaram a **79,833 ms e 3,818x**.
- A Big-O permanece **O(n)**; o paralelismo melhora o tempo real, não a quantidade total de trabalho.
- **Concorrência** organiza várias tarefas em progresso; **paralelismo** executa partes do mesmo trabalho simultaneamente em múltiplos núcleos.
- Threads virtuais são um diferencial do Java 21, mas em carga CPU-bound não garantem automaticamente desempenho superior.
- Se uma única JVM deixar de ser suficiente, a evolução natural é distribuir lotes entre **filas, workers e múltiplas instâncias**.

---

## 🌐 Endpoints principais

```text
GET /api/validacoes/sequencial?quantidade=100000

GET /api/validacoes/paralelo?quantidade=100000&numeroThreads=4

GET /api/validacoes/virtuais?quantidade=100000&numeroTarefas=8
```

A aplicação também possui interface web em:

```text
http://localhost:8080
```

---

## ▶️ Como executar

### Requisitos

- Java 21+
- Maven

### Executar

```bash
mvn spring-boot:run
```

Depois acesse:

```text
http://localhost:8080
```

### Testes

```bash
mvn test
```

O teste de consistência verifica se:

```text
Sequencial = Plataforma = Virtual
```

---

## 🧰 Tecnologias

`Java 21` · `Spring Boot` · `Maven` · `ExecutorService` · `Virtual Threads` · `SHA-256` · `JUnit 5` · `HTML` · `CSS` · `JavaScript` · `HttpClient`

---

## 👥 Equipe

- **Ewerton Guilherme da Silva**
- **Pablo Arthur Eustáquio de Lima**
- **Saulo Eduardo Almeida dos Santos**
- **Lucas Aprigio dos Santos**
- **João Ricardo Alves de Brito**
- **Thiago Cardozo da Conceição**
- **Eloi de Lima Sousa**

---

<div align="center">

### 🩸 HemoTrack

**Processamento sequencial e paralelo aplicado a grandes volumes de dados**

</div>
