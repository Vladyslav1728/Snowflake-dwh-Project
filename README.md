### **Selecting a database**
Federal Exchange Rates 
https://app.snowflake.com/marketplace/listing/GZTYZAPS3FL/insights-federal-exchange-rates
Táto databáza poskytuje informácie o výmenných kurzoch mien a kryptomien.

- prečo ste si vybrali daný dataset :
Vybrali sme si ho, pretože obsahuje niekoľko tabuliek, má jasné názvy a všetky údaje sú už vyplnené na overenie výsledkov.
Práca s jasnými a zrozumiteľnými ekonomickými údajmi vyzerala atraktívne.
- aký biznis proces vaše dáta podporujú :
Naše dáta podporujú analýzu a rozhodovanie v oblasti finančných trhov a obchodovania s menami a kryptomenami.
Vhodné na predpovedanie kurzov, hodnotenie globálnej kurzovej situácie, výber ekonomickej stratégie a ďalšie obchodné rozhodnutia.
- aké typy údajov obsahuje :
```sql
DESCRIBE TABLE FACT_FOREX;
DESCRIBE TABLE DIM_STATUS;
DESCRIBE TABLE DIM_COUNTRY;
DESCRIBE TABLE DIM_DATE;
DESCRIBE TABLE DIM_CURRENCY;
```
NUMBER
BOOLEAN
VARCHAR
DATE
Číselné údaje : OPEN, CLOSE, HIGH, LOW, VOLUME, CLOSE_DIFF, NEXT_CLOSE_DIFF, AVG_LAST_7_DAYS
Textové údaje : CURRENCY, DESCRIPTION, COUNTRY_NAME, STATUS_NAME, DOW_NAME, SEASON
Boolean : TOP5, G21, G7, STATUS_FLAG
Dátumové údaje : DATE, PRICE_DATE, YEAR, MONTH, DAY, QUARTER
- na čo bude analýza zameraná :
Pre podrobnú analýzu výmenného kurzu sa pozrieme na peňažné meny; tabuľka faktov sa zameria na ne a na krajiny, nie na kryptomeny.

<p align="center">
  <img src="https://github.com/Vladyslav1728/Snowflake-dwh-Project/blob/main/img/raw_schema.png" alt="ERD">
  <br>
  <em>Ako pôvodne vyzerala databáza</em>
</p>

---
### **Preparing**
V Snowflake bola vytvorená verejná databáza : SCORPION_AND_SEAL_PROJECT_DB
Bola vytvorena schema : labSKUSKA
```sql
CREATE DATABASE SCORPION_AND_SEAL_PROJECT_DB;
USE DATABASE SCORPION_AND_SEAL_PROJECT_DB;
CREATE OR REPLACE SCHEMA labSKUSKA;
USE SCHEMA labSKUSKA;
```
---
### **STG tables**
Začalo sa vytváranie tabuliek STG s niekoľkými úpravami pre pohodlnejšiu prácu.
- Pri kopírovaní dát zo zdrojových tabuliek som premenovali všetky názvy mien na skratky (napr. ARGENTINE_PESO AS ARS), aby bolo možné neskôr pracovať s týmito názvami v DIM_CURRENCY.
- Odstraňovali som nepotrebné stĺpce, ktoré nie sú potrebné pre ďalšie spracovanie.
- Všetky dátumy som premenili na jednotný formát pomocou: CAST(TIMESTAMP AS DATE) AS PRICE_DATE

<p align="center">
  <img src="https://github.com/Vladyslav1728/Snowflake-dwh-Project/blob/main/img/erd_schema.png" alt="ERD">
  <br>
  <em>ERD stage</em>
</p>

STG_CRYPTO_CURRENCY_HISTORICAL_DATA - Zobrazuje dátum, kryptomenu, názov meny a kurz, pri ktorom sa otvorila na burze, kurz, pri ktorom sa uzavrela, minimálnu hodnotu počas dňa a maximálnu hodnotu.

STG_FOREIGN_EXCHANGE_RATE - Zobrazuje dátum a výmenný kurz každej svetovej meny voči doláru (každá mena v samostatnom stĺpci)

