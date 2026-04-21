🛠️ Data Tools — merge_pma_shopify2.py

Applicazione Streamlit con due strumenti per la gestione e pulizia di dati CSV relativi a campagne pubblicitarie e vendite e-commerce.

Tab 1 — 🔄 Shopify & PMA Merger
Risolve le discrepanze settimanali tra il report pubblicitario aggregato (Meta + Google Ads, detto "PMA") e i dati di vendita reali esportati da Shopify.

Come funziona:

Si caricano due CSV: il report PMA (META_GOOGLE_SHOPIFY) e l'export vendite di Shopify.
Lo script crea una chiave ISO settimanale (anno + numero settimana) per allineare i dati tra le due fonti.
Le metriche di Shopify (Vendite totali, Fatturato netto/lordo, Ordini, Resi, Sconti) sovrascrivono i valori corrispondenti nel PMA.
Le celle modificate vengono evidenziate in rosso nell'anteprima dell'applicazione.
Il file corretto è scaricabile come META_GOOGLE_SHOPIFY_CORRETTO.csv.
Tab 2 — 📂 Generic CSV Merger
Strumento generico per unire più file CSV con la stessa struttura in un unico file finale.

Come funziona:

Si caricano diversi file CSV contemporaneamente.
Vengono concatenati in un unico set di dati (con una colonna opzionale Source_File che traccia l'origine di ogni riga).
Il file unito è scaricabile come merged_data.csv.
Stack: Python · Streamlit · Pandas
