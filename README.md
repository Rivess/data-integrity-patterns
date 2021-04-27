# Vzory pro zajištění integrity relační databáze v systému řízení báze dat Oracle

## Zajištění entitní integrity

#### Popis problému

Entitní integrita je v databází potřeba z toho důvodu, aby se v databází nemohli nalézat žádné duplicitní záznamy. To nám zajistí vyhnutí se případným problémům, které by díky tomuto mohli vzniknout.

#### Příklad problému

Databáze nedodržuje entitní integritu, do databáze se nám dostanou dva stejné záznamy. Když chceme jeden z těchto záznamů upravit, databáze neví, který z těchto dvou záznamů má upravit.

#### Řešení problému

K řešení toho problému se využívá primárního klíče. Primární klíč je nenulový unikátní identifikátor, díky kterému můžeme zajistit entitní integritu v relační databázi. Většina relačních databází naštěstí toto chování vynucuje, totéž platí i pro Oracle databázi

#### Příklad kódu

```sql
ALTER TABLE EVALUATIONS
ADD CONSTRAINT EVAL_EVAL_ID_PK PRIMARY KEY (EVALUATION_ID);
```

#### Typy omezení v Oracle

Omezení pomocí primarního klíče

## 

