# UFBA-BSI-LabBD-Proj01


# Projeto 1 - Banco de Dados de alto desempenho

## Entrega 01 - Construção de base de consultas

### CCB - Consultas de complexidade baixa

#### C01
```sql
-- qual o percentual de pessoas cadastradas que trabalham por conta própria? (1 - Trabalhador por conta própria (bico, autônomo))
SELECT COUNT(*) || ' de ' || (SELECT COUNT(*) FROM tb_trab) AS percentual FROM tb_trab
WHERE cod_principal_trab_memb = 1;
```
#### C02
```sql
-- Quais famílias tiveram seus dados alterados no ano de 2015?
SELECT * FROM tb_familia
WHERE dat_alteracao_fam
BETWEEN '2015-01-01' AND '2015-12-31'
ORDER BY dat_alteracao_fam;
```
#### C03
```sql
-- quais são as 100 pessoas mais idosas cadastradas?
SELECT id_familia, id_pessoa, idade
FROM tb_pessoa
ORDER BY idade DESC LIMIT 100;
```
#### C04
```sql
-- qual o valor da renda média das famílias na cidade de Salvador?
SELECT vlr_renda_media_fam
FROM tb_familia f
JOIN tb_mun m ON f.cd_ibge = m.cd_ibge
WHERE m.nome_municipio = 'Salvador';
```
#### C05
```sql
-- Quais são as famílias que possuem menos dormitorios do que a quantidade de membros?
SELECT f.id_familia, f.qtde_pessoas AS pessoas, d.qtd_comodos_dormitorio_fam AS dormitorios
FROM tb_domicilio d
JOIN tb_familia f ON f.id_familia = d.id_familia
WHERE d.qtd_comodos_dormitorio_fam < f.qtde_pessoas;
```
### CCI - Consultas de complexidade intermediária

#### C06
```sql
-- qual é a média da renda média de cada família por estado?
SELECT m.uf, ROUND(AVG(replace(vlr_renda_media_fam,',','.')::DECIMAL), 2)
    FROM tb_familia f
    JOIN tb_mun m ON f.cd_ibge = m.cd_ibge
	GROUP BY m.uf;
```
#### C07
```sql
-- qual o percentual de famílias sem água canalizada? (COD_AGUA_CANALIZADA_FAM = 2 (Não possui))
SELECT ROUND((COUNT(*) FILTER (WHERE cod_agua_canalizada_fam = 2)::numeric / COUNT(*)) * 100, 2) || '%' AS percent_sem_agua_canalizada
FROM tb_domicilio;
```
#### C08
```sql
-- Qual é o top 100 de municípios com mais famílias cadastradas?
SELECT tb_mun.nome_municipio, tb_mun.uf , COUNT(tb_familia.id_familia) AS qtd_familias
FROM tb_familia
JOIN tb_mun ON tb_familia.cd_ibge = tb_mun.cd_ibge
GROUP BY tb_mun.nome_municipio, tb_mun.uf
ORDER BY qtd_familias DESC LIMIT 100;
```
#### C09
```sql
-- Qual é o top 10 de estados com mais famílias cadastradas?
SELECT tb_mun.uf, COUNT(tb_familia.id_familia) AS qtd_familias
FROM tb_familia
JOIN tb_mun ON tb_familia.cd_ibge = tb_mun.cd_ibge
GROUP BY tb_mun.uf
ORDER BY qtd_familias DESC LIMIT 10;
```
#### C10
```sql
-- Quais são as pessoas que trabalharam 3 meses ou menos nos últimos 12 meses, quantos meses elas trabalharam e quais são as cidades em que moram? (qtd_meses_12_meses_memb < 3)
SELECT t.id_familia, t.id_pessoa, m.nome_municipio, t.qtd_meses_12_meses_memb
FROM tb_trab t
JOIN tb_familia f ON f.id_familia = t.id_familia
JOIN tb_mun m ON m.cd_ibge = f.cd_ibge
WHERE qtd_meses_12_meses_memb <= 3
ORDER BY t.id_familia, t.id_pessoa;
```

### CCA - Consultas de complexidade alta

