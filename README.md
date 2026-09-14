# 🩸 Rota Vital — Threads II

**Projeto Integrador — Processos e Threads**

Aplicação Spring Boot para demonstrar, na prática, o ganho de desempenho obtido com paralelismo em uma operação de alto volume do sistema **Rota Vital**.

</div>

---

# 🎯 Objetivo

O projeto simula uma operação do sistema **Rota Vital em escala nacional**, considerando centenas de milhares ou milhões de requisições hospitalares.

A operação escolhida foi a:

> **Validação em lote de requisições hospitalares.**

Cada requisição pode ser analisada de forma independente.

Isso permite dividir os dados em partes menores e processar essas partes simultaneamente.

O projeto compara três formas diferentes de realizar exatamente o mesmo processamento:

| Modo | Funcionamento |
|---|---|
| 🐢 **Sequencial** | Processa todas as requisições utilizando apenas um fluxo de execução. |
| ⚡ **Threads de plataforma** | Divide as requisições entre várias threads tradicionais. |
| 🪶 **Threads virtuais** | Divide as requisições entre threads virtuais disponíveis no Java 21. |

> ✅ Todas as versões devem produzir exatamente o mesmo resultado.

A diferença está no **tempo necessário para realizar o processamento**.

---

# 💡 Ideia escolhida

Cada requisição hospitalar possui informações como:

- 🆔 Identificador;
- 🏥 Hospital;
- 📦 Item solicitado;
- 🔢 Quantidade;
- 🚨 Prioridade;
- 🕐 Momento da criação;
- 🔐 Código de integridade.

Durante o processamento, o sistema verifica cada uma dessas informações.

Além disso, é realizado um cálculo utilizando **SHA-256**.

Esse cálculo aumenta o trabalho realizado pela CPU e torna possível observar melhor a diferença entre processamento sequencial e paralelo.

---

# 🧠 Como funciona

O fluxo básico é:

```text
📥 Requisição
     │
     ▼
🔎 Validar campos
     │
     ▼
🔐 Calcular SHA-256
     │
     ▼
❓ Código de integridade confere?
     │
   ┌─┴─┐
   │   │
  Sim Não
   │   │
   ▼   ▼
  ✅   ❌
Válida Inválida
   │   │
   └─┬─┘
     ▼
📊 Resultado final
```

Na versão paralela, o conjunto de requisições é dividido:

```text
               📚 Requisições

                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼

     🧵 Parte 1    🧵 Parte 2    🧵 Parte 3
        │            │            │
        └────────────┼────────────┘
                     │
                     ▼
              ➕ Agregação
                     │
                     ▼
              ✅ Resultado final
```

Cada thread processa uma parte diferente.

Depois, os resultados são reunidos.

---

# 🏗️ Estrutura do projeto

```text
RotaVital_ThreadsII_Portugues/
│
├── 📄 pom.xml
├── 📊 medicoes.csv
├── 📘 README.md
│
└── src/
    │
    ├── main/
    │   │
    │   ├── java/br/com/rotavital/
    │   │   │
    │   │   ├── 🚀 RotaVitalAplicacao.java
    │   │   │
    │   │   ├── controle/
    │   │   │   ├── 🌐 ControladorValidacao.java
    │   │   │   └── ⚠️ TratadorErros.java
    │   │   │
    │   │   ├── modelo/
    │   │   │   ├── 📦 Requisicao.java
    │   │   │   └── 📊 ResultadoValidacao.java
    │   │   │
    │   │   └── servico/
    │   │       ├── 🔐 CalculadoraIntegridade.java
    │   │       ├── 🏭 GeradorRequisicoes.java
    │   │       └── 🧵 ServicoValidacaoRequisicoes.java
    │   │
    │   └── resources/
    │       │
    │       ├── ⚙️ application.properties
    │       │
    │       └── static/
    │           └── 🖥️ index.html
    │
    └── test/java/br/com/rotavital/
        │
        ├── ✅ TesteConsistenciaResultados.java
        └── ⏱️ MedidorRotas.java
```