STG_FOREIGN_EXCHANGE_RATES_TRENDS - Zobrazuje dátum, krajinu, menu, názov meny a kurz, pri ktorom sa otvorila na burze, kurz, pri ktorom sa uzavrela, minimálnu hodnotu počas dňa a maximálnu hodnotu.

STG_G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE - Stĺpce mien krajín zahrnutých do Global 21 a dátum a kurz pri uzávierke

STG_G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES - Stĺpce mien krajín zahrnutých do Global 7 a dátum a kurz pri uzávierke

STG_TOP_100_MARKET_CAP_CRYPTO - Cena a názvy 100 najlepších kryptomien, dátumy zatvorenia a otvorenia na burze a denné maximá a minimá

STG_TOP_FIVE_TRADABLE_CURRENCIES - 5 najobchodovanejších mien, dátum a cena na burze

```sql
CREATE OR REPLACE TABLE STG_CRYPTO_CURRENCY_HISTORICAL_DATA AS
SELECT
    CURRENCY,
    BASE,
    CAST(TIMESTAMP AS DATE) AS PRICE_DATE,
    OPEN,
    CLOSE,
    HIGH,
    LOW,
    VOLUME
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.CRYPTO_CURRENCY_HISTORICAL_DATA;

CREATE OR REPLACE TABLE STG_FOREIGN_EXCHANGE_RATE AS
SELECT *
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.FOREIGN_EXCHANGE_RATE
WHERE TIME IS NOT NULL;

ALTER TABLE STG_FOREIGN_EXCHANGE_RATE DROP COLUMN S_NO;

CREATE OR REPLACE TABLE STG_FOREIGN_EXCHANGE_RATES_TRENDS AS
SELECT *
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.FOREIGN_EXCHANGE_RATES_TRENDS;

CREATE OR REPLACE TABLE STG_G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE AS
SELECT
    ARGENTINE_PESO              AS ARS,
    AUSTRALIAN_DOLLAR           AS AUD,
    BRAZIL_REAL                 AS BRL,
    CANADIAN_DOLLAR             AS CAD,
    CENTRAL_AFRICAN_CFA_FRANC   AS XAF,
    CHINA_YUAN                  AS CNY,
    EURO_AREA_EURO              AS EUR,
    INDIAN_RUPEE                AS INR,
    INDONESIAN_RUPIAH           AS IDR,
    JAPAN_YEN                   AS JPY,
    KOREAN_WON                  AS KRW,
    MEXICAN_PESO                AS MXN,
    RUSSIAN_RUBLE               AS RUB,
    SAUDI_RIYAL                 AS SAR,
    SOUTH_AFRICA_RAND           AS ZAR,
    TURKISH_LIRA                AS TRY,
    UNITED_KINGDOM_POUND        AS GBP,
    DATE
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE;

CREATE OR REPLACE TABLE STG_G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES AS
SELECT
    CANADIAN_DOLLAR AS CAD,
    EURO_AREA_EURO AS EUR,
    JAPAN_YEN AS JPY,
    RUSSIAN_RUBLE AS RUB,
    UNITED_KINGDOM_POUND AS GBP,
    DATE
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES;

CREATE OR REPLACE TABLE STG_TOP_100_MARKET_CAP_CRYPTO AS
SELECT
    CURRENCY,
    BASE,
    CAST(TIMESTAMP AS DATE) AS PRICE_DATE,
    OPEN,
    HIGH,
    LOW,
    CLOSE,
    VOLUME
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.TOP_100_MARKET_CAP_CRYPTO_CURRENCY_HISTORICAL_DATA;

CREATE OR REPLACE TABLE STG_TOP_FIVE_TRADABLE_CURRENCIES AS
SELECT
    CANADIAN_DOLLAR AS CAD,
    EURO_AREA_EURO AS EUR,
    JAPAN_YEN AS JPY,
    UNITED_KINGDOM_POUND AS GBP,
    WITZERLAND_FRANC AS CHF,
    DATE
FROM FEDERAL_EXCHANGE_RATES.INSIGHTS.TOP_FIVE_TRADABLE_CURRENCIES_FOREIGN_EXCHANGE_RATE;
```
Následne sme skontrolovali správnosť vytvorenia.
```sql
SELECT
    CURRENCY,
    BASE,
    PRICE_DATE,
    ROUND(OPEN, 8) AS OPEN,
    ROUND(HIGH, 8) AS HIGH,
    ROUND(LOW, 8) AS LOW,
    ROUND(CLOSE, 8) AS CLOSE,
    ROUND(VOLUME, 8) AS VOLUME
FROM STG_CRYPTO_CURRENCY_HISTORICAL_DATA;
SELECT * FROM STG_FOREIGN_EXCHANGE_RATE ORDER BY TIME DESC;
SELECT * FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS;
SELECT * FROM STG_G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE;
SELECT * FROM STG_G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES;
SELECT * FROM STG_TOP_100_MARKET_CAP_CRYPTO;
SELECT * FROM STG_TOP_FIVE_TRADABLE_CURRENCIES;
```

