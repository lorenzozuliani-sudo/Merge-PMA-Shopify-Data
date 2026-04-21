import streamlit as st
import pandas as pd
import io

# -------------------------------
# CONFIGURAZIONE PAGINA
# -------------------------------
st.set_page_config(
    page_title="Data Tools: Fixer & Merger",
    layout="centered"
)

st.title("🛠️ Data Tools")

tab1, tab2 = st.tabs(["🔄 Shopify & PMA Merger", "📂 Generic CSV Merger"])

# -------------------------------
# TAB 1: SHOPIFY & PMA MERGER
# -------------------------------
with tab1:
    st.header("Shopify & PMA Data Merger")
    st.markdown("""
    Carica i **due file CSV** per correggere le discrepanze settimanali.  
    I valori **sovrascritti da Shopify** verranno **evidenziati in rosso**.
    """)

    # -------------------------------
    # FUNZIONE PULIZIA VALUTE
    # -------------------------------
    def clean_currency(val):
        if pd.isna(val):
            return 0.0
        s = str(val).replace('€', '').strip()
        if '.' in s and ',' in s:
            s = s.replace('.', '')
        s = s.replace(',', '.')
        try:
            return float(s)
        except:
            return 0.0

    # -------------------------------
    # UPLOAD FILE
    # -------------------------------
    col1, col2 = st.columns(2)

    with col1:
        pma_file = st.file_uploader(
            "1. Carica META_GOOGLE_SHOPIFY (PMA)",
            type="csv",
            key="pma_upl"
        )

    with col2:
        shopify_file = st.file_uploader(
            "2. Carica Vendite Totali Shopify",
            type="csv",
            key="shop_upl"
        )

    # -------------------------------
    # ELABORAZIONE
    # -------------------------------
    if pma_file and shopify_file:
        try:
            # Caricamento CSV
            df_pma = pd.read_csv(pma_file)
            df_shop = pd.read_csv(shopify_file)

            df_pma.columns = df_pma.columns.str.strip()
            df_shop.columns = df_shop.columns.str.strip()

            # 🔴 COPIA PMA ORIGINALE (per confronto)
            df_pma_original = df_pma.copy()

            # -------------------------------
            # CREAZIONE KEY SETTIMANALE
            # -------------------------------
            df_shop['Data_DT'] = pd.to_datetime(df_shop['Settimana'])
            df_shop['ISO_Y'] = df_shop['Data_DT'].dt.isocalendar().year
            df_shop['ISO_W'] = df_shop['Data_DT'].dt.isocalendar().week
            df_shop['Key'] = (
                df_shop['ISO_Y'].astype(str)
                + df_shop['ISO_W'].astype(str).str.zfill(2)
            )

            df_pma['Key'] = (
                df_pma['Year Week']
                .astype(str)
                .str.split('.')
                .str[0]
                .str.strip()
            )

            # -------------------------------
            # MAPPATURA SHOPIFY → PMA
            # -------------------------------
            mapping_data = {
                'Total sales': 'Vendite totali',
                'Net sales': 'Fatturato netto',
                'Gross sales': 'Fatturato lordo',
                'Orders': 'Ordini',
                'Returns': 'Resi',
                'Discounts': 'Sconti'
            }

            st.info("🔄 Sincronizzazione metriche Shopify in corso...")

            shop_keyed = df_shop.set_index('Key')

            for pma_col, shop_col in mapping_data.items():

                if shop_col not in df_shop.columns:
                    continue

                cleaned_series = shop_keyed[shop_col].apply(clean_currency)

                if pma_col not in df_pma.columns:
                    df_pma[pma_col] = 0.0

                df_pma[pma_col] = (
                    df_pma['Key']
                    .map(cleaned_series)
                    .fillna(0.0)
                )

            # -------------------------------
            # CONFRONTO E CELLE MODIFICATE
            # -------------------------------
            corrected_cells = set()

            for col in mapping_data.keys():
                if col in df_pma.columns and col in df_pma_original.columns:
                    before = df_pma_original[col].apply(clean_currency)
                    after = df_pma[col].apply(clean_currency)

                    diff_mask = before != after
                    for idx in df_pma.index[diff_mask]:
                        corrected_cells.add((idx, col))

            st.success(f"✅ Celle corrette: {len(corrected_cells)}")

            # -------------------------------
            # FORMATTAZIONE FINALE
            # -------------------------------
            df_final = df_pma.drop(columns=['Key'])

            cols_to_format = [
                'Amount Spent',
                'Cost',
                'Total sales',
                'Net sales',
                'Gross sales',
                'Returns',
                'Discounts'
            ]

            for col in cols_to_format:
                if col in df_final.columns:
                    df_final[col] = (
                        df_final[col]
                        .apply(clean_currency)
                        .apply(lambda x: f"€{x:.2f}")
                    )

            # -------------------------------
            # STYLING: ROSSO SE MODIFICATO
            # -------------------------------
            def highlight_changes(row):
                styles = []
                for col in df_final.columns:
                    if (row.name, col) in corrected_cells:
                        styles.append("background-color: #ffcccc")
                    else:
                        styles.append("")
                return styles

            styled_df = df_final.style.apply(
                highlight_changes,
                axis=1
            )

            # -------------------------------
            # EXPORT CSV (senza colori)
            # -------------------------------
            buffer = io.StringIO()
            df_final.to_csv(buffer, index=False)
            csv_data = buffer.getvalue()

            st.download_button(
                label="📥 Scarica META_GOOGLE_SHOPIFY_CORRETTO.csv",
                data=csv_data,
                file_name="META_GOOGLE_SHOPIFY_CORRETTO.csv",
                mime="text/csv",
                key="pma_download"
            )

            # -------------------------------
            # ANTEPRIMA
            # -------------------------------
            st.subheader("🔴 Anteprima dati corretti (celle rosse = modificate)")
            st.dataframe(styled_df, use_container_width=True)

        except Exception as e:
            st.error(f"❌ Errore durante l'elaborazione: {e}")
    else:
        st.warning("⚠️ Carica entrambi i file per procedere.")