---

# 🧩 Explicação do código

## 🚀 RotaVitalAplicacao.java

Essa é a classe responsável por iniciar o sistema.

```java
@SpringBootApplication
public class RotaVitalAplicacao {

    public static void main(String[] argumentos) {
        SpringApplication.run(
            RotaVitalAplicacao.class,
            argumentos
        );
    }
}
```

### 📌 O que essa classe faz?

Ela:

- inicia o Spring Boot;
- cria o servidor web;
- identifica os serviços;
- identifica os controladores;
- disponibiliza a aplicação no navegador.

Quando o projeto iniciar corretamente, ele poderá ser acessado através de:

```text
http://localhost:8080
```

---

# 📦 Requisicao.java

Essa classe representa uma requisição hospitalar.

```java
public record Requisicao(
        long identificador,
        int hospital,
        int item,
        int quantidade,
        int prioridade,
        long instanteCriacao,
        long codigoIntegridade
) {
}
```

Foi utilizado um `record` porque o objeto serve principalmente para armazenar dados.

Cada requisição possui:

| Campo | Significado |
|---|---|
| `identificador` | Identificador da requisição |
| `hospital` | Hospital responsável |
| `item` | Item solicitado |
| `quantidade` | Quantidade solicitada |
| `prioridade` | Nível de prioridade |
| `instanteCriacao` | Momento em que foi criada |
| `codigoIntegridade` | Código utilizado na validação |

---

# 📊 ResultadoValidacao.java

Essa classe armazena o resultado final.

```java
public record ResultadoValidacao(
        int totalProcessado,
        int requisicoesValidas,
        int requisicoesInvalidas,
        int requisicoesUrgentes,
        long somaQuantidadesValidas,
        long assinaturaDoResultado
) {
}
```

Ela informa:

- 📦 quantidade total processada;
- ✅ quantidade válida;
- ❌ quantidade inválida;
- 🚨 quantidade urgente;
- ➕ soma das quantidades válidas;
- 🔐 assinatura final.

O mesmo resultado deve ser produzido pelas três versões.

---

# 🔐 CalculadoraIntegridade.java

Essa classe calcula o código de integridade das requisições.

Ela utiliza:

```java
MessageDigest
```

com o algoritmo:

```text
SHA-256
```

Exemplo:

```java
public static long calcular(
        MessageDigest calculadora,
        Requisicao requisicao,
        byte[] saida
) {

    calculadora.reset();

    adicionarLong(
        calculadora,
        requisicao.identificador()
    );

    adicionarInteiro(
        calculadora,
        requisicao.hospital()
    );

    adicionarInteiro(
        calculadora,
        requisicao.item()
    );

    adicionarInteiro(
        calculadora,
        requisicao.quantidade()
    );

    adicionarInteiro(
        calculadora,
        requisicao.prioridade()
    );

    adicionarLong(
        calculadora,
        requisicao.instanteCriacao()
    );

    // restante do cálculo
}
```

## 🧠 Por que utilizar SHA-256?

O SHA-256 exige processamento da CPU.

Como o cálculo é realizado várias vezes, o custo aumenta conforme o número de requisições cresce.

Por isso, essa operação é adequada para demonstrar uma tarefa:

```text
CPU-bound
```

Ou seja:

> grande parte do tempo é gasta realizando cálculos na CPU.

---

# 🏭 GeradorRequisicoes.java

Essa classe gera requisições fictícias para os testes.

Ela utiliza uma semente fixa:

```java
private static final long SEMENTE = 20260911L;
```

Isso significa que os dados gerados podem ser reproduzidos.

Assim, a versão sequencial e a versão paralela recebem os mesmos registros.

---

## 🧠 Memória de requisições

Existe também:

```java
private final ConcurrentHashMap<Integer, Requisicao[]>
        memoriaRequisicoes =
        new ConcurrentHashMap<>();
```

Essa estrutura guarda os dados já gerados.

Assim, os registros não precisam ser recriados a cada teste.