## **DIM tables**
DIM_DATE :
Tabuľka na poskytovanie informácií o čase a dátume
Berieme údaje z poddotazu, ktorý získava dáta z STG_FOREIGN_EXCHANGE_RATES_TRENDS. Odstránia sa nulové a duplicitné hodnoty.
DATE_ID – cez window funkciu ROW_NUMBER() sa nastaví unikátny identifikátor.
DATE – berieme priamo hodnotu dátumu zo staging tabuľky.
DAY – cez DATE_PART() získame iba deň z dátumu.
DOW – číslovanie dňa v týždni (americký štýl +1), začína od nedele.
DOW_NAME – cez CASE názov dňa, napríklad 1 = ‘Monday’.
MONTH – cez DATE_PART() získame mesiac z dátumu.
YEAR – cez DATE_PART() získame rok z dátumu.
QUARTER – rozdelenie roka na 4 časti, pre rýchle získanie údajov (nie úplne presné).
SEASON – cez CASE názov ročného obdobia podľa dátumu.
```sql
CREATE OR REPLACE TABLE DIM_DATE AS
SELECT
    ROW_NUMBER() OVER (ORDER BY DATE) AS DATE_ID,
    DATE AS DATE,
    DATE_PART(day, DATE) AS DAY,
    DATE_PART(dow, DATE) + 1 AS DOW,
    CASE DATE_PART(dow, DATE) + 1
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
        WHEN 7 THEN 'Sunday'
    END AS DOW_NAME,
    DATE_PART(month, DATE) AS MONTH,
    DATE_PART(year, DATE) AS YEAR,
    DATE_PART(quarter, DATE) AS QUARTER
FROM (
    SELECT DISTINCT DATE
    FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS
    WHERE DATE IS NOT NULL
) AS unique_dates
ORDER BY DATE;
```

