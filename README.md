# Vzory pro zajištění integrity relační databáze v systému řízení báze dat Oracle

## Entitní integrita

#### Popis problému

Entitní integrita je v databází potřeba z toho důvodu, aby se v databází nemohli nalézat žádné duplicitní záznamy. To nám zajistí vyhnutí se případným problémům, které by díky tomuto mohli vzniknout.

#### Příklad problému

Databáze nedodržuje entitní integritu, do databáze se nám dostanou dva stejné záznamy. Když chceme jeden z těchto záznamů upravit, databáze neví, který z těchto dvou záznamů má upravit.

#### Řešení problému

K řešení toho problému se využívá primárního klíče. Primární klíč je nenulový unikátní identifikátor, díky kterému můžeme zajistit entitní integritu v relační databázi. Většina relačních databází naštěstí toto chování vynucuje, totéž platí i pro Oracle databázi

#### Příklad kódu

Vytvoření primárního klíče:

```sql
ALTER TABLE EVALUATIONS
ADD CONSTRAINT EVAL_EVAL_ID_PK PRIMARY KEY (EVALUATION_ID);
```

#### Typy omezení v Oracle dokumentaci

Omezení pomocí primarního klíče, toto omezení kombinuje omezení na unikatní hodnoty a omezení vložení hodnoty null.

## Referenční integrita

#### Popis problému

Referenční integrita se zabývá vztahy jednotlivých záznamů v relační databázi. Z entitní integrity víme, že každý záznam musí mít svůj primarní klíč, tento klíč můžeme použít v jiném záznamu jako cizí klíč. Cizí klíč slouží k referenci na záznam u kterého tento klíč je jako primarní klíč. Když nám vznikne takováto logická závislost, tak při vymazání nadřazeného záznamu, nám může vzniknout chyba v systému.

#### Příklad problému

Máme databázi bankovní společnosti, v této databázi, existuje tabulka klientů banky a tabulka bankovních účtů. Každý záznam v tabulce bankovních účtů, má cízí klíč z tabulky klientů banky, jelikož každý účet musí patřit jednomu klientovi. Pokud ale smažame klienta kterému patří alespoň jeden účet, v záznamu v tabulce s bankovnimi účty, nám zůstane cizí klíč, který neodkazuje na žádný záznam. Tomuto zabráníme dodržením referenční integrity.

#### Řešení problému

K řešení tohoto problému se využívá cizího klíče a nastavení toho co se má dít, při vymazání nebo uprávě záznamu na který tento cizí klíč odkazuje. Když je vymazán nebo upracven záznam na který cizí klíč odkazuje, můžeme nastavit následující možnosti toho co se stane:

* Žádná akce při mazání, nebo úpravě záznamu - to znamená, že uživatelé nemůžou vymazat nebo změnit hodnotu primárního klíče v záznamu na který odkazuje cízí klíč. Například pokud zaměstnanec patří do oddělení, tak toto oddělení není možné vymazat.
* Kaskádové odstranění \(DELETE CASCADE\) - Pokud záznam obsahující primarní klíč, na který odkazují cizí klíče je vymazán, tak všechny záznamy s tímto cizím klíčem budou také vymazány.
* Odstranění, které nastaví cizí klíč na hodnotu null \(DELETE SET NULL\) - Pokud záznam obsahující primární klíč, na který odkazují cizí klíče je vymazán, tak všechny tyto cizí klíče jsou nastaveny na hodnotu null. 

#### Příklad kódu

Přidání cizího klíče do jíž existující tabulky:

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

#### Typy omezení v Oracle dokumentaci

Omezení pomocí cizího klíče, toto omezení nastaví sloupec v tabulce jako cizí klíč, který odkazuje na primarní klíč a jaká akce se provede při vymazání tohoto primarního klíče.