---

## ❌ Geração de registros inválidos

Uma pequena parte das requisições é alterada propositalmente.

Exemplo:

```java
if (indice % 100 == 0) {

    codigoIntegridade ^=
        0x5A5A5A5A5A5A5A5AL;
}
```

Isso faz com que aproximadamente:

```text
1%
```

dos registros apresentem erro de integridade.

Assim, o sistema consegue demonstrar a identificação de requisições inválidas.

---

# 🧵 ServicoValidacaoRequisicoes.java

Essa é a classe mais importante da atividade.

Ela contém as três formas de processamento.

---

# 🐢 1. Processamento sequencial

```java
public ResultadoValidacao validarSequencial(
        Requisicao[] requisicoes
) {

    ResultadoParcial parcial =
            processarTrecho(
                requisicoes,
                0,
                requisicoes.length
            );

    return parcial.converterParaResultado();
}
```

## 📌 O que acontece?

Uma única thread percorre todo o vetor.

Exemplo:

```text
Requisição 1
      ↓
Requisição 2
      ↓
Requisição 3
      ↓
Requisição 4
      ↓
Requisição 5
```

Uma requisição é processada após a outra.

---

# ⚡ 2. Threads de plataforma

```java
public ResultadoValidacao validarComThreadsDePlataforma(
        Requisicao[] requisicoes,
        int numeroThreads
) {

    validarQuantidadeDeTarefas(numeroThreads);

    try (
        ExecutorService executador =
            Executors.newFixedThreadPool(
                numeroThreads
            )
    ) {

        return processarEmParalelo(
            requisicoes,
            numeroThreads,
            executador
        );
    }
}
```

Aqui é utilizado:

```java
Executors.newFixedThreadPool(numeroThreads)
```

Isso cria um conjunto fixo de threads.

Exemplo com 4 threads:

```text
1.000.000 registros

        │
 ┌──────┼──────┬──────┐
 │      │      │      │
 ▼      ▼      ▼      ▼

T1     T2     T3     T4

250k   250k   250k   250k
```

Cada thread processa uma parte diferente.

---

# 🪶 3. Threads virtuais

```java
public ResultadoValidacao validarComThreadsVirtuais(
        Requisicao[] requisicoes,
        int numeroTarefas
) {

    validarQuantidadeDeTarefas(numeroTarefas);

    try (
        ExecutorService executador =
            Executors.newVirtualThreadPerTaskExecutor()
    ) {

        return processarEmParalelo(
            requisicoes,
            numeroTarefas,
            executador
        );
    }
}
```

A principal diferença está em:

```java
Executors.newVirtualThreadPerTaskExecutor()
```

Esse recurso está disponível no:

```text
Java 21
```

As threads virtuais são mais leves que threads tradicionais.

---

# ✂️ Divisão dos dados

O sistema calcula o tamanho de cada parte:

```java
int tamanhoTrecho =
        (totalRegistros
        + numeroTarefas
        - 1)
        / numeroTarefas;
```

Depois cada trecho é enviado para uma tarefa:

```java
tarefas.add(
    executador.submit(
        () -> processarTrecho(
            requisicoes,
            inicio,
            fim
        )
    )
);
```

---

# ⏳ Future

O sistema utiliza:

```java
Future<ResultadoParcial>
```

O `Future` representa um resultado que será disponibilizado quando a tarefa terminar.

Depois o programa recupera os resultados:

```java
for (
    Future<ResultadoParcial> tarefa :
    tarefas
) {

    resultadoFinal.somar(
        tarefa.get()
    );
}
```

---

# 🔎 processarTrecho()

Esse método realiza a validação.

Primeiramente verifica os campos:

```java
boolean camposValidos =
        requisicao.hospital() >= 1
        && requisicao.hospital() <= 5_000

        && requisicao.item() >= 1
        && requisicao.item() <= 20_000

        && requisicao.quantidade() >= 1
        && requisicao.quantidade() <= 500

        && requisicao.prioridade() >= 1
        && requisicao.prioridade() <= 5;
```

