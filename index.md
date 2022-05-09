# Katalog vzorů řešení pro problémy při zajištění integrity v relačním systému řízení báze dat Oracle

Na této stránce je uveden katalog vzorů řešení pro problémy při zajištění integrity v relačním systémů řízení báze dat Oracle, který vznikl jako výsledek mé bakalářské práce.

[Zachování entitní integrity](#Zachování entitní integrity)

## Zachování entitní integrity

### Popis problému

‌Entitní integrita je v databází potřeba z toho důvodu, aby se v databázi nemohly nalézat žádné duplicitní záznamy. To nám zajistí vyhnutí se případným problémům, které by kvůli tomuto mohly vzniknout.‌

### Příklad problému

‌‌Databáze nedodržuje entitní integritu, do databáze se nám dostanou dva stejné záznamy. Když chceme jeden z těchto záznamů upravit, databáze neví, který z těchto dvou záznamů má upravit.‌

### Řešení problému

‌K řešení toho problému se využívá primárního klíče. Primární klíč je nenulový unikátní identifikátor, díky kterému můžeme zajistit entitní integritu v relační databázi. Většina relačních databází naštěstí toto chování vynucuje, totéž platí i pro Oracle databázi (Oracle, 2022j)‌.

### Důsledky řešení

Použití primárních klíčů nám přináší i rychlejší vyhledávání dat v databázi, jelikož databáze Oracle automaticky indexuje primární klíče. Negativní důsledky zde nejsou, jelikož primární klíč je povinný v systému databáze Oracle, tím pádem tento problém nemá ani alternativní řešení.

### Příklad kódu

Vytvoření tabulky klientů s identifikátorem, který je použit jako primární klíč:

```sql
CREATE TABLE clients (
    client_id    NUMBER
        GENERATED ALWAYS AS IDENTITY,
    birthday     DATE NOT NULL,
    phone_number CHAR(9),
    created_at   TIMESTAMP NOT NULL,
    verified     CHAR(1) DEFAULT 0 NOT NULL,
    PRIMARY KEY ( client_id )
);
```

## Přirozený primární klíč versus umělý primární klíč

### Popis problému

Při modelování databáze nám může vyvstat otázka, zda při vytváření tabulky použít přirozený nebo umělý primární klíč, kde přirozený primární klíč znamená použití již existujícího atributu dané entity, zatímco umělý primární klíč je atribut vytvořen čistě pro účel primárního klíče. Problém nám může vzniknout, pokud by ve všech tabulkách byl použit umělý primární klíč, avšak ve spoustě tabulkách není tento umělý primární klíč potřeba, jelikož spousta entit již má atribut perfektně vyhovující roli primárního klíče a tím pádem akorát přidáváme další atribut bez role, který nám pouze zpomaluje transakční operace na danou tabulku. Na druhou stranu se může stát, že použijeme již existující atribut, který si myslíme, že splňuje všechny podmínky k tomu, aby byl primárním klíčem, ale v reálném světě se například může jeho hodnota změnit a tím pádem není vyhovující pro roli primárního klíče.

### Příklad problému

Nevhodné použití umělého primárního klíče v případě, kdy není umělý klíč potřeba. Nebo vybrání nevhodného primárního klíče z již existujících atributů dané entity.

### Řešení problému

Při výběru primárního klíče se chceme řídit několika pravidly (Logicalmind, 2008):

* Primární klíč by měl být tak malý, jak je potřeba a měly by být preferovány číselné primární klíče, jelikož číselné typy jsou uložený v mnohem kompaktnějším formátu než znakové formáty. Tohle je důležité, jelikož většina primárních klíčů bude uložena v jiných tabulkách jako cizí klíč a také budou použity jako indexy. Čím menší primární klíč, tím menší index a tím lepší výkonost bude mít naše databáze.
* Primární klíče by se nikdy neměly měnit. Aktualizace primárního klíče by nám nikdy neměla přijít na mysl, jelikož je vysoce pravděpodobné, že primární klíč bude použit ve více indexech a jako cizí klíč. Aktualizace pouze jednoho primárního klíče nám může způsobit spoustu nepředpověditelných problémů.
* Nelze vždy použít první atribut který se nabízí jako přirozený primární klíč, je vždy potřeba se zamyslet, zda daný klíč vyhovuje všem požadavkům, jako například číslo pasu nebo číslo smlouvy zaměstnance, tyto atributy jsou nevyhovující pro primární klíč, jelikož se v reálném světě v jistých případech mohou změnit. Případy podobné těmto příkladům by měly mít nastaveno omezení jedinečnosti, aby nám v databázi nevznikly chyby, ale neměly by být použity jako primární klíče.

### Důsledky řešení

Řízení se pravidly při výběru primárního klíče nám přináší konzistenci v našem navrhování databázových modelů a pomáhá se vyvarovat nevhodnému vybrání primárního klíče. Alternativní řešení je použití umělého primárního klíče pro každou tabulku, to ale není optimální, jelikož ve spoustě tabulkách nám akorát přibude zbytečný sloupec bez role.

### Příklad kódu

Tento problém je bez příkladu kódu.

## Zachování referenční integrity

### Popis problému

Referenční integrita se zabývá vztahy jednotlivých záznamů v relační databázi (Oracle, 2022d). Z entitní integrity víme, že každý záznam musí mít svůj primární klíč, tento klíč můžeme použít v jiném záznamu jako cizí klíč. Cizí klíč slouží k referenci na záznam, u kterého tento klíč je jako primární klíč. Když nám vznikne takováto logická závislost, tak při vymazání nadřazeného záznamu, nám může vzniknout chyba v systému.

### Příklad problému

Máme databázi bankovní společnosti, v této databázi, existuje tabulka klientů banky a tabulka bankovních účtů. Každý záznam v tabulce bankovních účtů má cizí klíč z tabulky klientů banky, jelikož každý účet musí patřit jednomu klientovi. Pokud ale smažeme klienta, kterému náleží alespoň jeden účet, v záznamu v tabulce s bankovními účty nám zůstane cizí klíč, který neodkazuje na žádný záznam. Tím se nám naruší integrita dat v naší databázi.

### Řešení problému

K řešení tohoto problému se využívá cizího klíče a nastavení toho, co se má dít při vymazání nebo úpravě záznamu, na který tento cizí klíč odkazuje (Oracle, 2022d). Když je vymazán nebo upraven záznam, na který cizí klíč odkazuje, můžeme nastavit následující možnosti toho, co se stane:

* Žádná akce při mazání nebo úpravě záznamu – to znamená, že uživatelé nemůžou vymazat nebo změnit hodnotu primárního klíče v záznamu, na který odkazuje cizí klíč. Například pokud zaměstnanec patří do oddělení, tak toto oddělení nám databáze neumožní vymazat, pokud bude existovat alespoň jeden zaměstnanec s cizím klíčem odkazujícím na toto oddělení.
* Kaskádové odstranění (DELETE CASCADE) - Pokud záznam obsahující primární klíč, na který odkazují cizí klíče je vymazán, tak všechny záznamy s tímto cizím klíčem budou také vymazány.
* Odstranění, které nastaví cizí klíč na neznámou hodnotu (DELETE SET NULL) - Pokud záznam obsahující primární klíč, na který odkazují cizí klíče je vymazán, tak všechny tyto cizí klíče jsou nastaveny na neznámou hodnotu.

### Důsledky řešení

Používání cizích klíčů je vyžadováno pro zmezení chyb v naší databázi. Pokud cizí klíče použijeme také jako indexy, tak nám můžou urychlit naší databázi. Alternativně můžeme tuto logiku nechat řešit aplikační vrstvu, to se zásadně nedoporučuje, už jenom z toho důvodu, že náš systém může používat více aplikací a my nikdy nemůžeme věřit, že aplikace tuto logiku pohlídá.

### Příklad kódu

Vytvoření ukázky tabulky bankovních klientů:

```sql
CREATE TABLE clients (
    client_id    NUMBER
        GENERATED ALWAYS AS IDENTITY,
    birthday     DATE NOT NULL,
    phone_number CHAR(9),
    created_at   TIMESTAMP NOT NULL,
    verified     CHAR(1) DEFAULT 0 NOT NULL,
    PRIMARY KEY ( client_id )
);
```

Vytvoření tabulky účtů s kaskádovým omezením:

```sql
CREATE TABLE accounts (
    account_id NUMBER
        GENERATED ALWAYS AS IDENTITY,
    client_id  NUMBER NOT NULL,
    type       VARCHAR2 NOT NULL,
    amount     NUMBER DEFAULT 0 NOT NULL,
    created_at TIMESTAMP NOT NULL,
    PRIMARY KEY ( account_id ),
    FOREIGN KEY ( client_id )
        REFERENCES clients
            ON DELETE CASCADE
);
```

Schéma vytvořených tabulek pro lepší představu vztahu:

![Obrázek 1 Schéma tabulek účety a klienti](.gitbook/assets/clients\_accounts\_relation.png)

## Zajištění vztahu mnoho k mnoha

### Popis problému

Velmi častý případ referenční integrity, je vztah mnoho k mnoha. Například nákupní košík na e-shopu a produktů daného e-shopu, kde v objednávce může být více produktů a produkt může být ve více objednávkách. Tohoto vztahu nelze docílit pouze využitím cizího klíče.

### Příklad problému

Máme databázi pro e-shop, kde je tabulka představující nákupní košík a tabulka produktů. Potřebujeme mezi těmito tabulkami vytvořit vztah mnoho k mnoha.

### Řešení problému

Tento problém vyřešíme vytvořením pomocné tabulky, která bude představovat vztah mnoho k mnoha mezi našimi dvěma tabulkami. Může nám vyvstat otázka ohledně primárního klíče, zda do této tabulky přidat umělý primární klíč (Mitty, 2011). Přidávat umělý klíč do tabulky pro mnoho k mnoha vztah není dobré, jelikož vždy v tabulce hledáme podle identifikátoru košíku nebo produktu. Lepší řešení je použít složený primární klíč, který nám zároveň zajistí, aby v této tabulce nebyly duplikované záznamy.

### Důsledky řešení

Řešení nám umožňuje vytvoření vztahu mnoho k mnoha na fyzické úrovni. Použití složeného primárního klíče nám urychluje vyhledávání v databázi a zajišťuje, že nemáme žádné duplicity.

### Příklad kódu

Pro náš příklad si vytvoříme tabulku s uživateli, která nemá se vztahem nic společného, ale je potřeba k tabulce nákupního košíku:

```sql
CREATE TABLE users (
    user_id     NUMBER
        GENERATED ALWAYS AS IDENTITY
    NOT NULL,
    created_at  TIMESTAMP NOT NULL,
    modified_at TIMESTAMP,
    PRIMARY KEY ( user_id )
);
```

Dále vytvoříme tabulku pro nákupní košíky:

```sql
CREATE TABLE carts (
    cart_id     NUMBER
        GENERATED ALWAYS AS IDENTITY
    NOT NULL,
    user_id     NUMBER NOT NULL,
    price       NUMBER DEFAULT 0 NOT NULL,
    created_at  TIMESTAMP NOT NULL,
    modified_at TIMESTAMP,
    PRIMARY KEY ( cart_id ),
    FOREIGN KEY ( user_id )
        REFERENCES users
            ON DELETE CASCADE
);
```

Poté vytvoříme tabulku s produkty:

```sql
CREATE TABLE products (
    product_id  NUMBER
        GENERATED ALWAYS AS IDENTITY
    NOT NULL,
    price       NUMBER NOT NULL,
    category    VARCHAR2(10) NOT NULL,
    created_at  TIMESTAMP NOT NULL,
    modified_at TIMESTAMP,
    PRIMARY KEY ( product_id )
);
```

Nakonec vytvoříme tabulku pro náš vztah mnoho k mnoha, v tabulce je přidán sloupec počet daného produktu v košíku:

```sql
CREATE TABLE cart_to_product (
    cart_id    NUMBER NOT NULL,
    product_id NUMBER NOT NULL,
    count      NUMBER NOT NULL,
    PRIMARY KEY ( cart_id,
                  product_id ),
    FOREIGN KEY ( cart_id )
        REFERENCES carts
            ON DELETE CASCADE,
    FOREIGN KEY ( product_id )
        REFERENCES products
            ON DELETE CASCADE
);
```

Schéma vytvořených tabulek pro lepší představu vztahu:

![Obrázek 2 Schéma vztahu mnoho k mnoha](.gitbook/assets/cart\_to\_product.png)

## Zachování doménové integrity

### Popis problému

Doménová integrita je v databázi potřeba, aby se nám ve sloupcích neobjevovala nechtěná data. To znamená, aby pro sloupec byly povoleny pouze hodnoty, které náleží do jeho domény.

### Příklad problému

Máme v databázi definovaný sloupec pro datum nástupu zaměstnance. V aplikaci nastane chyba a do data nástupu zaměstnance se snaží uložit hodnotu, která není datum, například "hodnota", dále chceme mít v tabulce telefonní číslo pro které platí, že má devět číslic.

### Řešení problému

Tento problém se řeší pomocí kontrolních omezení a při tvorbě tabulky. Při vytváření tabulky se volí datový typ pro každý jednotlivý sloupec v tabulce, tím se dociluje lepší doménové integrity databáze (Oracle, 2022h). Pro dodržení správných hodnot ve sloupci s datem nástupu zvolíme datový typ datum. Pro dodržení počtu číslic telefonního čísla zvolíme datový typ znak s velikostí devět znaků.

### Důsledky řešení

Nastavení datových typů nám zajišťuje doménovou integritu, jedná se o využití základní mechaniky v systému databáze Oracle.

### Příklad kódu

Vytvoření tabulky zaměstnanců se sloupcem data nástupu, který má datový typ data (DATE) a sloupec telefonní číslo a délce devíti znaků:

```sql
CREATE TABLE employees (
    employee_id   NUMBER
        GENERATED ALWAYS AS IDENTITY,
    birthday      DATE NOT NULL,
    boarding_date DATE NOT NULL,
    created_at    TIMESTAMP NOT NULL,
    salary        NUMBER NOT NULL,
    phone_number CHAR(9),
    PRIMARY KEY ( employee_id )
);
```

## Kontrola věku z data narození

### Popis problému

Potřebujeme zkontrolovat věk a máme k dispozici datum narození v databázi. Nelze vytvořit jednoduché kontrolní omezení, jelikož potřebujeme zjistit dnešní datum a kontrolní omezení musí být deterministická, takže v nich nelze použít databázové funkce k zjištění aktuálního data (sysdate) (Oracle, 2022b).

### Příklad problému

Uživatel potřebuje zajistit, aby v databázi byli uloženi pouze zaměstnanci s věkem větším než osmnáct let. Pokud nebudeme mít nastavené omezení, které by vynucovalo toto pravidlo, do databáze se může dostat zaměstnanec s menším věkem než osmnáct let a vznikne nám chybový záznam v databázi.

### Řešení problému

Tento problém vyřešíme pomocí triggeru, který se vyvolá při každém vložení nebo aktualizaci (Oracle, 2016a). Řešení funguje tak, že trigger pro každé vložení věku nebo aktualizaci věku, vezme dnešní datum a datum narození, z těchto hodnot vypočte věk a porovná ho s naším nastaveným limitem, pokud je věk menší než nastavený limit, tak se změny neprovedou.

### Důsledky řešení

Toto řešení může přinést zpomalení naší databáze, jelikož vytváříme trigger, který se spouští pro každý řádek daného vložení či aktualizace, v případě, že se jedná o tabulku s častým vkládáním či aktualizací mnoha řádků, může nastat zpomalení naší databáze.

Alternativní řešení je použití uložených procedur a balíčku (Oracle, 2022i). V našem případě vytvoříme v balíčku procedury pro aktualizaci data narození a vložení dat do tabulky uživatele, v těchto procedurách se zároveň bude kontrolovat naplnění naší podmínky. Tyto procedury se pak budou používat pro změnu věku a vložení nového uživatele.

### Příklad kódu

Přidání triggeru, který kontroluje věk při přidávání nebo aktualizaci záznamu v tabulce zaměstnanců:

```sql
CREATE OR REPLACE TRIGGER check_birth_date BEFORE
    INSERT OR UPDATE ON employees
    FOR EACH ROW
BEGIN
    IF ( trunc((to_number(to_char(sysdate, 'YYYYMMDD')) - to_number(to_char(:new.birthday, 'YYYYMMDD'))) / 10000) < 18 ) THEN
        raise_application_error(-20001, 'Employee age is not 18 and above!');
    END IF;
END;
```

Alternativní řešení, vytvoření balíčku s procedurami:

```sql
CREATE OR REPLACE PACKAGE pkg_age_check IS
    PROCEDURE employee_insert (
        birthday      DATE,
        boarding_date DATE,
        created_at    TIMESTAMP,
        salary        NUMBER,
        phone_number  CHAR
    );

    PROCEDURE employee_age_update (
        employee_id NUMBER,
        birthday    DATE
    );

END pkg_age_check;

CREATE OR REPLACE PACKAGE BODY pkg_age_check IS

    PROCEDURE employee_insert (
        birthday      DATE,
        boarding_date DATE,
        created_at    TIMESTAMP,
        salary        NUMBER,
        phone_number  CHAR
    ) IS
    BEGIN
        IF ( trunc((to_number(to_char(sysdate, 'YYYYMMDD')) - to_number(to_char(birthday, 'YYYYMMDD'))) / 10000) < 18 ) THEN
            raise_application_error(-20001, 'Employee age is not 18 and above!');
        ELSE
            INSERT INTO employees(birthday, boarding_date, created_at, salary, phone_number) VALUES (
                birthday,
                boarding_date,
                created_at,
                salary,
                phone_number
            );

        END IF;
    END;

    PROCEDURE employee_age_update (
        employee_id NUMBER,
        birthday    DATE
    ) IS
    BEGIN
        IF ( trunc((to_number(to_char(sysdate, 'YYYYMMDD')) - to_number(to_char(birthday, 'YYYYMMDD'))) / 10000) < 18 ) THEN
            raise_application_error(-20001, 'Employee age is not 18 and above!');
        ELSE
            UPDATE employees
            SET
                birthday = birthday
            WHERE
                employee_id = employee_id;

        END IF;
    END;

END pkg_age_check;

```

## Zachování integrity neznámé hodnoty

### Popis problému

Tato integrita řeší, zda hodnota ve sloupci může nabýt neznámé hodnoty. Jelikož některé atributy musí mít vždy známou hodnotu, může nedodržení této integrity způsobit chybně zadaná data v databázi.

### Příklad problému

Máme databázi zaměstnanců, kde zaměstnanec má atributy narozeniny, datum nástupu, plat. Žádný z těchto atributů nesmí nabýt neznámé hodnoty, jinak je integrita naší databáze narušena.

### Řešení problému

Tento problém se řeší zakázáním neznámé hodnoty pro příslušné sloupce (Oracle, 2022g).

### Důsledky řešení

Jedná se o použití základní mechaniky systému databáze Oracle, alternativní řešení zde žádné není.

### Příklad kódu

Vytvoření tabulky zaměstnanců se zakázanými neznámými hodnotami pro příslušné sloupce:

```sql
CREATE TABLE employees (
    employee_id   NUMBER
        GENERATED ALWAYS AS IDENTITY,
    birthday      DATE NOT NULL,
    boarding_date DATE NOT NULL,
    created_at    TIMESTAMP NOT NULL,
    salary        NUMBER NOT NULL,
    phone_number  CHAR(9),
    PRIMARY KEY ( employee_id )
);
```

## Více tabulková kontrolní omezení

### Popis problému

Kontrolní omezení v Oracle databázi může pracovat pouze se sloupci tabulky, na které je omezení vytvořeno. Oracle uvažuje o přidaní SQL tvrzení, které by tento problém řešily (Oracle, 2016b). SQL tvrzení by vlastně bylo kontrolní omezení na databázové úrovní a v tomto kontrolním omezení by byly povoleny SQL dotazy. Díky tomu by bylo možné se doptat na data z jiných tabulek, které potřebujeme k vytvoření našeho omezení. Ovšem SQL tvrzení zatím přidána nejsou, a tak se tento problém musí řešit oklikou.

### Příklad problému

Předpokládejme, že máme tabulku smluv a tabulku účastníků smluv, která obsahuje procentuální hodnotu účastníka na smlouvě. Potřebujeme zajistit, aby jedna smlouva neměla celkem více jak sto procent účasti.

### Řešení problému

Řešení tohoto problému je použití triggeru (Oracle, 2011). Je zde důležité se zamyslet nad důsledky vytvořeného triggeru, zvláště co se týče výkonu naší databáze, jelikož triggery mohou mít zásadní vliv na rychlost prováděných transakcí. Pokud se jedná o trigger, který se spouští nad každým řádkem a je v tabulce s častými a velkými změnami, tak zpomalení databáze může být velmi výrazné.

Pro náš případ do tabulky smluv přidáme odložené kontrolní omezení, které se spustí až při provedení změn v tabulce. Dále pak vytvoříme trigger na tabulku s účastníky smlouvy, který když se přidá, odebere nebo změní hodnota procentuální účasti na smlouvě, tak se přepíše celková účast u smlouvy v tabulce smluv, pokud daná hodnota přesáhne sto procent, tak naše odložené kontrolní omezení při provedení změn vyhodí chybu a nenechá nás tyto hodnoty změnit a dané změny vrátí (Oracle, 2011).

### Důsledky řešení

Toto řešení může přinést zpomalení naší databáze, jelikož vytváříme trigger, který se spouští pro každý řádek daného vložení či aktualizace, v případě, že se jedná o tabulku s častým vkládáním či aktualizací mnoha řádků, může nastat zpomalení naší databáze.

Alternativní řešení by bylo použití uložených procedur a balíčku. Toto alternativní řešení by fungovalo tak, že naprogramujeme balíček, který bude obsahovat procedury, které nám zajistí naši definovanou podmínku. V tomto případě specificky, by procedury aktualizovaly procenta účasti na smlouvě a tyto procedury by se používaly při vkládání, aktualizování a vymazávání dat z tabulky účastníků smlouvy. Dále naši podmínku zajistí stejně jako při použití triggeru odložené kontrolní omezení, které při provedení změn zkontroluje, zda hodnota procentuální účasti u smlouvy nepřesahuje sto procent.

### Příklad kódu

Vytvoření tabulky smluv s odloženým kontrolním omezením:

```sql
CREATE TABLE agreements (
    agreement_id   NUMBER NOT NULL,
    agreement_name VARCHAR2(10) NOT NULL,
    ppct           NUMBER DEFAULT 0
        CONSTRAINT ppct_must_be_100 CHECK ( ppct = 100 ) DEFERRABLE INITIALLY DEFERRED,
    PRIMARY KEY ( agreement_id )
);
```

Vytvoření tabulky s účastníky smluv:

```sql
CREATE TABLE participant (
    participant_name  NUMBER NOT NULL,
    agreement_id      NUMBER NOT NULL,
    participation_pct NUMBER NOT NULL,
    PRIMARY KEY ( participant_name,
                  agreement_id ),
    FOREIGN KEY ( agreement_id )
        REFERENCES agreements
);
```

Vytvoření triggeru nad tabulkou s účastníky smluv:

```sql
CREATE OR REPLACE TRIGGER participant_trigger AFTER
    INSERT OR UPDATE OR DELETE ON participant
    FOR EACH ROW
BEGIN
    IF ( inserting OR updating ) THEN
        UPDATE agreements
        SET
            ppct = nvl(ppct, 0) + :new.participation_pct
        WHERE
            agreement_id = :new.agreement_id;

    END IF;

    IF ( updating OR deleting ) THEN
        UPDATE agreements
        SET
            ppct = nvl(ppct, 0) - :old.participation_pct
        WHERE
            agreement_id = :old.agreement_id;
    END IF;
END; 
```

Alternativní řešení, vytvoření balíčku s uloženými procedurami:

```sql
CREATE OR REPLACE PACKAGE pkg_agreements_pct_check IS
    PROCEDURE participant_insert (
        participant_name         NUMBER,
        participant_agreement_id NUMBER,
        participation_pct        NUMBER
    );

    PROCEDURE participant_update (
        participant_name         NUMBER,
        participant_agreement_id NUMBER,
        participation_pct        NUMBER
    );

    PROCEDURE participant_delete (
        participant_name NUMBER
    );

END pkg_agreements_pct_check;

CREATE OR REPLACE PACKAGE BODY pkg_agreements_pct_check IS

    PROCEDURE participant_insert (
        participant_name         NUMBER,
        participant_agreement_id NUMBER,
        participation_pct        NUMBER
    ) IS
    BEGIN
        INSERT INTO participant VALUES (
            participant_name,
            participant_agreement_id,
            participation_pct
        );

        UPDATE agreements
        SET
            ppct = nvl(ppct, 0) + participation_pct
        WHERE
            agreement_id = participant_agreement_id;

    END;

    PROCEDURE participant_update (
        participant_name         NUMBER,
        participant_agreement_id NUMBER,
        participation_pct        NUMBER
    ) IS
    BEGIN
        UPDATE participant
        SET
            agreement_id = participant_agreement_id,
            participation_pct = participation_pct
        WHERE
            participant_name = participant_name;

        IF ( participation_pct > 0 ) THEN
            UPDATE agreements
            SET
                ppct = nvl(ppct, 0) + participation_pct
            WHERE
                agreement_id = participant_agreement_id;

        ELSE
            UPDATE agreements
            SET
                ppct = nvl(ppct, 0) - participation_pct
            WHERE
                agreement_id = participant_agreement_id;

        END IF;

    END;

    PROCEDURE participant_delete (
        participant_name NUMBER
    ) IS
        participation_pct NUMBER;
        agreement_id      NUMBER;
    BEGIN
        SELECT
            participation_pct
        INTO participation_pct
        FROM
            participant
        WHERE
            participant_name = participant_name;

        SELECT
            agreement_id
        INTO agreement_id
        FROM
            participant
        WHERE
            participant_name = participant_name;

        DELETE FROM participant
        WHERE
            participant_name = participant_name;

        UPDATE agreements
        SET
            ppct = nvl(ppct, 0) - participation_pct
        WHERE
            agreement_id = agreement_id;

    END;

END pkg_agreements_pct_check;
```

## Problém mutující tabulky

### Popis problému

Tento problém vzniká kvůli Oracle mechanismu k zajištění konzistence při čtení proti takzvaným „špinavým čtením“, což je čtení neprovedených změn (Oracle, 2022f). V Oracle databázi nejsou normálně neprovedené změny vidět mimo relaci ve které změny probíhají. Tento problém se nám objeví, pokud máme trigger v databázi, který je vyvolán vložením, aktualizací či vymazáním dat v tabulce a v té samé tabulce se daný trigger snaží provést změny. V tuto chvíli upravující relace vidí své vlastní změny a Oracle se pokusí daný trigger spustit, ale ten nám selže, jelikož chce upravit nekompletně pozměněná data. Tento trigger je tedy neúspěšný, jelikož chce udělat změny v průběhu změn probíhající transakce. Oracle toto chování nedovoluje a vyhodí nám chybu.

### Příklad problému

Máme databázi zaměstnanců a chceme přidat kontrolu, zda navýšení platu zaměstnance nepřesahuje limit, který si nastavíme a je závislý na dalších údajích z naší databáze, například dvanáct procent z průměru platů všech zaměstnanců ze stejného oddělení jako je zaměstnanec, u kterého se plat navyšuje.

### Řešení problému

Řešení tohoto problému může být provedeno dvěma způsoby. První, který by měl být používán přednostně, lze použít pouze pokud používáme databázi Oracle 11g nebo novější, což by měla být samozřejmost, jelikož nižší verze jsou už zastaralé. Toto řešení využívá funkce compound trigger (Oracle, 2022a). Tento trigger funguje trochu jinak než normální trigger. Compound trigger může být vyvolány ve více časových bodech oproti normálnímu triggeru, který může být vyvolán pouze jednou. Díky tomu se můžeme dostat k tabulce předtím, než naše transakce začne data měnit a žádná chyba nám tak nevznikne, jelikož se nesnažíme číst žádné neprovedené změny.

Druhý způsob, v případě, že máme starší verzi databáze Oracle, tak je zde možnost vyřešit tento problém pomocí naprogramování uložených procedur a balíčků (Oracle, 2022i). V balíčku vytvoříme procedury k vyřešení našeho problémů a ty pak použijeme ve dvou triggerech, kdy v prvním před tím, než proběhne aktualizace dat, si uložíme průměrné platy pro všechny oddělení a ve druhém který probíhá po aktualizaci dat, pro každý plat zkontrolujeme, zda vyhovuje naší podmínce.

V našem případě vytvoříme compound trigger, který předtím, než transakce začne měnit data, si uloží průměrné hodnoty platu všech zaměstnanců podle jednotlivých oddělení a pak pro každý řádek zkontroluje, zda navýšení platu nepřesáhlo hodnotu dvanácti procent z průměrného platu v daném oddělení (Fitzjarrell, 2016).

### Důsledky řešení

Toto řešení může přinést zpomalení naší databáze, jelikož vytváříme trigger, který se spouští pro každý řádek daného vložení či aktualizace, v případě, že se jedná o tabulku s častým vkládáním či aktualizací mnoha řádků, může nastat zpomalení naší databáze.

### Příklad kódu

Vytvoření compound triggeru:

```sql
CREATE OR REPLACE TRIGGER check_raise_on_avg FOR
    UPDATE OF sal ON emp
COMPOUND TRIGGER
    twelve_percent          CONSTANT NUMBER := 0.12;
  
      -- Declare collection type and variable:

    TYPE department_salaries_t IS
        TABLE OF emp.sal%TYPE INDEX BY VARCHAR2(80);
    department_avg_salaries department_salaries_t;
    TYPE sal_t IS
        TABLE OF emp.sal%TYPE;
    avg_salaries            sal_t;
    TYPE deptno_t IS
        TABLE OF emp.deptno%TYPE;
    department_ids          deptno_t;
    BEFORE STATEMENT IS BEGIN
        SELECT
            AVG(e.sal),
            nvl(e.deptno, - 1)
        BULK COLLECT
        INTO
            avg_salaries,
            department_ids
        FROM
            emp e
        GROUP BY
            e.deptno;
        FOR j IN 1..department_ids.count() LOOP
            department_avg_salaries(department_ids(j)) := avg_salaries(j);
        END LOOP;
    END BEFORE STATEMENT;
    AFTER EACH ROW IS BEGIN
        IF :new.sal - :old.sal > twelve_percent * department_avg_salaries(:new.deptno) THEN
            raise_application_error(-20000, 'Raise too large');
        END IF;
    END AFTER EACH ROW;
END check_raise_on_avg;
```

Vytvoření balíčku s procedurami a přidání triggerů do tabulky zaměstnanců:

```sql
CREATE OR REPLACE PACKAGE pkg_check_raise_on_avg IS
    twelve_percent CONSTANT NUMBER := 0.12;
    TYPE department_salaries_t IS
        TABLE OF emp.sal%TYPE INDEX BY VARCHAR2(80);
    department_avg_salaries department_salaries_t;
    TYPE sal_t IS
        TABLE OF emp.sal%TYPE;
    avg_salaries sal_t;
    TYPE deptno_t IS
        TABLE OF emp.deptno%TYPE;
    department_ids deptno_t;
    PROCEDURE save_data;

    PROCEDURE check_sal (
        new_salary NUMBER,
        old_salary NUMBER,
        deptno     NUMBER
    );

END pkg_check_raise_on_avg;

CREATE OR REPLACE PACKAGE BODY pkg_check_raise_on_avg IS

    PROCEDURE save_data IS
    BEGIN
        SELECT
            AVG(e.sal),
            nvl(e.deptno, - 1)
        BULK COLLECT
        INTO
            avg_salaries,
            department_ids
        FROM
            emp e
        GROUP BY
            e.deptno;

        FOR j IN 1..department_ids.count() LOOP
            department_avg_salaries(department_ids(j)) := avg_salaries(j);
        END LOOP;

    END;

    PROCEDURE check_sal (
        new_salary NUMBER,
        old_salary NUMBER,
        deptno     NUMBER
    ) IS
    BEGIN
        IF new_salary - old_salary > twelve_percent * department_avg_salaries(deptno) THEN
            raise_application_error(-20000, 'Raise too large');
        END IF;
    END;

END pkg_check_raise_on_avg;

CREATE OR REPLACE TRIGGER save_before_check BEFORE
    UPDATE OF sal ON emp
BEGIN
    pkg_check_raise_on_avg.save_data;
END save_before_check;

CREATE OR REPLACE TRIGGER check_each_row AFTER
    UPDATE OF sal ON emp
    FOR EACH ROW
BEGIN
    pkg_check_raise_on_avg.check_sal(:new.sal, :old.sal, :new.deptno);
END check_each_row;
```

## Zdroje

FITZJARRELL, David, 2016. Mutating Table Error In Oracle: Why It Happens And What You Can Do About It. _Database Journal_ \[online] \[vid. 2022-05-07]. Dostupné z: https://www.databasejournal.com/oracle/mutating-table-error-in-oracle-why-it-happens-and-what-you-can-do-about-it/

LOGICALMIND, 2008. Answer to „What’s the best practice for primary keys in tables?" In: _Stack Overflow_ \[online]. \[vid. 2022-05-07]. Dostupné z: https://stackoverflow.com/a/338424/8096705

MITTY, Walter, 2011. Answer to „ORACLE Table design: M:N table best practice". In: _Stack Overflow_ \[online]. \[vid. 2022-05-07]. Dostupné z: https://stackoverflow.com/a/7448488/8096705

ORACLE, 2011. _Best way to enforce cross-row constraints? - Ask TOM_ \[online] \[vid. 2022-05-07]. Dostupné z: https://asktom.oracle.com/pls/apex/f?p=100:11:0::::p11\_question\_id:4233459000346171405

ORACLE, 2016a. _CALCULATION OF AGE - Ask TOM_ \[online] \[vid. 2022-05-07]. Dostupné z: https://asktom.oracle.com/pls/apex/f?p=100:11:::NO:RP:P11\_QUESTION\_ID:9531934000346471628

ORACLE, 2022a. Compound DML Triggers. _Oracle Help Center_ \[online]. B.m.: December2021 \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/lnpls/plsql-triggers.html#GUID-8A0DA171-BE6A-4798-A1A4-677B88EA16A0

ORACLE, 2022b. constraint. _Oracle Help Center_ \[online]. B.m.: March2022 \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/constraint.html#GUID-1055EA97-BA6F-4764-A15F-1024FD5B6DFE

ORACLE, 2022c. Data Integrity. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-A8893CD7-8B19-42AA-8550-9713071FA679

ORACLE, 2022d. Foreign Key Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-7CD73D16-EA1A-4AA8-AA7D-4288557395B8

ORACLE, 2022e. Check Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-5AF9C206-0139-4506-96DE-F6AD1D41CD41

ORACLE, 2022f. Mutating-Table Restriction. _Oracle Help Center_ \[online]. B.m.: March2022 \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/lnpls/plsql-triggers.html#GUID-73B70893-9E45-4C08-B327-13ECBE4BE920

ORACLE, 2022g. NOT NULL Integrity Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-CF2E06A6-6A35-46CE-808E-305A459457CC

ORACLE, 2022h. Oracle Built-in Data Types. _Oracle Help Center_ \[online]. B.m.: March2022 \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Data-Types.html#GUID-7B72E154-677A-4342-A1EA-C74C1EA928E6

ORACLE, 2022i. _Overview of PL/SQL_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/server-side-programming.html#GUID-1E17CED5-73C6-4C10-85F1-A2CB4D5F9855

ORACLE, 2022j. Primary Key Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-E1033BB9-0F67-4E59-82AC-B8B572FD82BB

ORACLE, 2022k. Reasons to Use Triggers. _Oracle Help Center_ \[online]. B.m.: February2022 \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/lnpls/plsql-triggers.html#GUID-C08EA160-8FD2-4A10-9733-6F2D20C83E93

ORACLE, 2016b. SQL Assertions / Declarative multi-row constraints. _oracle-tech_ \[online] \[vid. 2022-05-07]. Dostupné z: https://community.oracle.com/tech/apps-infra/discussion/4390732/sql-assertions-declarative-multi-row-constraints

ORACLE, 2022l. Types of Integrity Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-1C9665AD-A444-4AFB-984F-6385FCBEA64E

ORACLE, 2022m. Unique Constraints. _Oracle Help Center_ \[online] \[vid. 2022-05-07]. Dostupné z: https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-integrity.html#GUID-077C26A1-49C3-4E72-AE1D-7CEDD997917A

ORACLE, 2022n. _What is a relational database?_ \[online] \[vid. 2022-05-07]. Dostupné z: https://www.oracle.com/database/what-is-a-relational-database/
