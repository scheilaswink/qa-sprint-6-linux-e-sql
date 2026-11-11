# Linux e SQL — análise de logs e consultas em banco de dados

Projeto da Sprint 6 do curso de QA Engineer da TripleTen: uso do **terminal Linux** para analisar logs de um servidor web e de **SQL (PostgreSQL)** para consultar um banco de dados de corridas de táxi de Chicago.

## Parte 1 — Terminal Linux (logs do servidor Apache)

**Tarefa 1.** Buscar nos logs de dezembro de 2019 as requisições vindas de IPs que começam com `233.201`, usando `cd` e `grep -R` com expressão regular. Foram encontradas 2 requisições (dias 18 e 21).

**Tarefa 2.** Separar os erros de um dia de log por código de status:

```bash
mkdir bug1
mkdir bug1/events

grep '[45]00' ~/logs/2019/12/apache_2019-12-30.txt > ~/bug1/main.txt

grep '400' ~/bug1/main.txt > ~/bug1/events/400.txt
grep '500' ~/bug1/main.txt > ~/bug1/events/500.txt
```

Os resultados foram conferidos com `wc`, `head -3` e `tail -3`: `500.txt` ficou com 160 linhas e `400.txt` ficou vazio.

## Parte 2 — Banco de dados (PostgreSQL)

**Tarefa 1. Número de carros:** 5529.

```sql
SELECT count(*) AS COUNT from cabs;
```

**Tarefa 2. Empresas com menos de 100 carros** (51 empresas):

```sql
select company_name, COUNT(*) AS cnt FROM cabs
GROUP BY company_name
HAVING count(*)<100
ORDER BY COUNT(*) DESC;
```

**Tarefa 3. Condição do tempo a cada hora em 05/11/2017** (`GOOD` ou `BAD`):

```sql
SELECT CASE WHEN description LIKE '%rain%' THEN 'BAD'
       WHEN description LIKE '%STORM%' THEN 'BAD'
       ELSE 'GOOD'
       END AS weather_conditions, TS
FROM weather_records
WHERE weather_records.TS BETWEEN '2017-11-05 00:00:00' AND '2017-11-05 23:59:59';
```

**Tarefa 4. Corridas por empresa em 15 e 16/11/2017** (64 empresas, de 19.558 corridas da Flash Cab até 2 da RC Andrews Cab):

```sql
select cabs.company_name, COUNT(*) AS trips_amount FROM cabs
INNER JOIN trips on trips.cab_id=cabs.cab_id
where trips.start_ts BETWEEN '2017-11-15 00:00:00' AND '2017-11-16 23:59:59'
group by cabs.company_name
ORDER BY COUNT(*) DESC;
```

Todos os resultados completos estão no PDF do projeto.

## Melhorias apontadas na revisão

O projeto foi aprovado. O revisor sugeriu estas melhorias. Elas mudam os resultados, e as versões abaixo **ainda não foram executadas** no ambiente do curso.

**1. Filtrar o código de status com contexto no `grep`.** Buscar só `'500'` também pega linhas em que 500 aparece no tamanho da resposta ou no IP (falsos positivos). Por isso o arquivo ficou com 160 linhas em vez de 156.

```bash
grep ' HTTP/1.1" 400 ' ~/bug1/main.txt > ~/bug1/events/400.txt
grep ' HTTP/1.1" 500 ' ~/bug1/main.txt > ~/bug1/events/500.txt
```

**2. Contar carros únicos com `COUNT(DISTINCT ...)`.** `COUNT(*)` conta todas as linhas, inclusive duplicadas.

```sql
SELECT COUNT(DISTINCT vehicle_id) AS cnt FROM cabs;
```

**3. Cuidado com maiúsculas no `LIKE`.** No PostgreSQL, `LIKE` diferencia maiúsculas de minúsculas, então `'%STORM%'` não encontra "storm".

```sql
WHEN description LIKE '%rain%' OR description LIKE '%storm%' THEN 'BAD'
```

## Arquivos

- [`projeto-sprint-6-linux-e-sql.pdf`](projeto-sprint-6-linux-e-sql.pdf): o projeto completo, com todos os comandos, consultas e resultados.

## O que aprendi

- **Linha de comando:** navegar por pastas, criar diretórios e filtrar logs com `grep`, `wc`, `head` e `tail`.
- **SQL:** consultas com `COUNT`, `GROUP BY`, `HAVING`, `CASE`, `BETWEEN` e `INNER JOIN`.
- **PostgreSQL:** consultar um banco de dados real com várias tabelas relacionadas.
- **Precisão nos filtros:** um filtro genérico pode gerar falsos positivos, e um bom teste precisa conferir se o resultado faz sentido.