Depois verifica a integridade:

```java
boolean integridadeValida =
        codigoCalculado
        ==
        requisicao.codigoIntegridade();
```

Por fim:

```java
boolean requisicaoValida =
        camposValidos
        &&
        integridadeValida;
```

Ou seja:

```text
Campos corretos
      +
Integridade correta
      =
Requisição válida
```

---

# 🌐 ControladorValidacao.java

Essa classe disponibiliza os endpoints da API.

Exemplo:

```java
@GetMapping("/sequencial")
public ResponseEntity<ResultadoValidacao>
validarSequencial(

        @RequestParam(
            defaultValue = "100000"
        )
        int quantidade
) {

    // processamento
}
```

---

## ⏱️ Medição do tempo

Antes do processamento:

```java
long inicio = System.nanoTime();
```

Depois:

```java
long tempoEmNanossegundos =
        System.nanoTime() - inicio;
```

O tempo é enviado no cabeçalho HTTP:

```text
X-Tempo-Processamento-Ms
```

A interface utiliza esse valor para mostrar quanto tempo cada execução levou.

---

# ⚠️ TratadorErros.java

Essa classe trata erros enviados pelo usuário.

Exemplo:

```java
@ExceptionHandler(
    IllegalArgumentException.class
)
public ResponseEntity<Map<String, String>>
tratarArgumentoInvalido(
        IllegalArgumentException excecao
) {

    return ResponseEntity
            .status(
                HttpStatus.BAD_REQUEST
            )
            .body(
                Map.of(
                    "erro",
                    excecao.getMessage()
                )
            );
}
```

Assim, em vez de o programa simplesmente quebrar, ele devolve uma mensagem compreensível.

---

# 🖥️ index.html

É a página visual do projeto.

Ela permite configurar:

- 📦 quantidade de requisições;
- 🧵 quantidade de threads;
- 🪶 quantidade de tarefas virtuais.

A interface possui os botões:

```text
▶ Testar sequencial

⚡ Testar threads de plataforma

🪶 Testar threads virtuais

📊 Comparar as três versões
```

---

## 🌐 Comunicação da interface com a API

O JavaScript utiliza:

```javascript
fetch()
```

Exemplo:

```javascript
const resposta = await fetch(
    montarEndereco(
        modo,
        configuracao
    )
);
```

O resultado é exibido diretamente na tela.

---

# ✅ TesteConsistenciaResultados.java

Esse teste garante que todas as versões produzam a mesma resposta.

```java
assertEquals(
    sequencial,
    plataforma
);

assertEquals(
    sequencial,
    virtuais
);
```

O resultado esperado é:

```text
Sequencial
    =
Threads de plataforma
    =
Threads virtuais
```

Isso é importante porque:

> não adianta uma versão ser mais rápida se ela produzir um resultado incorreto.

---

# ⏱️ MedidorRotas.java

Essa classe mede o desempenho dos endpoints.

O programa realiza:

1. 🔥 aquecimento da aplicação;
2. 🧪 múltiplas execuções;
3. ⏱️ coleta dos tempos;
4. 📊 ordenação dos resultados;
5. 📐 cálculo da mediana;
6. ✅ comparação das respostas;
7. 🚀 cálculo de speedup.

---

# 🚀 Speedup

A fórmula utilizada é:

```text
Speedup =
Tempo sequencial
────────────────
Tempo paralelo
```

Exemplo:

```text
304,817 ms
──────────
90,734 ms
```

Resultado:

```text
≈ 3,359x
```

Isso significa que a versão paralela foi aproximadamente:

```text
3,36 vezes mais rápida
```

---

# 🧵 Tipos de processamento

| Característica | 🐢 Sequencial | ⚡ Plataforma | 🪶 Virtual |
|---|:---:|:---:|:---:|
| Um único fluxo | ✅ | ❌ | ❌ |
| Divide os dados | ❌ | ✅ | ✅ |
| Usa ExecutorService | ❌ | ✅ | ✅ |
| Threads tradicionais | ❌ | ✅ | ❌ |
| Threads virtuais | ❌ | ❌ | ✅ |
| Mesmo resultado | ✅ | ✅ | ✅ |