#### C11
```sql
-- Considerando a soma da renda de cada integrante de cada família, e excluindo valores nulos, qual é o valor total da renda bruta em 12 meses em ordem decrescente?
SELECT * FROM (
	SELECT tb_trab.id_familia, SUM(tb_trab.val_renda_bruta_12_meses_memb) AS renda_bruta_12m_consolidada
FROM tb_trab
GROUP BY tb_trab.id_familia
ORDER BY renda_bruta_12m_consolidada DESC)
WHERE renda_bruta_12m_consolidada IS NOT NULL;
```
#### C12
```sql
-- Quais são as cidades com o maior número de familias abaixo da linha da pobreza?
-- Cálculo: US$ 2.15 por dia, por pessoa. 2.15 * 30 = 64.5 USD ~= 375 BRL (11/2024).

SELECT tb_mun.nome_municipio, COUNT(tb_familia.id_familia) AS qtd_domicilios
    FROM tb_familia
    JOIN tb_mun ON tb_familia.cd_ibge = tb_mun.cd_ibge
    WHERE (CAST(REPLACE(tb_familia.vlr_renda_media_fam, ',', '.') AS NUMERIC)) / NULLIF(tb_familia.qtde_pessoas,0) < 375
    GROUP BY tb_mun.nome_municipio
    ORDER BY qtd_domicilios DESC;
```
#### C13
```sql
-- Qual a quantidade de pessoas em cada município que nunca frequentou a escola? ind_frequenta_escola_membro = 4 (nunca frequentou escola)
SELECT DISTINCT COUNT(p.id_pessoa) AS numero_pessoas, m.nome_municipio
FROM tb_familia f
JOIN tb_mun m ON m.cd_ibge = f.cd_ibge
JOIN tb_pessoa p ON f.id_familia = p.id_familia
JOIN tb_esc e ON e.id_familia = f.id_familia
WHERE e.ind_frequenta_escola_memb = 4
GROUP BY m.nome_municipio
ORDER BY numero_pessoas DESC;
```
#### C14
```sql
-- Quantos adultos, idosos e crianças têm em cada família?
SELECT id_familia,
COUNT(CASE WHEN tb_pessoa.idade >= 18 AND tb_pessoa.idade < 60 THEN 1 END) AS qtd_adultos, 
COUNT(CASE WHEN tb_pessoa.idade >= 60 THEN 1 END) AS qtd_idosos,
COUNT(CASE WHEN tb_pessoa.idade < 18 THEN 1 END) AS qtd_criancas
FROM tb_pessoa
GROUP BY id_familia
ORDER BY qtd_adultos DESC, qtd_idosos DESC, qtd_criancas DESC;
```
#### C15
```sql
-- Quais são as pessoas do estado da Bahia que não possuem banheiro em casa, e as cidades em que moram? (cod_banheiro_domic_fam = 2)
SELECT p.id_pessoa, d.id_familia, m.nome_municipio
FROM tb_domicilio d
JOIN tb_pessoa p ON p.id_familia = d.id_familia
JOIN tb_familia f ON f.id_familia = p.id_familia
JOIN tb_mun m ON m.cd_ibge = f.cd_ibge
WHERE d.cod_banheiro_domic_fam = 2 AND m.uf = 'BA'
ORDER BY m.nome_municipio ASC;
```

### Avaliação das consultas


| Cód. da Consulta | Nível de Corretude                                    |
| ---------------- | ----------------------------------------------------- |
| C01              | I - Correta, coerente com a descrição e complexidade. |
| C02              | I - Correta, coerente com a descrição e complexidade. |
| C03              | I - Correta, coerente com a descrição e complexidade. |
| C04              | I - Correta, coerente com a descrição e complexidade. |
| C05              | I - Correta, coerente com a descrição e complexidade. |
| C06              | I - Correta, coerente com a descrição e complexidade. |
| C07              | I - Correta, coerente com a descrição e complexidade. |
| C08              | I - Correta, coerente com a descrição e complexidade. |
| C09              | I - Correta, coerente com a descrição e complexidade. |
| C10              | I - Correta, coerente com a descrição e complexidade. |
| C11              | I - Correta, coerente com a descrição e complexidade. |
| C12              | I - Correta, coerente com a descrição e complexidade. |
| C13              | I - Correta, coerente com a descrição e complexidade. |
| C14              | I - Correta, coerente com a descrição e complexidade. |
| C15              | I - Correta, coerente com a descrição e complexidade. |
#### Desempenho Original

Concatenando a instrução SQL **EXPLAIN ANALYSE** antes de cada consulta, foram obtidas as seguintes estatísticas:
##### C01

```
"Finalize Aggregate  (cost=23479.99..23480.01 rows=1 width=32) (actual time=184.260..188.987 rows=1 loops=1)"
"  InitPlan 1"
"    ->  Finalize Aggregate  (cost=11693.56..11693.57 rows=1 width=8) (actual time=47.255..51.921 rows=1 loops=1)"
"          ->  Gather  (cost=11693.34..11693.56 rows=2 width=8) (actual time=47.038..51.912 rows=3 loops=1)"
"                Workers Planned: 2"
"                Workers Launched: 2"
"                ->  Partial Aggregate  (cost=10693.34..10693.35 rows=1 width=8) (actual time=22.586..22.586 rows=1 loops=3)"
"                      ->  Parallel Seq Scan on tb_trab tb_trab_1  (cost=0.00..9978.67 rows=285868 width=0) (actual time=0.229..15.111 rows=228694 loops=3)"
"  ->  Gather  (cost=11786.20..11786.41 rows=2 width=8) (actual time=65.091..137.022 rows=3 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Partial Aggregate  (cost=10786.20..10786.21 rows=1 width=8) (actual time=28.964..28.965 rows=1 loops=3)"
"              ->  Parallel Seq Scan on tb_trab  (cost=0.00..10693.34 rows=37144 width=0) (actual time=0.198..27.734 rows=29435 loops=3)"
"                    Filter: (cod_principal_trab_memb = 1)"
"                    Rows Removed by Filter: 199259"
"Planning Time: 0.140 ms"
"Execution Time: 189.027 ms"
```

