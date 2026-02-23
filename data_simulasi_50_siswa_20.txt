import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import matplotlib.pyplot as plt
import seaborn as sns

st.set_page_config(page_title="Dashboard Penelitian Hasil Belajar", layout="wide")

st.title("📊 Dashboard Penelitian Analisis Hasil Belajar")
st.write("Analisis 50 Siswa – 20 Soal (Skala 1–4)")

uploaded_file = st.file_uploader("Upload file Excel", type=["xlsx"])

if uploaded_file is not None:
    df = pd.read_excel(uploaded_file)

    st.subheader("📌 Data Preview")
    st.dataframe(df.head())

    # =========================
    # OVERVIEW STATISTIK
    # =========================
    st.subheader("📊 Statistik Deskriptif")

    jumlah_siswa = df.shape[0]
    jumlah_soal = df.shape[1]
    rata_total = df.mean().mean()
    std_total = df.stack().std()

    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Jumlah Siswa", jumlah_siswa)
    col2.metric("Jumlah Soal", jumlah_soal)
    col3.metric("Rata-rata Total", round(rata_total,2))
    col4.metric("Standar Deviasi", round(std_total,2))

    # =========================
    # ANALISIS PER SISWA
    # =========================
    st.subheader("👩‍🎓 Analisis Per Siswa")

    df["Rata_Siswa"] = df.mean(axis=1)

    def kategori(x):
        if x < 2:
            return "Rendah"
        elif x < 3:
            return "Sedang"
        else:
            return "Tinggi"

    df["Kategori"] = df["Rata_Siswa"].apply(kategori)

    fig_hist = px.histogram(df, x="Rata_Siswa", nbins=10,
                            title="Distribusi Rata-rata Siswa")
    st.plotly_chart(fig_hist)

    st.subheader("🏆 Ranking Siswa")
    ranking = df.sort_values("Rata_Siswa", ascending=False)
    ranking["Ranking"] = range(1, len(ranking)+1)
    st.dataframe(ranking[["Ranking","Rata_Siswa","Kategori"]])

    # =========================
    # ANALISIS PER SOAL
    # =========================
    st.subheader("📝 Analisis Tingkat Kesulitan Soal")

    rata_soal = df.iloc[:, :jumlah_soal].mean()
    soal_df = pd.DataFrame({
        "Soal": rata_soal.index,
        "Rata-rata": rata_soal.values
    })

    fig_bar = px.bar(soal_df, x="Soal", y="Rata-rata",
                     title="Rata-rata Skor Per Soal")
    st.plotly_chart(fig_bar)

    soal_tertinggi = rata_soal.idxmax()
    soal_terendah = rata_soal.idxmin()

    st.success(f"Soal termudah (rata-rata tertinggi): {soal_tertinggi}")
    st.error(f"Soal tersulit (rata-rata terendah): {soal_terendah}")

    # =========================
    # HEATMAP PERFORMA
    # =========================
    st.subheader("🔥 Heatmap Performa Siswa")

    fig, ax = plt.subplots(figsize=(12,6))
    sns.heatmap(df.iloc[:, :jumlah_soal], cmap="YlGnBu", ax=ax)
    st.pyplot(fig)

    # =========================
    # RELIABILITAS (CRONBACH ALPHA)
    # =========================
    st.subheader("📈 Analisis Reliabilitas Instrumen")

    item_scores = df.iloc[:, :jumlah_soal]
    item_variances = item_scores.var(axis=0, ddof=1)
    total_score = item_scores.sum(axis=1)
    total_variance = total_score.var(ddof=1)

    k = jumlah_soal
    cronbach_alpha = (k/(k-1)) * (1 - item_variances.sum()/total_variance)

    st.metric("Cronbach's Alpha", round(cronbach_alpha,3))

    if cronbach_alpha >= 0.9:
        st.success("Reliabilitas Sangat Tinggi")
    elif cronbach_alpha >= 0.8:
        st.success("Reliabilitas Tinggi")
    elif cronbach_alpha >= 0.7:
        st.warning("Reliabilitas Cukup")
    else:
        st.error("Reliabilitas Rendah")