DIM_CURRENCY :
Tabuľka na získavanie informácií o mene
Berieme hodnoty zo staging tabuľky. Vytvárame 3 poddotazy: TOP5, G21, G7, ktoré čerpajú údaje z tabuliek STG_TOP5, STG_G7, STG_G21.
V týchto poddotazoch sa ukladajú názvy mien z INFORMATION_SCHEMA.COLUMNS, pretože potrebujeme zistiť, či mena patrí do TOP5, G7 alebo G21.
Tiež berieme dáta z poddotazu, ktorý čerpá z STG_FOREIGN_EXCHANGE_RATES_TRENDS a odstraňuje duplicitné a prázdne hodnoty.
CURRENCY_ID – cez window funkciu ROW_NUMBER() sa nastaví unikátny identifikátor.
f.CURRENCY – samotná mena z poddotazu.
f.DESCRIPTION – popis meny.
TOP5 – boolean flag, cez CASE kontrolujeme, či je f.CURRENCY v TOP5.
G21 – boolean flag, cez CASE kontrolujeme, či je f.CURRENCY v G21.
G7 – boolean flag, cez CASE kontrolujeme, či je f.CURRENCY v G7.
```sql
CREATE OR REPLACE TABLE DIM_CURRENCY AS
WITH top5 AS (
    SELECT COLUMN_NAME AS CURRENCY
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_NAME = 'STG_TOP_FIVE_TRADABLE_CURRENCIES'
      AND COLUMN_NAME <> 'DATE'
),
g21 AS (
    SELECT COLUMN_NAME AS CURRENCY
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_NAME = 'STG_G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE'
      AND COLUMN_NAME <> 'DATE'
),
g7 AS (
    SELECT COLUMN_NAME AS CURRENCY
    FROM INFORMATION_SCHEMA.COLUMNS
    WHERE TABLE_NAME = 'STG_G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES'
      AND COLUMN_NAME <> 'DATE'
)
SELECT
    ROW_NUMBER() OVER (ORDER BY f.CURRENCY) AS CURRENCY_ID,
    f.CURRENCY,
    f.DESCRIPTION,
    CASE WHEN f.CURRENCY IN (SELECT CURRENCY FROM top5) THEN TRUE ELSE FALSE END AS TOP5,
    CASE WHEN f.CURRENCY IN (SELECT CURRENCY FROM g21) THEN TRUE ELSE FALSE END AS G21,
    CASE WHEN f.CURRENCY IN (SELECT CURRENCY FROM g7) THEN TRUE ELSE FALSE END AS G7
FROM (
    SELECT DISTINCT CURRENCY, DESCRIPTION
    FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS
    WHERE CURRENCY IS NOT NULL
) AS f
ORDER BY f.CURRENCY;
```

DIM_STATUS :
Tabuľka na získavanie informácií o stave (DIM_STATUS)
Berieme údaje z STG_FOREIGN_EXCHANGE_RATES_TRENDS cez poddotaz, ktorý odstráni duplicitné hodnoty.
STATUS_ID – cez window funkciu ROW_NUMBER() sa nastaví unikátny identifikátor pre každý stav.
STATUS_FLAG – priamy boolean z pôvodnej tabuľky (ISACTIVE).
STATUS_NAME – cez CASE sa prekladá flag na názov:
TRUE = 'ACTIVE'
FALSE = 'NO ACTIVE'
NULL alebo iné = 'UNKNOWN'
```sql
CREATE OR REPLACE TABLE DIM_STATUS AS
SELECT
    ROW_NUMBER() OVER (ORDER BY ISACTIVE) AS STATUS_ID,
    ISACTIVE AS STATUS_FLAG,
    CASE
        WHEN ISACTIVE = TRUE  THEN 'ACTIVE'
        WHEN ISACTIVE = FALSE THEN 'NO ACTIVE'
        ELSE 'UNKNOWN'
    END AS STATUS_NAME
FROM (
    SELECT DISTINCT ISACTIVE
    FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS
) AS unique_status;
```

DIM_COUNTRY :
Tabuľka na získavanie informácií o krajinách (DIM_COUNTRY)
Berieme údaje z STG_FOREIGN_EXCHANGE_RATES_TRENDS cez poddotaz, ktorý odstráni duplicitné a prázdne hodnoty.
COUNTRY_ID – cez window funkciu ROW_NUMBER() sa nastaví unikátny identifikátor pre každú krajinu.
COUNTRY_NAME – samotný názov krajiny zo staging tabuľky.
```sql
CREATE OR REPLACE TABLE DIM_COUNTRY AS
SELECT
    ROW_NUMBER() OVER (ORDER BY COUNTRY) AS COUNTRY_ID,
    COUNTRY AS COUNTRY_NAME
FROM (
    SELECT DISTINCT COUNTRY
    FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS
    WHERE COUNTRY IS NOT NULL
) AS unique_countries;
```
Následne sme skontrolovali správnosť vytvorenia.
```sql
SELECT * FROM DIM_DATE;
SELECT * FROM DIM_CURRENCY;
SELECT * FROM DIM_COUNTRY;
SELECT * FROM DIM_STATUS;
```
---
### **FACT table**

<p align="center">
  <img src="https://github.com/Vladyslav1728/Snowflake-dwh-Project/blob/main/img/star_schema.png" alt="star">
  <br>
  <em>STAR schema</em>