##### C02

```
"Sort  (cost=7059.79..7103.05 rows=17305 width=32) (actual time=16.093..18.028 rows=16906 loops=1)"
"  Sort Key: dat_alteracao_fam"
"  Sort Method: quicksort  Memory: 1672kB"
"  ->  Seq Scan on tb_familia  (cost=0.00..5841.61 rows=17305 width=32) (actual time=0.011..14.150 rows=16906 loops=1)"
"        Filter: ((dat_alteracao_fam >= '2015-01-01'::date) AND (dat_alteracao_fam <= '2015-12-31'::date))"
"        Rows Removed by Filter: 231735"
"Planning Time: 0.064 ms"
"Execution Time: 18.578 ms"
```

##### C03

```
"Limit  (cost=21128.37..21140.04 rows=100 width=25) (actual time=171.342..175.026 rows=100 loops=1)"
"  ->  Gather Merge  (cost=21128.37..87835.56 rows=571736 width=25) (actual time=171.340..175.021 rows=100 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        ->  Sort  (cost=20128.34..20843.01 rows=285868 width=25) (actual time=112.061..112.065 rows=88 loops=3)"
"              Sort Key: idade DESC"
"              Sort Method: top-N heapsort  Memory: 36kB"
"              Worker 0:  Sort Method: top-N heapsort  Memory: 37kB"
"              Worker 1:  Sort Method: top-N heapsort  Memory: 37kB"
"              ->  Parallel Seq Scan on tb_pessoa  (cost=0.00..9202.67 rows=285868 width=25) (actual time=0.482..90.044 rows=228694 loops=3)"
"Planning Time: 10.383 ms"
"Execution Time: 175.082 ms"
```

##### C04

```
"Gather  (cost=1135.64..5263.46 rows=45 width=3) (actual time=2.719..38.762 rows=3027 loops=1)"
"  Workers Planned: 1"
"  Workers Launched: 1"
"  ->  Hash Join  (cost=135.64..4258.96 rows=26 width=3) (actual time=1.070..14.268 rows=1514 loops=2)"
"        Hash Cond: (f.cd_ibge = m.cd_ibge)"
"        ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=11) (actual time=0.002..5.318 rows=124320 loops=2)"
"        ->  Hash  (cost=135.62..135.62 rows=1 width=8) (actual time=2.114..2.115 rows=1 loops=1)"
"              Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"              ->  Seq Scan on tb_mun m  (cost=0.00..135.62 rows=1 width=8) (actual time=1.310..2.106 rows=1 loops=1)"
"                    Filter: (nome_municipio = 'Salvador'::text)"
"                    Rows Removed by Filter: 5569"
"Planning Time: 11.191 ms"
"Execution Time: 38.857 ms"
```

##### C05

```
"Gather  (cost=6402.83..20817.32 rows=82880 width=21) (actual time=54.940..152.014 rows=77380 loops=1)"
"  Workers Planned: 1"
"  Workers Launched: 1"
"  ->  Parallel Hash Join  (cost=5402.83..11529.32 rows=48753 width=21) (actual time=37.755..119.342 rows=38690 loops=2)"
"        Hash Cond: (d.id_familia = f.id_familia)"
"        Join Filter: (d.qtd_comodos_dormitorio_fam < f.qtde_pessoas)"
"        Rows Removed by Join Filter: 85630"
"        ->  Parallel Seq Scan on tb_domicilio d  (cost=0.00..4502.59 rows=146259 width=17) (actual time=0.004..7.120 rows=124320 loops=2)"
"        ->  Parallel Hash  (cost=3574.59..3574.59 rows=146259 width=17) (actual time=37.123..37.125 rows=124320 loops=2)"
"              Buckets: 262144  Batches: 1  Memory Usage: 15520kB"
"              ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=17) (actual time=0.009..10.354 rows=124320 loops=2)"
"Planning Time: 0.159 ms"
"Execution Time: 153.443 ms"
```

##### C06

```
"Finalize GroupAggregate  (cost=8606.21..8609.92 rows=27 width=35) (actual time=85.445..90.136 rows=27 loops=1)"
"  Group Key: m.uf"
"  ->  Gather Merge  (cost=8606.21..8609.31 rows=27 width=35) (actual time=85.426..90.090 rows=54 loops=1)"
"        Workers Planned: 1"
"        Workers Launched: 1"
"        ->  Sort  (cost=7606.20..7606.26 rows=27 width=35) (actual time=68.667..68.669 rows=27 loops=2)"
"              Sort Key: m.uf"
"              Sort Method: quicksort  Memory: 27kB"
"              Worker 0:  Sort Method: quicksort  Memory: 27kB"
"              ->  Partial HashAggregate  (cost=7605.22..7605.56 rows=27 width=35) (actual time=68.613..68.619 rows=27 loops=2)"
"                    Group Key: m.uf"
"                    Batches: 1  Memory Usage: 32kB"
"                    Worker 0:  Batches: 1  Memory Usage: 32kB"
"                    ->  Hash Join  (cost=191.32..5776.98 rows=146259 width=6) (actual time=1.172..31.805 rows=124320 loops=2)"
"                          Hash Cond: (f.cd_ibge = m.cd_ibge)"
"                          ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=11) (actual time=0.006..5.805 rows=124320 loops=2)"
"                          ->  Hash  (cost=121.70..121.70 rows=5570 width=11) (actual time=1.135..1.135 rows=5570 loops=2)"
"                                Buckets: 8192  Batches: 1  Memory Usage: 298kB"
"                                ->  Seq Scan on tb_mun m  (cost=0.00..121.70 rows=5570 width=11) (actual time=0.181..0.642 rows=5570 loops=2)"
"Planning Time: 0.452 ms"
"Execution Time: 90.276 ms"
```

