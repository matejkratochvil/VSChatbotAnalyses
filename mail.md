
Po byznysové stránce jsou konkurencí klasické chatboty, které jsou v dobré kvalitě zdarma a mají přehled o webu (bod 4.). Naší výhodou je znalost klienta, ten nicméně dokáže svoje požadavky sdělit na pár iterací klasickému chatbotovi. Z tohoto důvodu a kvůli ambicióznímu deadline leden 2026 bych byl pro MVP, které podrží náklady na tokeny nízko, dá nám představu o zájmu uživatelů, na základě čehož se lze rozhodnout o dalších krocích.

K jednotlivým bodům:
1. Dlouhodobá paměť: Aplikace nejspíš má nějakou svoji databázi, kam by se chatbot mohl ptát pomocí toolu. Konverzace samotné je v Azure možno ukládat do key-value databáze Cosmos DB. Je třeba zajistit možnost smazání dat kvůli GDPR. Pro účely zlepšování modelu by se hodilo mít anonymizaci a odlévání dat do bucketu, což je ale mimo MVP.
2. Dynamické zpracování scénářů: přijde mi důležité se držet pouze těchto scénářů kvůli jednoduchosti řešení a minimalizaci nákladů na tokeny.
4. Web search: používal bych to v MVP maximálně jen k plnění databáze (ony dny otevřených dveří nebo podmínky přijetí), pokud databázi už nedostaneme hotovou.
5. Kvalitní rozlišení CZ/SK bych v MVP bral jako nice to have.
7. Zpětná vazba od uživatelů: toto mi přijde zásadní rozumně nastavit, ať můžeme v budoucnu A/B testovat různé varianty modelů nebo prostě vyhodnocovat kvalitu. Je to ale zároveň tricky, když je uživatel nespokojený, zavře okno a nedá zpětnou vazbu. Když je spokojený, taky zavře okno a často nedá zpětnou vazbu. Líbilo by se mi tlačítko jako "poslat shrnutí na email", z čehož na první pohled nečiší snaha o získání zpětné vazby, ale když to uživatel udělá, je celkem jasné, že mu přišla konverzace užitečná.
9. LLM a spotřeba tokenů: toto si myslím může celý projekt byznysově efektivně pohřbít. Ono cachování LLM dotazů za mě je složité, když vstupy nebudou nikdy přesně stejné.
10. Analytika: logoval bych funnel typu návštěva - scénář (ad 2.) - flag úspěchu (ad 7.)

Odpověď na otázku dělat/nedělat za mě leží v těch, kdo to budou programovat. Termín leden 2026 vidím jako dost ambiciózní. Jestli to bude byznysově vycházet nebo ne, je za mě věcí investora, my se můžeme snažit zvýšit šanci na úspěch.
ad 1.: v MVP bych nějaké shrnutí klientova kontextu z databáze aplikace dal do promptu a ideálně se vyhnul toolům úplně. bude to jednodušší na vývoj a levnější na provoz.