</p>

FACT_FOREX :
Tabuľka pre ukladanie faktov o menách
Berieme údaje zo STG_FOREIGN_EXCHANGE_RATES_TRENDS cez poddotaz a spájame ich s dimenziami:
DIM_DATE -> dátum
DIM_CURRENCY -> mena
DIM_COUNTRY -> krajina
DIM_STATUS -> stav aktivity

FACT_ID – cez window funkciu ROW_NUMBER() sa nastaví unikátny identifikátor pre každý riadok v tabuľke.
DATE_ID – ID dátumu z DIM_DATE.
CURRENCY_ID – ID meny z DIM_CURRENCY.
COUNTRY_ID – ID krajiny z DIM_COUNTRY.
STATUS_ID – ID stavu z DIM_STATUS.
OPEN, CLOSE, HIGH, LOW – pôvodné hodnoty otvorenia, zatvorenia, maxima a minima z STG tabuľky.

Window fun:
CLOSE_DIFF – rozdiel medzi dnešnou a predchádzajúcou hodnotou zatvorenia pre rovnakú menu, počítané cez LAG().
NEXT_CLOSE_DIFF – rozdiel medzi dnešnou a nasledujúcou hodnotou zatvorenia pre rovnakú menu, počítané cez LEAD() (napomáha pri predikcii).
AVG_LAST_7_DAYS – priemerná hodnota zatvorenia za posledných 7 dní pre rovnakú menu a krajinu, počítané cez AVG() OVER() s PARTITION BY a ROWS BETWEEN 6 PRECEDING AND CURRENT ROW.

Použité LEFT JOINy zabezpečujú, že každá mena, krajina, dátum a stav zodpovedá svojej dimenzii.
WHERE podmienky zabezpečujú, že sa do FACT_FOREX uložia iba kompletné záznamy, kde CURRENCY, DATE a COUNTRY nie sú NULL.

```sql
CREATE OR REPLACE TABLE FACT_FOREX AS
SELECT
    ROW_NUMBER() OVER (ORDER BY f.DATE, f.CURRENCY, f.COUNTRY) AS FACT_ID,
    d_date.DATE_ID,
    d_currency.CURRENCY_ID,
    d_country.COUNTRY_ID,
    d_status.STATUS_ID,
    f.OPEN,
    f.CLOSE,
    f.HIGH,
    f.LOW,
    f.CLOSE - LAG(f.CLOSE) OVER (
        PARTITION BY f.CURRENCY
        ORDER BY f.DATE
    ) AS CLOSE_DIFF,
    f.CLOSE - LEAD(f.CLOSE) OVER (
        PARTITION BY f.CURRENCY
        ORDER BY f.DATE
    ) AS NEXT_CLOSE_DIFF,
    AVG(f.CLOSE) OVER (
        PARTITION BY f.CURRENCY, f.COUNTRY
        ORDER BY f.DATE
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS AVG_LAST_7_DAYS
FROM STG_FOREIGN_EXCHANGE_RATES_TRENDS f
LEFT JOIN DIM_DATE     d_date     ON f.DATE     = d_date.DATE
LEFT JOIN DIM_CURRENCY d_currency ON f.CURRENCY = d_currency.CURRENCY
LEFT JOIN DIM_COUNTRY  d_country  ON f.COUNTRY  = d_country.COUNTRY_NAME
LEFT JOIN DIM_STATUS   d_status   ON f.ISACTIVE = d_status.STATUS_FLAG
WHERE f.CURRENCY IS NOT NULL
  AND f.DATE IS NOT NULL
  AND f.COUNTRY IS NOT NULL;
```
Kontrola správnosti tabuľky faktov.
```sql
SELECT * FROM FACT_FOREX;
```
---
### **Deleting tables**
Po úspešnom vytvorení tabuliek faktov a tabuliek dim odstráňte tabuľky STG.
```sql
DROP TABLE IF EXISTS STG_CRYPTO_CURRENCY_HISTORICAL_DATA;
DROP TABLE IF EXISTS STG_FOREIGN_EXCHANGE_RATE;
DROP TABLE IF EXISTS STG_FOREIGN_EXCHANGE_RATES_TRENDS;
DROP TABLE IF EXISTS STG_G21_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATE;
DROP TABLE IF EXISTS STG_G7_COUNTRY_CURRENCIES_FOREIGN_EXCHANGE_RATES;
DROP TABLE IF EXISTS STG_TOP_100_MARKET_CAP_CRYPTO;
DROP TABLE IF EXISTS STG_TOP_FIVE_TRADABLE_CURRENCIES;
```
---
## **Vizualizácia dát**
Dashboard obsahuje vizualizácie, ktoré ukazujú hlavné trendy a správanie vybraných mien v rôznych obdobiach a sezónach. Tieto grafy pomáhajú lepšie pochopiť kolísanie kurzov a sezónne výkyvy.