##### C07

```
"Finalize Aggregate  (cost=6599.65..6599.68 rows=1 width=32) (actual time=39.529..47.715 rows=1 loops=1)"
"  ->  Gather  (cost=6599.54..6599.65 rows=1 width=16) (actual time=33.367..47.677 rows=2 loops=1)"
"        Workers Planned: 1"
"        Workers Launched: 1"
"        ->  Partial Aggregate  (cost=5599.54..5599.55 rows=1 width=16) (actual time=16.580..16.581 rows=1 loops=2)"
"              ->  Parallel Seq Scan on tb_domicilio  (cost=0.00..4502.59 rows=146259 width=4) (actual time=0.003..5.562 rows=124320 loops=2)"
"Planning Time: 0.127 ms"
"Execution Time: 47.775 ms"
```

##### C08

```
"Limit  (cost=8751.74..8751.99 rows=100 width=24) (actual time=79.076..79.855 rows=100 loops=1)"
"  ->  Sort  (cost=8751.74..8764.98 rows=5297 width=24) (actual time=79.075..79.849 rows=100 loops=1)"
"        Sort Key: (count(tb_familia.id_familia)) DESC"
"        Sort Method: top-N heapsort  Memory: 36kB"
"        ->  Finalize HashAggregate  (cost=8496.32..8549.29 rows=5297 width=24) (actual time=77.386..78.986 rows=5414 loops=1)"
"              Group Key: tb_mun.nome_municipio, tb_mun.uf"
"              Batches: 1  Memory Usage: 721kB"
"              ->  Gather  (cost=7873.92..8456.59 rows=5297 width=24) (actual time=73.380..76.026 rows=10084 loops=1)"
"                    Workers Planned: 1"
"                    Workers Launched: 1"
"                    ->  Partial HashAggregate  (cost=6873.92..6926.89 rows=5297 width=24) (actual time=56.167..56.867 rows=5042 loops=2)"
"                          Group Key: tb_mun.nome_municipio, tb_mun.uf"
"                          Batches: 1  Memory Usage: 721kB"
"                          Worker 0:  Batches: 1  Memory Usage: 721kB"
"                          ->  Hash Join  (cost=191.32..5776.98 rows=146259 width=29) (actual time=1.407..31.821 rows=124320 loops=2)"
"                                Hash Cond: (tb_familia.cd_ibge = tb_mun.cd_ibge)"
"                                ->  Parallel Seq Scan on tb_familia  (cost=0.00..3574.59 rows=146259 width=21) (actual time=0.007..5.855 rows=124320 loops=2)"
"                                ->  Hash  (cost=121.70..121.70 rows=5570 width=24) (actual time=1.367..1.368 rows=5570 loops=2)"
"                                      Buckets: 8192  Batches: 1  Memory Usage: 370kB"
"                                      ->  Seq Scan on tb_mun  (cost=0.00..121.70 rows=5570 width=24) (actual time=0.176..0.714 rows=5570 loops=2)"
"Planning Time: 0.725 ms"
"Execution Time: 80.219 ms"
```
##### C09

```
"Limit  (cost=7513.29..7513.32 rows=10 width=11) (actual time=76.297..81.547 rows=10 loops=1)"
"  ->  Sort  (cost=7513.29..7513.36 rows=27 width=11) (actual time=76.296..81.546 rows=10 loops=1)"
"        Sort Key: (count(tb_familia.id_familia)) DESC"
"        Sort Method: top-N heapsort  Memory: 25kB"
"        ->  Finalize GroupAggregate  (cost=7509.20..7512.71 rows=27 width=11) (actual time=76.260..81.532 rows=27 loops=1)"
"              Group Key: tb_mun.uf"
"              ->  Gather Merge  (cost=7509.20..7512.30 rows=27 width=11) (actual time=76.253..81.519 rows=54 loops=1)"
"                    Workers Planned: 1"
"                    Workers Launched: 1"
"                    ->  Sort  (cost=6509.19..6509.25 rows=27 width=11) (actual time=59.976..59.978 rows=27 loops=2)"
"                          Sort Key: tb_mun.uf"
"                          Sort Method: quicksort  Memory: 25kB"
"                          Worker 0:  Sort Method: quicksort  Memory: 25kB"
"                          ->  Partial HashAggregate  (cost=6508.28..6508.55 rows=27 width=11) (actual time=59.920..59.923 rows=27 loops=2)"
"                                Group Key: tb_mun.uf"
"                                Batches: 1  Memory Usage: 24kB"
"                                Worker 0:  Batches: 1  Memory Usage: 24kB"
"                                ->  Hash Join  (cost=191.32..5776.98 rows=146259 width=16) (actual time=1.314..38.862 rows=124320 loops=2)"
"                                      Hash Cond: (tb_familia.cd_ibge = tb_mun.cd_ibge)"
"                                      ->  Parallel Seq Scan on tb_familia  (cost=0.00..3574.59 rows=146259 width=21) (actual time=0.009..7.089 rows=124320 loops=2)"
"                                      ->  Hash  (cost=121.70..121.70 rows=5570 width=11) (actual time=1.269..1.269 rows=5570 loops=2)"
"                                            Buckets: 8192  Batches: 1  Memory Usage: 298kB"
"                                            ->  Seq Scan on tb_mun  (cost=0.00..121.70 rows=5570 width=11) (actual time=0.226..0.721 rows=5570 loops=2)"
"Planning Time: 0.228 ms"
"Execution Time: 81.650 ms"
```
##### C10

