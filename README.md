# 🩸 HemoTrack | US07 Demanda Preditiva com Threads

Atividade de **Infraestrutura de Software** (CESAR School, ADS 2026.2).

## A atividade
Encontrar uma operação da camada de aplicação do HemoTrack que fica lenta em uma única thread na escala nacional. Depois, implementá-la como serviço real em Java com Spring Boot, criar uma versão sequencial e outra com threads (2, 4 e 8), medir o ganho (speedup) e explicar os resultados.

## O que escolhemos
**US07 – Análise Preditiva de Demanda por Tipo Sanguíneo.** O sistema percorre o histórico de requisições e calcula, para cada tipo sanguíneo:
- a média (μ) e o desvio (σ) do consumo semanal;
- a tendência e a projeção do próximo trimestre;
- a probabilidade de faltar estoque (ex.: O−).

É uma operação O(n), limitada por CPU e fácil de dividir em fatias independentes, por isso ganha com paralelismo.

## Como funciona
- **Sequencial:** uma thread processa todo o histórico.
- **Com threads:** o histórico é dividido em fatias; cada thread (`ExecutorService`) soma a sua fatia em um acumulador próprio, e os parciais são somados no final.
- **Threads virtuais (Java 21):** a mesma lógica, para comparação.
- Todas as versões devolvem **exatamente o mesmo resultado**, comprovado por assinatura SHA-256 e testes.
- Há também uma versão sem proteção, feita de propósito para mostrar a race condition.

Resultado com 1 milhão de registros em 16 núcleos: **90,1 ms → 33,6 ms com 4 threads (2,68×)**.

## Como rodar
Requisitos: Java 21 e Maven.

    mvn spring-boot:run

Depois, abra **http://localhost:8080** para gerar a base, comparar as versões e rodar as medições.

## Equipe
- Ewerton Guilherme da Silva
- Pablo Arthur Eustáquio de Lima
- Saulo Eduardo Almeida dos Santos
- Lucas Aprigio dos Santos
- João Ricardo Alves de Brito
- Thiago Cardozo da Conceição
- Eloi de Lima Sousa