<p align="center">
  <img src="https://github.com/Vladyslav1728/Snowflake-dwh-Project/blob/main/img/SQL_Visualization.png" alt="Dashboard ">
  <br>
  <em>Dashboard na vizualizáciu požiadaviek</em>
</p>

---
### **Graf 1: Uzatváranie eura v pondelok (jeseň 2008)**
Táto vizualizácia zobrazuje výmenný kurz eura voči doláru pre každý pondelok na jeseň roku 2008. Číslovanie pondelkov uľahčuje sledovanie zmien výmenného kurzu počas celej sezóny. Údaje nám pomáhajú pochopiť, ako výmenný kurz kolísal počas tohto jesenného týždňa.

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY d.DATE) AS MONDAY_NUMBER,
    d.DOW_NAME,
    f.CLOSE,
    d.DATE
FROM FACT_FOREX f
JOIN DIM_DATE d
    ON f.DATE_ID = d.DATE_ID
WHERE f.CURRENCY_ID = 31
  AND d.YEAR = 2008
  AND d.DOW = 2
  AND d.SEASON = 'Autumn'
ORDER BY d.DATE;
```
---
### **Graf 2: Rast hrivny voči doláru (2006 – 2024)**

Táto vizualizácia zobrazuje zmenu výmenného kurzu ukrajinskej hrivny voči americkému doláru od roku 2006 do roku 2024. Graf zobrazuje maximálny výmenný kurz pre každý rok, pričom jasne demonštruje vrcholy zhodnocovania a znehodnocovania hrivny v priebehu času.

```sql
SELECT
    c.COUNTRY_NAME,
    d.YEAR,
    MAX(f.CLOSE) AS MAX_CLOSE
FROM FACT_FOREX f
JOIN DIM_COUNTRY c
    ON f.COUNTRY_ID = c.COUNTRY_ID
JOIN DIM_DATE d
    ON f.DATE_ID = d.DATE_ID
WHERE f.CURRENCY_ID = 105
  AND d.YEAR BETWEEN 2005 AND 2024
GROUP BY c.COUNTRY_NAME, d.YEAR
ORDER BY c.COUNTRY_NAME, d.YEAR;
```
---
### **Graf 3: Rozsah výmenných kurzov pre Top 21 (nie Top 7/Top 5) pre rok 2024**
Tento graf zobrazuje rozsah kolísania výmenných kurzov v roku 2024 pre meny v prvej 21 hodnote, ale mimo prvej 7 a prvej 5. Hodnota sa vypočíta ako rozdiel medzi najvyšším a najnižším záverečným kurzom za rok, čo umožňuje vidieť volatilitu týchto mien v porovnaní s väčšími a stabilnejšími menami.

```sql
WITH middle_currencies AS (
    SELECT CURRENCY_ID, CURRENCY AS CURRENCY_NAME
    FROM DIM_CURRENCY
    WHERE G21 = TRUE
      AND G7 = FALSE
      AND TOP5 = FALSE
)
SELECT
    c.CURRENCY_NAME,
    MAX(f.CLOSE) - MIN(f.CLOSE) AS CLOSE_RANGE_2024
FROM FACT_FOREX f
JOIN DIM_DATE d
    ON f.DATE_ID = d.DATE_ID