```
"Gather Merge  (cost=17715.40..19734.34 rows=17556 width=38) (actual time=134.239..149.578 rows=29799 loops=1)"
"  Workers Planned: 1"
"  Workers Launched: 1"
"  ->  Sort  (cost=16715.39..16759.28 rows=17556 width=38) (actual time=114.149..115.266 rows=14900 loops=2)"
"        Sort Key: t.id_familia, t.id_pessoa"
"        Sort Method: quicksort  Memory: 1249kB"
"        Worker 0:  Sort Method: quicksort  Memory: 1199kB"
"        ->  Hash Join  (cost=11040.11..15477.72 rows=17556 width=38) (actual time=42.195..73.843 rows=14900 loops=2)"
"              Hash Cond: (f.cd_ibge = m.cd_ibge)"
"              ->  Parallel Hash Join  (cost=10848.78..15045.00 rows=17556 width=33) (actual time=40.963..69.150 rows=14900 loops=2)"
"                    Hash Cond: (f.id_familia = t.id_familia)"
"                    ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=21) (actual time=0.003..5.458 rows=124320 loops=2)"
"                    ->  Parallel Hash  (cost=10693.34..10693.34 rows=12435 width=25) (actual time=40.864..40.865 rows=14900 loops=2)"
"                          Buckets: 32768  Batches: 1  Memory Usage: 2144kB"
"                          ->  Parallel Seq Scan on tb_trab t  (cost=0.00..10693.34 rows=12435 width=25) (actual time=0.184..35.515 rows=14900 loops=2)"
"                                Filter: (qtd_meses_12_meses_memb <= 3)"
"                                Rows Removed by Filter: 328142"
"              ->  Hash  (cost=121.70..121.70 rows=5570 width=21) (actual time=1.172..1.175 rows=5570 loops=2)"
"                    Buckets: 8192  Batches: 1  Memory Usage: 353kB"
"                    ->  Seq Scan on tb_mun m  (cost=0.00..121.70 rows=5570 width=21) (actual time=0.103..0.562 rows=5570 loops=2)"
"Planning Time: 0.254 ms"
"Execution Time: 150.472 ms"
```
##### C11

```
"Sort  (cost=94621.57..95112.95 rows=196550 width=21) (actual time=508.698..521.026 rows=145729 loops=1)"
"  Sort Key: (sum(tb_trab.val_renda_bruta_12_meses_memb)) DESC"
"  Sort Method: external merge  Disk: 4856kB"
"  ->  HashAggregate  (cost=63292.96..73308.37 rows=196550 width=21) (actual time=306.942..471.977 rows=145729 loops=1)"
"        Group Key: tb_trab.id_familia"
"        Filter: (sum(tb_trab.val_renda_bruta_12_meses_memb) IS NOT NULL)"
"        Planned Partitions: 4  Batches: 5  Memory Usage: 8241kB  Disk Usage: 20088kB"
"        Rows Removed by Filter: 102912"
"        ->  Seq Scan on tb_trab  (cost=0.00..13980.82 rows=686082 width=17) (actual time=0.060..46.454 rows=686082 loops=1)"
"Planning Time: 0.121 ms"
"Execution Time: 532.876 ms"
```
##### C12

