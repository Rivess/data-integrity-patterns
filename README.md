# Vzory pro zajištění integrity relační databáze v systému řízení báze dat Oracle

## Zachování entitní integrity

#### Popis problému

Entitní integrita je v databází potřeba z toho důvodu, aby se v databází nemohli nalézat žádné duplicitní záznamy. To nám zajistí vyhnutí se případným problémům, které by díky tomuto mohli vzniknout.

#### Příklad problému

Databáze nedodržuje entitní integritu, do databáze se nám dostanou dva stejné záznamy. Když chceme jeden z těchto záznamů upravit, databáze neví, který z těchto dvou záznamů má upravit.

#### Řešení problému

K řešení toho problému se využívá primárního klíče. Primární klíč je nenulový unikátní identifikátor, díky kterému můžeme zajistit entitní integritu v relační databázi. Většina relačních databází naštěstí toto chování vynucuje, totéž platí i pro Oracle databázi.

#### Příklad kódu

Vytvoření primárního klíče:

```sql
ALTER TABLE EVALUATIONS
ADD CONSTRAINT EVAL_EVAL_ID_PK PRIMARY KEY (EVALUATION_ID);
```

#### Typ omezení podle Oracle dokumentace

Omezení pomocí primárního klíče, toto omezení kombinuje omezení na unikátní hodnoty a omezení vložení neznámé hodnoty.

## Zachování referenční integrity

#### Popis problému

Referenční integrita se zabývá vztahy jednotlivých záznamů v relační databázi. Z entitní integrity víme, že každý záznam musí mít svůj primární klíč, tento klíč můžeme použít v jiném záznamu jako cizí klíč. Cizí klíč slouží k referenci na záznam u kterého tento klíč je jako primární klíč. Když nám vznikne takováto logická závislost, tak při vymazání nadřazeného záznamu, nám může vzniknout chyba v systému.

#### Příklad problému

Máme databázi bankovní společnosti, v této databázi, existuje tabulka klientů banky a tabulka bankovních účtů. Každý záznam v tabulce bankovních účtů, má cizí klíč z tabulky klientů banky, jelikož každý účet musí patřit jednomu klientovi. Pokud ale smažeme klienta kterému patří alespoň jeden účet, v záznamu v tabulce s bankovnimi účty, nám zůstane cizí klíč, který neodkazuje na žádný záznam. Tomuto zabráníme dodržením referenční integrity.

#### Řešení problému

K řešení tohoto problému se využívá cizího klíče a nastavení toho co se má dít, při vymazání nebo upravě záznamu na který tento cizí klíč odkazuje. Když je vymazán nebo upraven záznam, na který cizí klíč odkazuje, můžeme nastavit následující možnosti toho co se stane:

* Žádná akce při mazání, nebo úpravě záznamu - to znamená, že uživatelé nemůžou vymazat nebo změnit hodnotu primárního klíče v záznamu na který odkazuje cízí klíč. Například pokud zaměstnanec patří do oddělení, tak toto oddělení není možné vymazat.
* Kaskádové odstranění \(DELETE CASCADE\) - Pokud záznam obsahující primární klíč, na který odkazují cizí klíče je vymazán, tak všechny záznamy s tímto cizím klíčem budou také vymazány.
* Odstranění, které nastaví cizí klíč na hodnotu null \(DELETE SET NULL\) - Pokud záznam obsahující primární klíč, na který odkazují cizí klíče je vymazán, tak všechny tyto cizí klíče jsou nastaveny na hodnotu null. 

#### Příklad kódu

Přidání cizího klíče, do již existující tabulky:

```sql
ALTER TABLE EVALUATIONS
ADD CONSTRAINT EVAL_EMP_ID_FK FOREIGN KEY (EMPLOYEE_ID)
REFERENCES EMPLOYEES (EMPLOYEE_ID);
```

Vytvoření tabulky s cizím klíčem, který má kaskádové odstranění:

```sql
CREATE TABLE Emp_tab (
FOREIGN KEY (Deptno) REFERENCES Dept_tab
ON DELETE CASCADE);
```

Vytvoření tabulky s cizím klíčem, který se nastaví na hodnotu null při odstranění:

```sql
CREATE TABLE Emp_tab (
FOREIGN KEY (Deptno) REFERENCES Dept_tab  
ON DELETE SET NULL); 
```

#### Typ omezení podle Oracle dokumentace

Omezení pomocí cizího klíče, toto omezení nastaví sloupec v tabulce jako cizí klíč, který odkazuje na primární klíč a jaká akce se provede při vymazání tohoto primárního klíče.

## Zachování doménové integrity

#### Popis problému

Doménová integrita je v databázi potřeba, aby se nám ve sloupcích neobjevovali nechtěná data. To znamená aby pro sloupec byly povoleny pouze hodnoty, které náležý do jeho domény.

#### Příklad problému

Máme v databázi definovaný sloupec pro datum nástupu zaměstnance. V aplikaci nastane chyba a do data nástupu zaměstnance se snaží uložit hodnotu, která není datum, například "hodnota". Pokud toto databáze umožní, tak se naruší doménová integrita této databáze.

#### Řešení problému

Tento problém se řeší pomocí kontrolních omezení a při tvorbě tabulky. Při vytváření tabulky se volí datový typ pro každý jednotlivý sloupec v tabulce, tím se dociluje lepší doménové integrity databáze. Dále pomocí kontrolních omezení, můžeme zajistit vyřešení ostatní problémů vznikajících při zachování doménové integrity, jako například, že plat zaměstnance není záporný, nebo že jméno dodavatele náleží výběru, který my zadáme.

#### Příklad kódu

Přidání omezení, aby jméno dodavatele bylo pouze ze zadaného výběru:

```sql
ALTER TABLE suppliers
ADD CONSTRAINT check_supplier_name
    CHECK (supplier_name IN ('IBM', 'Microsoft', 'NVIDIA'));
```

Přidání omezení pro zajištění nezáporného platu zaměstnance:

```sql
ALTER TABLE employees
ADD CONSTRAINT emp_salary_min_demo
    CHECK (salary > 0);
```

#### Typ omezení podle Oracle dokumentace

Omezení pomocí kontroly, požadují po hodnotě v databázi, aby splňovala zadané podmínky.

## Uživatelem definovaná integritní omezení

#### Popis problému

Uživatelem definované integritní omezení, nebo také někdy nazývaná byznysová omezení, jsou omezení, které vyplívají z požadavků uživatele databáze a jeho potřeb. 

#### Příklad problému

Uživatel potřebuje zajistit, aby v databázi byli uloženi pouze zaměstnanci s věkem větším než osmnáct let. Pokud nebudeme mít nastavené omezení, které bý vynucovalo toto pravidlo, do databáze se může dostat zaměstnanec s menším věkem než osmnáct let a vznikne nám chybový záznam v databázi.

#### Řešení problému

Tento problém se řeší pomocí kontrolních omezení stejně jako u doménových intgritních omezení. Pomocí kontrolních omezení, můžeme zajistit splnění uživatelem definovaných integritních požadavků.

#### Příklad kódu

Přidání kontrolního omezení, pro zajištění velikosti věku zaměstnance:

```sql
ALTER TABLE employees
ADD CONSTRAINT check_employee_age
    CHECK (employee_age >= 18);
```

#### Typ omezení podle Oracle dokumentace

Omezení pomocí kontroly, požadují po hodnotě v databázi, aby splňovala zadané podmínky.

## Zachování integrity neznámé hodnoty

#### Popis problému

Tato integrita řeší zda hodnota ve sloupci může nabýt neznámé hodnoty. Jelikož některé atributy musí mít vždy známou hodnotu, může nedodržení této integrity způsobit chybně zadaná data v databázi.

#### Příklad problému

Máme databázi zaměstnanců, kde zaměstnanec má atributy jméno, věk, plat. Věk zaměstnance nesmí nikdy nabýt neznámé hodnoty, jinak je tato integrita narušena

#### Řešení problému

Tento problém se řeší zakázáním neznámé hodnoty pro příslušné sloupce.

#### Příklad kódu

Přidání omezení neznámé hodnoty:

```sql
ALTER TABLE PERFORMANCE_PARTS
MODIFY WEIGHT NOT NULL;
```

#### Typ omezení podle Oracle dokumentace

Omezení pomocí nastavení, zda sloupec může nabýt neznámé hodnoty.

