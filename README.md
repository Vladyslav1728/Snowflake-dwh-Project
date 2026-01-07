## **Vizualizácia dát**
Dashboard obsahuje vizualizácie, ktoré ukazujú hlavné trendy a správanie vybraných mien v rôznych obdobiach a sezónach. Tieto grafy pomáhajú lepšie pochopiť kolísanie kurzov a sezónne výkyvy.

<p align="center">
  <img src="https://github.com/Vladyslav1728/Snowflake-dwh-Project/blob/main/img/SQL_Visualization.png" alt="Dashboard ">
  <br>
  <em>Obrázok 2 Schéma hviezdy pre AmazonBooks</em>
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