```
"Sort  (cost=9229.35..9242.59 rows=5297 width=21) (actual time=98.173..100.331 rows=4960 loops=1)"
"  Sort Key: (count(tb_familia.id_familia)) DESC"
"  Sort Method: quicksort  Memory: 389kB"
"  ->  Finalize HashAggregate  (cost=8848.73..8901.70 rows=5297 width=21) (actual time=97.107..99.460 rows=4960 loops=1)"
"        Group Key: tb_mun.nome_municipio"
"        Batches: 1  Memory Usage: 721kB"
"        ->  Gather  (cost=8239.58..8822.25 rows=5297 width=21) (actual time=93.797..97.503 rows=8960 loops=1)"
"              Workers Planned: 1"
"              Workers Launched: 1"
"              ->  Partial HashAggregate  (cost=7239.58..7292.55 rows=5297 width=21) (actual time=77.614..78.272 rows=4480 loops=2)"
"                    Group Key: tb_mun.nome_municipio"
"                    Batches: 1  Memory Usage: 721kB"
"                    Worker 0:  Batches: 1  Memory Usage: 721kB"
"                    ->  Hash Join  (cost=191.32..6995.81 rows=48753 width=26) (actual time=1.386..62.339 rows=86111 loops=2)"
"                          Hash Cond: (tb_familia.cd_ibge = tb_mun.cd_ibge)"
"                          ->  Parallel Seq Scan on tb_familia  (cost=0.00..6134.13 rows=48753 width=21) (actual time=0.017..44.847 rows=86111 loops=2)"
"                                Filter: (((replace(vlr_renda_media_fam, ','::text, '.'::text))::numeric / (NULLIF(qtde_pessoas, 0))::numeric) < '375'::numeric)"
"                                Rows Removed by Filter: 38210"
"                          ->  Hash  (cost=121.70..121.70 rows=5570 width=21) (actual time=1.333..1.333 rows=5570 loops=2)"
"                                Buckets: 8192  Batches: 1  Memory Usage: 353kB"
"                                ->  Seq Scan on tb_mun  (cost=0.00..121.70 rows=5570 width=21) (actual time=0.190..0.691 rows=5570 loops=2)"
"Planning Time: 0.282 ms"
"Execution Time: 100.799 ms"
```

##### C13

```
"Unique  (cost=29663.78..29703.51 rows=5297 width=21) (actual time=310.176..313.864 rows=4492 loops=1)"
"  ->  Sort  (cost=29663.78..29677.03 rows=5297 width=21) (actual time=310.174..313.426 rows=4492 loops=1)"
"        Sort Key: (count(p.id_pessoa)) DESC, m.nome_municipio"
"        Sort Method: quicksort  Memory: 371kB"
"        ->  Finalize HashAggregate  (cost=29283.17..29336.14 rows=5297 width=21) (actual time=303.518..307.160 rows=4492 loops=1)"
"              Group Key: m.nome_municipio"
"              Batches: 1  Memory Usage: 721kB"
"              ->  Gather  (cost=28117.83..29230.20 rows=10594 width=21) (actual time=298.259..303.971 rows=12411 loops=1)"
"                    Workers Planned: 2"
"                    Workers Launched: 2"
"                    ->  Partial HashAggregate  (cost=27117.83..27170.80 rows=5297 width=21) (actual time=262.825..263.344 rows=4137 loops=3)"
"                          Group Key: m.nome_municipio"
"                          Batches: 1  Memory Usage: 721kB"
"                          Worker 0:  Batches: 1  Memory Usage: 721kB"
"                          Worker 1:  Batches: 1  Memory Usage: 721kB"
"                          ->  Parallel Hash Join  (cost=15585.59..26540.03 rows=115560 width=21) (actual time=146.747..240.117 rows=114190 loops=3)"
"                                Hash Cond: (p.id_familia = f.id_familia)"
"                                ->  Parallel Seq Scan on tb_pessoa p  (cost=0.00..9202.67 rows=285868 width=21) (actual time=0.573..18.636 rows=228694 loops=3)"
"                                ->  Parallel Hash  (cost=14846.54..14846.54 rows=59124 width=39) (actual time=145.949..145.955 rows=33068 loops=3)"
"                                      Buckets: 131072  Batches: 1  Memory Usage: 8352kB"
"                                      ->  Hash Join  (cost=9664.17..14846.54 rows=59124 width=39) (actual time=100.139..137.266 rows=33068 loops=3)"
"                                            Hash Cond: (f.cd_ibge = m.cd_ibge)"
"                                            ->  Parallel Hash Join  (cost=9472.84..13842.26 rows=59124 width=34) (actual time=97.637..127.967 rows=33068 loops=3)"
"                                                  Hash Cond: (f.id_familia = e.id_familia)"
"                                                  ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=21) (actual time=0.004..4.242 rows=82880 loops=3)"
"                                                  ->  Parallel Hash  (cost=8949.34..8949.34 rows=41880 width=13) (actual time=97.404..97.405 rows=33068 loops=3)"
"                                                        Buckets: 131072  Batches: 1  Memory Usage: 5728kB"
"                                                        ->  Parallel Seq Scan on tb_esc e  (cost=0.00..8949.34 rows=41880 width=13) (actual time=0.714..85.642 rows=33068 loops=3)"
"                                                              Filter: (ind_frequenta_escola_memb = 4)"
"                                                              Rows Removed by Filter: 195626"
"                                            ->  Hash  (cost=121.70..121.70 rows=5570 width=21) (actual time=2.450..2.451 rows=5570 loops=3)"
"                                                  Buckets: 8192  Batches: 1  Memory Usage: 353kB"
"                                                  ->  Seq Scan on tb_mun m  (cost=0.00..121.70 rows=5570 width=21) (actual time=0.468..1.335 rows=5570 loops=3)"
"Planning Time: 8.233 ms"
"Execution Time: 314.374 ms"
```

##### C14