JOIN middle_currencies c
    ON f.CURRENCY_ID = c.CURRENCY_ID
WHERE d.YEAR = 2024
GROUP BY c.CURRENCY_NAME
ORDER BY CLOSE_RANGE_2024 DESC;
```
---
### **Graf 4: Výmenný kurz v Arménsku pre rok 2025 (zima a leto)**
Táto vizualizácia zobrazuje minimálne a maximálne výmenné kurzy v Arménsku pre rok 2025 pre zimnú a letnú sezónu. Graf jasne ukazuje rozsah kolísania výmenných kurzov v rôznych ročných obdobiach, čo pomáha posúdiť sezónne zmeny meny a identifikovať obdobia najväčšej volatility.

```sql
SELECT 
    d.SEASON, 
    MIN(f.CLOSE) AS MIN_CURRENCY, 
    MAX(f.CLOSE) AS MAX_CURRENCY
FROM FACT_FOREX f
JOIN DIM_DATE d ON f.DATE_ID = d.DATE_ID
JOIN DIM_COUNTRY c ON f.COUNTRY_ID = c.COUNTRY_ID
WHERE c.COUNTRY_NAME = 'Armenia'
  AND d.YEAR = 2024
  AND d.SEASON IN ('Winter', 'Summer')
GROUP BY d.SEASON
ORDER BY d.SEASON;
```
---
### **Graf 5: min + max výmenné kurzy (BGN) medzi aktívnymi menami (obdobie 2020 – 2025).**
Tento graf zobrazuje extrémne hodnoty kurzu bulharského leva (BGN) voči základnej mene za vybrané obdobie od roku 2020 do roku 2025. Vo vizualizácii sú zvýraznené dva body: najnižšia a najvyššia hodnota meny. Zohľadňujú sa iba aktívne meny, čo umožňuje zamerať sa na aktuálne trhové údaje. Tieto informácie sú užitočné na analýzu volatility meny a identifikáciu kľúčových období maxím a minim.

```sql
WITH BGN_ACTIVE AS (
    SELECT 
        f.OPEN,
        d.DATE
    FROM FACT_FOREX f
    JOIN DIM_CURRENCY c
        ON f.CURRENCY_ID = c.CURRENCY_ID
    JOIN DIM_DATE d
        ON f.DATE_ID = d.DATE_ID
    WHERE c.CURRENCY = 'BGN'
      AND f.STATUS_ID = 2
      AND d.YEAR BETWEEN 2020 AND 2025
)
SELECT
    'MIN' AS TYPE,
    OPEN AS CURRENCY,
    DATE AS DATE_VALUE
FROM BGN_ACTIVE
QUALIFY OPEN = MIN(OPEN) OVER ()

UNION ALL

SELECT
    'MAX' AS TYPE,
    OPEN AS CURRENCY,
    DATE AS DATE_VALUE
FROM BGN_ACTIVE
QUALIFY OPEN = MAX(OPEN) OVER ();
```
---
### **Graf 6: Priemerná miera otvorenia eura podľa sezóny za rok 2024.**
Táto vizualizácia zobrazuje, ako sa menil priemerný otvárací kurz eura počas jednotlivých sezón v roku 2024. Údaje zahŕňajú iba aktívne meny a umožňujú jasné pochopenie sezónnych výkyvov, ako je napríklad zhodnocovanie alebo znehodnocovanie na jar, v lete, na jeseň a v zime.

```sql
SELECT
    d.SEASON,
    ROUND(AVG(f.OPEN), 2) AS AVG_OPEN
FROM FACT_FOREX f
JOIN DIM_CURRENCY c ON f.CURRENCY_ID = c.CURRENCY_ID
JOIN DIM_DATE d ON f.DATE_ID = d.DATE_ID
WHERE c.TOP5 = TRUE
  AND c.CURRENCY = 'EUR'
  AND d.YEAR = 2024
  AND f.STATUS_ID = 2
GROUP BY d.SEASON
ORDER BY d.SEASON;
```

---

**Autor:** Shcherbyna Vladyslav a Davyd Shapovalov