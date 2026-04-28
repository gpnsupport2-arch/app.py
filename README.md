# app.pyimport streamlit as st
import pandas as pd
import plotly.express as px

# Page configuration
st.set_page_config(page_title="Support Analytics Pro", layout="wide")

st.title("🎧 Customer Support Analytics Portal")
st.markdown("---")

# 1. File Uploader Component
uploaded_file = st.file_uploader("Upload your Support Export (CSV or Excel)", type=['csv', 'xlsx'])

# 2. Data Loading Logic
if uploaded_file is not None:
    try:
        if uploaded_file.name.endswith('.csv'):
            df = pd.read_csv(uploaded_file)
        else:
            df = pd.read_excel(uploaded_file)
        
        st.success("File uploaded successfully!")
        
        # --- Dashboard UI Starts Here ---
        
        # KPI Row
        col1, col2, col3, col4 = st.columns(4)
        total_tickets = len(df)
        avg_csat = df['CSAT'].mean() if 'CSAT' in df.columns else 0
        
        col1.metric("Total Tickets", total_tickets)
        col2.metric("Avg CSAT", f"{avg_csat:.1f} / 5")
        col3.metric("Status: Open", len(df[df['Status'] == 'Open']) if 'Status' in df.columns else "N/A")
        col4.metric("Status: Resolved", len(df[df['Status'] == 'Resolved']) if 'Status' in df.columns else "N/A")

        st.markdown("---")

        # Charts Row
        c1, c2 = st.columns(2)
        
        with c1:
            if 'Channel' in df.columns:
                st.subheader("Volume by Channel")
                fig = px.pie(df, names='Channel', hole=0.4, color_discrete_sequence=px.colors.qualitative.Pastel)
                st.plotly_chart(fig, use_container_width=True)
        
        with c2:
            if 'Date' in df.columns:
                st.subheader("Daily Ticket Trend")
                df['Date'] = pd.to_datetime(df['Date'])
                trend = df.groupby('Date').size().reset_index(name='Count')
                fig_line = px.line(trend, x='Date', y='Count', markers=True)
                st.plotly_chart(fig_line, use_container_width=True)

        # Data Table View
        with st.expander("View Raw Data Table"):
            st.dataframe(df)

    except Exception as e:
        st.error(f"Error processing file: {e}")
else:
    st.info("👋 Welcome! Please upload a data file to generate the presentation dashboard.")
    st.image("https://streamlit.io/images/brand/streamlit-mark-color.png", width=100)