---

# 🔒 Condição de corrida

Um problema comum em programas com várias threads é a:

```text
Race Condition
```

ou:

```text
Condição de corrida
```

Ela pode acontecer quando várias threads modificam a mesma variável ao mesmo tempo.

Exemplo problemático:

```java
contador++;
```

Se várias threads fizerem isso simultaneamente, alguns incrementos podem ser perdidos.

---

## ✅ Como o projeto evita isso?

Cada tarefa possui seu próprio resultado:

```java
ResultadoParcial resultado =
        new ResultadoParcial();
```

Cada thread altera apenas o próprio objeto.

Depois que todas terminam, ocorre a soma:

```java
for (
    Future<ResultadoParcial> tarefa :
    tarefas
) {

    resultadoFinal.somar(
        tarefa.get()
    );
}
```

Fluxo:

```text
Thread 1 ──► Resultado 1 ─┐
                          │
Thread 2 ──► Resultado 2 ─┼──► Resultado final
                          │
Thread 3 ──► Resultado 3 ─┘
```

Assim, as threads não precisam alterar simultaneamente um mesmo contador global.

---

# 🌐 Interface e rotas

## 🏠 Página principal

```text
http://localhost:8080
```

---

## 🐢 Sequencial

```text
http://localhost:8080/api/validacoes/sequencial?quantidade=100000
```

---

## ⚡ Threads de plataforma

```text
http://localhost:8080/api/validacoes/paralelo?quantidade=100000&numeroThreads=4
```

---

## 🪶 Threads virtuais

```text
http://localhost:8080/api/validacoes/virtuais?quantidade=100000&numeroTarefas=4
```

---

# 📤 Exemplo de resposta

```json
{
  "totalProcessado": 100000,
  "requisicoesValidas": 99000,
  "requisicoesInvalidas": 1000,
  "requisicoesUrgentes": 39599,
  "somaQuantidadesValidas": 24823490,
  "assinaturaDoResultado": 123456789
}
```

> ⚠️ Os valores acima servem apenas como exemplo do formato da resposta.

---

# ▶️ Como executar

## 1️⃣ Requisitos

É necessário ter instalado:

- ☕ Java 21 ou superior;
- 📦 Maven;
- 💻 VS Code, IntelliJ ou outro editor.

Verifique o Java:

```cmd
java -version
```

Verifique o Maven:

```cmd
mvn -version
```

---

## 2️⃣ Abrir a pasta

Abra a pasta que contém:

```text
pom.xml
```

Exemplo:

```cmd
cd RotaVital_ThreadsII_Portugues
```

---

## 3️⃣ Executar

Digite:

```cmd
mvn spring-boot:run
```

Quando aparecer:

```text
Started RotaVitalAplicacao
```

abra:

```text
http://localhost:8080
```

---

## 4️⃣ Encerrar

Para parar o servidor:

```text
Ctrl + C
```

---

# 🚧 Porta 8080 ocupada

Verifique:

```cmd
netstat -ano | findstr :8080
```

O Windows mostrará o PID do processo utilizando a porta.

---

# 🧪 Testes

Para executar os testes automáticos:

```cmd
mvn test
```

O objetivo é confirmar que:

```text
Sequencial
=
Plataforma
=
Virtual
```

---

# 📊 Medições

Resultados registrados:

| Quantidade | Modo | Threads | Tempo | Speedup |
|---:|---|---:|---:|---:|
| 100.000 | 🐢 Sequencial | 1 | 31,193 ms | 1,000x |
| 100.000 | ⚡ Plataforma | 2 | 17,275 ms | 1,806x |
| 100.000 | ⚡ Plataforma | 4 | 8,893 ms | **3,507x** |
| 100.000 | ⚡ Plataforma | 8 | 12,116 ms | 2,574x |
| 100.000 | ⚡ Plataforma | 16 | 15,260 ms | 2,044x |
| 100.000 | 🪶 Virtual | 8 | 9,983 ms | 3,125x |
| 1.000.000 | 🐢 Sequencial | 1 | 304,817 ms | 1,000x |
| 1.000.000 | ⚡ Plataforma | 2 | 166,003 ms | 1,836x |
| 1.000.000 | ⚡ Plataforma | 4 | 90,734 ms | 3,359x |
| 1.000.000 | ⚡ Plataforma | 8 | 93,209 ms | 3,270x |
| 1.000.000 | ⚡ Plataforma | 16 | 81,884 ms | 3,723x |
| 1.000.000 | 🪶 Virtual | 8 | 79,833 ms | **3,818x** |

---

# 🔍 Análise dos resultados

Os resultados mostram que:

- ⚡ o paralelismo trouxe ganho real;
- 📈 com 100 mil registros, 4 threads apresentaram bom desempenho;
- 🚀 com 1 milhão de registros, o ganho ficou ainda mais evidente;
- 🧵 mais threads não significam necessariamente maior desempenho;
- 🖥️ a quantidade de núcleos da CPU limita o paralelismo;
- ⏱️ criar, controlar e sincronizar tarefas também possui custo.

---

# 📈 Big-O e Speedup

## Big-O

O processamento sequencial percorre todos os registros.

Portanto:

```text
O(n)
```

Mesmo utilizando várias threads, o trabalho total continua sendo:

```text
O(n)
```

A complexidade não muda.

O que muda é o tempo real necessário para executar o trabalho.

Com `p` tarefas, uma aproximação ideal seria:

```text
O(n / p)
```

Porém existem custos adicionais.

---

# 🆚 Concorrência x Paralelismo

## 🔄 Concorrência

Várias tarefas progridem durante o mesmo intervalo de tempo.

Elas não precisam necessariamente estar sendo executadas exatamente ao mesmo tempo.

---

## ⚡ Paralelismo

Duas ou mais tarefas executam realmente ao mesmo tempo em diferentes núcleos da CPU.

Neste projeto, o objetivo principal é:

> utilizar paralelismo para acelerar o processamento de um grande volume de requisições.

---

# 🧰 Tecnologias

| Tecnologia | Uso |
|---|---|
| ☕ **Java 21** | Linguagem principal |
| 🌱 **Spring Boot** | API e servidor |
| 📦 **Maven** | Dependências e execução |
| 🧵 **ExecutorService** | Controle das tarefas |
| 🔐 **SHA-256** | Processamento de integridade |
| 🧪 **JUnit 5** | Testes |
| 🌐 **HTML** | Estrutura da interface |
| 🎨 **CSS** | Estilo da interface |
| ⚙️ **JavaScript** | Comunicação com a API |
| 📡 **HttpClient** | Medição dos endpoints |

---

# 👥 Integrantes

<table>

<tr>
<td>👨‍💻</td>
<td><strong>Ewerton Guilherme da Silva</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>Pablo Arthur Eustáquio de Lima</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>Saulo Eduardo Almeida dos Santos</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>Lucas Aprigio dos Santos</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>João Ricardo Alves de Brito</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>Thiago Cardozo da Conceição</strong></td>
</tr>

<tr>
<td>👨‍💻</td>
<td><strong>Eloi de Lima Sousa</strong></td>
</tr>

</table>

---

# 📝 Palavras que permanecem em inglês

Algumas palavras não podem ser traduzidas porque fazem parte da linguagem Java, do Spring ou das bibliotecas.

Exemplos:

```text
public
class
record
return
String
MessageDigest
ExecutorService
Future
HttpClient
RestController
GetMapping
```

Também utilizamos o termo:

```text
Thread
```

porque é o conceito técnico principal da atividade.

---

<div align="center">

# 🩸 Rota Vital

### Processamento sequencial e paralelo aplicado a grandes volumes de dados

💻 Projeto Integrador — Threads II

</div>