```
"Sort  (cost=105754.34..106251.04 rows=198679 width=37) (actual time=746.397..793.914 rows=248641 loops=1)"
"  Sort Key: (count(CASE WHEN ((idade >= 18) AND (idade < 60)) THEN 1 ELSE NULL::integer END)) DESC, (count(CASE WHEN (idade >= 60) THEN 1 ELSE NULL::integer END)) DESC, (count(CASE WHEN (idade < 18) THEN 1 ELSE NULL::integer END)) DESC"
"  Sort Method: external merge  Disk: 12192kB"
"  ->  HashAggregate  (cost=72808.19..82835.01 rows=198679 width=37) (actual time=355.411..637.630 rows=248641 loops=1)"
"        Group Key: id_familia"
"        Planned Partitions: 4  Batches: 21  Memory Usage: 8753kB  Disk Usage: 23528kB"
"        ->  Seq Scan on tb_pessoa  (cost=0.00..13204.82 rows=686082 width=17) (actual time=0.061..48.409 rows=686082 loops=1)"
"Planning Time: 0.103 ms"
"Execution Time: 816.959 ms"
```

##### C15

```
"Gather Merge  (cost=20687.07..20983.89 rows=2544 width=34) (actual time=145.272..152.037 rows=7738 loops=1)"
"  Workers Planned: 2"
"  Workers Launched: 2"
"  ->  Sort  (cost=19687.05..19690.23 rows=1272 width=34) (actual time=84.865..84.963 rows=2579 loops=3)"
"        Sort Key: m.nome_municipio"
"        Sort Method: quicksort  Memory: 304kB"
"        Worker 0:  Sort Method: quicksort  Memory: 290kB"
"        Worker 1:  Sort Method: quicksort  Memory: 25kB"
"        ->  Parallel Hash Join  (cost=9339.30..19621.46 rows=1272 width=34) (actual time=41.496..80.519 rows=2579 loops=3)"
"              Hash Cond: (p.id_familia = d.id_familia)"
"              ->  Parallel Seq Scan on tb_pessoa p  (cost=0.00..9202.67 rows=285868 width=21) (actual time=0.401..24.004 rows=343041 loops=2)"
"              ->  Parallel Hash  (cost=9331.16..9331.16 rows=651 width=39) (actual time=41.027..41.030 rows=877 loops=3)"
"                    Buckets: 4096 (originally 2048)  Batches: 1 (originally 1)  Memory Usage: 272kB"
"                    ->  Hash Join  (cost=5117.81..9331.16 rows=651 width=39) (actual time=12.601..53.441 rows=1316 loops=2)"
"                          Hash Cond: (f.cd_ibge = m.cd_ibge)"
"                          ->  Parallel Hash Join  (cost=4976.97..9151.19 rows=8698 width=34) (actual time=11.745..50.360 rows=7422 loops=2)"
"                                Hash Cond: (f.id_familia = d.id_familia)"
"                                ->  Parallel Seq Scan on tb_familia f  (cost=0.00..3574.59 rows=146259 width=21) (actual time=0.004..8.266 rows=124320 loops=2)"
"                                ->  Parallel Hash  (cost=4868.24..4868.24 rows=8698 width=13) (actual time=11.697..11.698 rows=7422 loops=2)"
"                                      Buckets: 16384  Batches: 1  Memory Usage: 832kB"
"                                      ->  Parallel Seq Scan on tb_domicilio d  (cost=0.00..4868.24 rows=8698 width=13) (actual time=0.005..20.823 rows=14844 loops=1)"
"                                            Filter: (cod_banheiro_domic_fam = 2)"
"                                            Rows Removed by Filter: 233797"
"                          ->  Hash  (cost=135.62..135.62 rows=417 width=21) (actual time=0.776..0.776 rows=417 loops=2)"
"                                Buckets: 1024  Batches: 1  Memory Usage: 30kB"
"                                ->  Seq Scan on tb_mun m  (cost=0.00..135.62 rows=417 width=21) (actual time=0.269..0.724 rows=417 loops=2)"
"                                      Filter: (uf = 'BA'::text)"
"                                      Rows Removed by Filter: 5153"
"Planning Time: 0.338 ms"
"Execution Time: 152.322 ms"
```

#### Estratégia de Indexação

Foram criados os índices para as tabelas **tb_pessoa, tb_familia e tb_mun**, através do comando a seguir:

```sql
CREATE INDEX ON tb_pessoa(id_familia);
CREATE INDEX ON tb_pessoa(id_pessoa);
CREATE INDEX ON tb_familia(id_familia);
CREATE INDEX ON tb_familia(cd_ibge);
CREATE INDEX ON tb_mun(cd_ibge);
CREATE INDEX ON tb_mun(uf);
CREATE INDEX ON tb_mun(nome_municipio);
```

Após a criação dos índices, as consultas foram executadas novamente. A seguir, há uma tabela comparativa dos tempos de execução:

##### Tabela Comparativa - Indexação

**Legenda:**
$T_0=\text{Tempo\ de\ execução\ inicial em ms}$
$T_1=\text{Tempo\ de\ execução\ pós-indexação em ms}$

