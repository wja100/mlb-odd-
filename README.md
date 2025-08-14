# mlb-odd-
odds
import streamlit as st
import pandas as pd
import plotly.express as px

# Sample data
data = [
    {"Matchup": "Reds vs Nationals", "OpeningML": -145, "CurrentML": -125, "Bets": 65, "Handle": 67},
    {"Matchup": "Reds vs Nationals", "OpeningML": +120, "CurrentML": +105, "Bets": 35, "Handle": 33},
    {"Matchup": "Padres vs Marlins", "OpeningML": -145, "CurrentML": -150, "Bets": 77, "Handle": 92},
    {"Matchup": "Padres vs Marlins", "OpeningML": +125, "CurrentML": +140, "Bets": 23, "Handle": 8},
    # Add more rows as needed...
]

df = pd.DataFrame(data)

# Calculated metrics
df['SharpMoney'] = df['Handle'] - df['Bets']
df['PublicFade'] = (df['Bets'] > 70) & (df['Handle'] < 50)
df['LineMovement'] = df['CurrentML'] - df['OpeningML']

# Sidebar filters
st.sidebar.header("🔧 Filters")
min_bets = st.sidebar.slider("Minimum Bets %", 0, 100, 0)
max_handle = st.sidebar.slider("Maximum Handle %", 0, 100, 100)

filtered_df = df[(df['Bets'] >= min_bets) & (df['Handle'] <= max_handle)]

# Title
st.title("📊 Betting Insights Dashboard (No Team Names)")

# Metrics summary
col1, col2 = st.columns(2)
with col1:
    st.metric("📈 Avg Sharp Money", round(df['SharpMoney'].mean(), 2))
with col2:
    st.metric("📉 Avg Line Movement", round(df['LineMovement'].mean(), 2))

# Tabs for strategy breakdown
tab1, tab2, tab3 = st.tabs(["🔍 Sharp Money", "⚠️ Public Fade", "📉 Line Movement"])

with tab1:
    sharp_plays = df[df['SharpMoney'] > 10]
    with st.expander("View Sharp Money Table"):
        st.dataframe(sharp_plays[['Matchup', 'Bets', 'Handle', 'SharpMoney']])
    fig1 = px.bar(sharp_plays, x="Matchup", y="SharpMoney", title="Sharp Money by Matchup")
    st.plotly_chart(fig1, use_container_width=True)

with tab2:
    public_fades = df[df['PublicFade']]
    with st.expander("View Public Fade Table"):
        st.dataframe(public_fades[['Matchup', 'Bets', 'Handle']])
    fig2 = px.scatter(public_fades, x="Bets", y="Handle", color="Matchup", title="Public Fade Bets vs Handle")
    st.plotly_chart(fig2, use_container_width=True)

with tab3:
    line_moves = df[df['LineMovement'].abs() > 10]
    with st.expander("View Line Movement Table"):
        st.dataframe(line_moves[['Matchup', 'OpeningML', 'CurrentML', 'LineMovement']])
    fig3 = px.bar(line_moves, x="Matchup", y="LineMovement", title="Line Movement by Matchup")
    st.plotly_chart(fig3, use_container_width=True)

# Filtered results
st.subheader("📌 Filtered Results")
st.dataframe(filtered_df[['Matchup', 'Bets', 'Handle', 'SharpMoney', 'LineMovement']])

# Download button
st.download_button(
    label="📥 Download Filtered Data",
    data=filtered_df.to_csv(index=False),
    file_name="filtered_betting_data.csv",
    mime="text/csv"
)
