import streamlit as st
import pandas as pd
import sqlite3

# --- KONFIGURACJA BAZY DANYCH ---
def init_db():
    conn = sqlite3.connect('magazyn.db')
    c = conn.cursor()
    # Tworzenie tabeli Kategoriee na podstawie obrazka
    c.execute('''
        CREATE TABLE IF NOT EXISTS kategoriee (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nazwa TEXT NOT NULL,
            opis TEXT
        )
    ''')
    # Tworzenie tabeli Produkty na podstawie obrazka
    c.execute('''
        CREATE TABLE IF NOT EXISTS produkty (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            nazwa TEXT NOT NULL,
            numer INTEGER,
            cena REAL,
            kategoria_id INTEGER,
            FOREIGN KEY (kategoria_id) REFERENCES kategoriee (id)
        )
    ''')
    conn.commit()
    return conn

conn = init_db()

# --- INTERFEJS STREAMLIT ---
st.set_page_config(page_title="Zarządzanie Produktami", layout="wide")
st.title("📦 System Zarządzania Produktami")

tabs = st.tabs(["Dodaj Dane", "Przeglądaj Magazyn"])

# --- TAB 1: DODAWANIE DANYCH ---
with tabs[0]:
    col1, col2 = st.columns(2)
    
    with col1:
        st.subheader("Nowa Kategoria")
        kat_nazwa = st.text_input("Nazwa kategorii")
        kat_opis = st.text_area("Opis kategorii")
        if st.button("Dodaj Kategorię"):
            conn.execute("INSERT INTO kategoriee (nazwa, opis) VALUES (?, ?)", (kat_nazwa, kat_opis))
            conn.commit()
            st.success(f"Dodano kategorię: {kat_nazwa}")

    with col2:
        st.subheader("Nowy Produkt")
        # Pobieranie kategorii do listy wyboru
        kategorie_df = pd.read_sql_query("SELECT id, nazwa FROM kategoriee", conn)
        
        if not kategorie_df.empty:
            p_nazwa = st.text_input("Nazwa produktu")
            p_numer = st.number_input("Numer (int8)", step=1)
            p_cena = st.number_input("Cena (numeric)", format="%.2f")
            p_kat = st.selectbox("Wybierz kategorię", kategorie_df['nazwa'].tolist())
            
            p_kat_id = int(kategorie_df[kategorie_df['nazwa'] == p_kat]['id'].values[0])
            
            if st.button("Dodaj Produkt"):
                conn.execute(
                    "INSERT INTO produkty (nazwa, numer, cena, kategoria_id) VALUES (?, ?, ?, ?)",
                    (p_nazwa, p_numer, p_cena, p_kat_id)
                )
                conn.commit()
                st.success(f"Dodano produkt: {p_nazwa}")
        else:
            st.warning("Najpierw dodaj przynajmniej jedną kategorię!")

# --- TAB 2: PRZEGLĄDANIE DANYCH ---
with tabs[1]:
    st.subheader("Lista Produktów (z Kategoriami)")
    query = """
    SELECT 
        p.id, p.nazwa, p.numer, p.cena, k.nazwa as kategoria
    FROM produkty p
    LEFT JOIN kategoriee k ON p.kategoria_id = k.id
    """
    df = pd.read_sql_query(query, conn)
    st.dataframe(df, use_container_width=True)

    if st.button("Wyczyść bazę danych"):
        conn.execute("DELETE FROM produkty")
        conn.execute("DELETE FROM kategoriee")
        conn.commit()
        st.rerun()