| Cód. da Consulta | $T_0$ | $T_1$ |
| ---------------- | ----- | ----- |
| C01              | 189   | 103   |
| C02              | 19    | 18    |
| C03              | 186   | 75    |
| C04              | 50    | 2     |
| C05              | 154   | 159   |
| C06              | 91    | 177   |
| C07              | 47    | 36    |
| C08              | 81    | 86    |
| C09              | 82    | 68    |
| C10              | 151   | 152   |
| C11              | 533   | 613   |
| C12              | 101   | 134   |
| C13              | 323   | 280   |
| C14              | 817   | 641   |
| C15              | 153   | 82    |
Resultado: nota-se a utilização dos índices criados principalmente nas consultas **C04 e C15** (índices para **nome_municipio** e **uf**, respectivamente). Em **C04**, o tempo de execução diminuiu substancialmente. Eis abaixo a análise completa da consulta C04 para demonstrar a utilização dos índices:

```
"Nested Loop  (cost=5.09..240.38 rows=45 width=3) (actual time=0.283..1.805 rows=3027 loops=1)"
"  ->  Index Scan using tb_mun_nome_municipio_idx on tb_mun m  (cost=0.28..8.30 rows=1 width=8) (actual time=0.021..0.023 rows=1 loops=1)"
"        Index Cond: (nome_municipio = 'Salvador'::text)"
"  ->  Bitmap Heap Scan on tb_familia f  (cost=4.81..231.42 rows=66 width=11) (actual time=0.260..1.599 rows=3027 loops=1)"
"        Recheck Cond: (cd_ibge = m.cd_ibge)"
"        Heap Blocks: exact=999"
"        ->  Bitmap Index Scan on tb_familia_cd_ibge_idx  (cost=0.00..4.79 rows=66 width=0) (actual time=0.167..0.167 rows=3027 loops=1)"
"              Index Cond: (cd_ibge = m.cd_ibge)"
"Planning Time: 0.398 ms"
"Execution Time: 1.883 ms"
```

### Estratégia de Tuning

Fonte - [PGTune](pgtune.leopard.in.ua)

Para cada cenário, a consulta escolhida para comparativo de desempenho foi a **C15**.
#### Cenário 1 - Web Application

```sql
-- DB Version: 17
-- OS Type: windows
-- DB Type: web
-- Total Memory (RAM): 24 GB
-- CPUs num: 6
-- Data Storage: ssd

ALTER SYSTEM SET
 max_connections = '200';
ALTER SYSTEM SET
 shared_buffers = '6GB';
ALTER SYSTEM SET
 effective_cache_size = '18GB';
ALTER SYSTEM SET
 maintenance_work_mem = '1536MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '1.1';
ALTER SYSTEM SET
 work_mem = '10485kB';
ALTER SYSTEM SET
 huge_pages = 'off';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';
ALTER SYSTEM SET
 max_worker_processes = '6';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '3';
ALTER SYSTEM SET
 max_parallel_workers = '6';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '3';
 ```

#### Cenário 2 - Data Warehouse

```sql
-- DB Version: 17
-- OS Type: windows
-- DB Type: dw
-- Total Memory (RAM): 24 GB
-- CPUs num: 6
-- Data Storage: ssd

ALTER SYSTEM SET
 max_connections = '40';
ALTER SYSTEM SET
 shared_buffers = '6GB';
ALTER SYSTEM SET
 effective_cache_size = '18GB';
ALTER SYSTEM SET
 maintenance_work_mem = '2047MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '500';
ALTER SYSTEM SET
 random_page_cost = '1.1';
ALTER SYSTEM SET
 work_mem = '26214kB';
ALTER SYSTEM SET
 huge_pages = 'off';
ALTER SYSTEM SET
 min_wal_size = '4GB';
ALTER SYSTEM SET
 max_wal_size = '16GB';
ALTER SYSTEM SET
 max_worker_processes = '6';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '3';
ALTER SYSTEM SET
 max_parallel_workers = '6';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '3';
```

#### Cenário 3 - Tipo misto de aplicação

```sql
-- DB Version: 17
-- OS Type: windows
-- DB Type: mixed
-- Total Memory (RAM): 24 GB
-- CPUs num: 6
-- Data Storage: ssd

ALTER SYSTEM SET
 max_connections = '100';
ALTER SYSTEM SET
 shared_buffers = '6GB';
ALTER SYSTEM SET
 effective_cache_size = '18GB';
ALTER SYSTEM SET
 maintenance_work_mem = '1536MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '1.1';
ALTER SYSTEM SET
 work_mem = '10485kB';
ALTER SYSTEM SET
 huge_pages = 'off';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';
ALTER SYSTEM SET
 max_worker_processes = '6';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '3';
ALTER SYSTEM SET
 max_parallel_workers = '6';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '3';
```

#### Resultado Final Comparativo - Consulta C15

| Original s/ index | Original c/ index | Cenário 1 | Cenário 2 | Cenário 3 |
| ----------------- | ----------------- | --------- | --------- | --------- |
| 153ms             | 82ms              | 94ms      | 82ms      | 76ms      |