# -------------------------------
# TAB 2: GENERIC CSV MERGER
# -------------------------------
with tab2:
    st.header("📂 Unione Multi-File CSV")
    st.markdown("""
    Carica più file CSV con la stessa struttura per unirli in un unico file finale.
    """)

    uploaded_files = st.file_uploader(
        "Carica i file CSV da unire",
        type="csv",
        accept_multiple_files=True,
        key="multi_csv_upl"
    )

    if uploaded_files:
        try:
            df_list = []
            for f in uploaded_files:
                # Prova a leggere il CSV (gestione delimitatore automatica o default)
                temp_df = pd.read_csv(f)
                temp_df['Source_File'] = f.name # Aggiunge info sulla sorgente
                df_list.append(temp_df)
            
            if df_list:
                merged_df = pd.concat(df_list, ignore_index=True)
                
                st.success(f"✅ {len(uploaded_files)} file uniti con successo!")
                st.info(f"Righe totali: {len(merged_df)}")
                
                # Opzione per rimuovere la colonna Source_File
                remove_source = st.checkbox("Rimuovi colonna 'Source_File' dal file finale", value=False)
                if remove_source:
                    merged_df = merged_df.drop(columns=['Source_File'])

                # Export
                buffer_merge = io.StringIO()
                merged_df.to_csv(buffer_merge, index=False)
                csv_merged_data = buffer_merge.getvalue()

                st.download_button(
                    label="📥 Scarica CSV Unito",
                    data=csv_merged_data,
                    file_name="merged_data.csv",
                    mime="text/csv",
                    key="merged_download"
                )

                st.subheader("Anteprima file unito")
                st.dataframe(merged_df.head(100), use_container_width=True)

        except Exception as e:
            st.error(f"❌ Errore durante l'unione: {e}")
    else:
        st.info("Inizia caricando almeno due file CSV.")
