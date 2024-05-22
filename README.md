import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

def normalize_costs(costs, total_cost):
    normalized_costs = [cost / total_cost for cost in costs]
    sum_other_costs = sum(normalized_costs) - normalized_costs[0]
    normalized_weights = [cost / sum_other_costs if i != 0 else cost for i, cost in enumerate(normalized_costs)]
    return normalized_weights

def calculate_scores(grades, normalized_weights):
    grades_df = pd.DataFrame(grades, index=criteria)
    scores = grades_df.mul(normalized_weights, axis=0)
    total_scores = scores.sum()
    return scores, total_scores

st.title('Wastewater Treatment Plant Decision Making')

# Define the criteria
criteria = ['Life Cycle Cost', 'Stable Operation', 'Flexibility', 'CO2 Footprint', 'Energy Consumption', 'Working Environment and Safety']

# Define the fixed total annual cost
total_annual_cost = 5051546.392

# Input for costs per criteria
st.sidebar.header('Input Costs for Each Criterion')
costs = []
cost_units = {
    'Life Cycle Cost': 'fixed',
    'Stable Operation': 'SEK/yr/Grade',
    'Flexibility': 'SEK/yr/Grade',
    'CO2 Footprint': 'ton CO2e/year',
    'Energy Consumption': 'kWh/year',
    'Working Environment and Safety': 'SEK/yr/Grade'
}

for crit in criteria:
    if cost_units[crit] == 'fixed':
        cost = 1  # Life Cycle Cost is fixed
    elif crit == 'CO2 Footprint':
        amount = st.sidebar.number_input(f'Amount for {crit} ({cost_units[crit]})', value=100)
        cost = abs(amount * -4250)
    elif crit == 'Energy Consumption':
        amount = st.sidebar.number_input(f'Amount for {crit} ({cost_units[crit]})', value=10000)
        cost = abs(amount * -593.4065934)
    else:
        cost = st.sidebar.number_input(f'Cost for {crit} ({cost_units[crit]})', value=1000000)
    costs.append(cost)

# Normalize the costs
normalized_weights = normalize_costs(costs, total_annual_cost)

# Input for expert grades
st.sidebar.header('Input Expert Grades for Alternatives')
alternatives = ['AS', 'MBR', 'AGS']
grades = {alt: [] for alt in alternatives}
for alt in alternatives:
    st.sidebar.subheader(f'{alt} Grades')
    for crit in criteria:
        grade = st.sidebar.slider(f'{crit} Grade for {alt}', 1.0, 5.0, 3.0, step=0.1)
        grades[alt].append(round(grade, 1))

# Calculate scores
scores, total_scores = calculate_scores(grades, normalized_weights)

# Ensure final scores are within 0-5 range
final_scores = total_scores / total_scores.max() * 5

# Display scores
st.subheader('Scores for Each Alternative')
scores_df = scores.copy()
scores_df.loc['Final Score'] = final_scores
st.dataframe(scores_df)

# Define colors for each criterion
colors = {
    'Life Cycle Cost': 'grey',
    'Stable Operation': 'orange',
    'Flexibility': 'lightcoral',
    'CO2 Footprint': 'lightgreen',
    'Energy Consumption': 'darkgreen',
    'Working Environment and Safety': 'lightblue'
}

# Plotting the scores
fig, ax = plt.subplots(figsize=(10, 6))
for i, crit in enumerate(criteria):
    ax.bar(alternatives, scores.loc[crit], bottom=scores.iloc[:i].sum(), label=crit, color=colors[crit])

# Adding final scores on top of each bar
for i, total in enumerate(final_scores):
    ax.text(i, total + 0.1, round(total, 2), ha='center')

ax.set_xlabel('Alternatives')
ax.set_ylabel('Score')
ax.set_title('Scores for Wastewater Treatment Plant Alternatives')
ax.legend(title='Criteria', bbox_to_anchor=(1.05, 1), loc='upper left')
ax.set_xticks(range(len(alternatives)))
ax.set_xticklabels(alternatives, rotation=0)
plt.tight_layout()
st.pyplot(fig)

# Display the optimal alternative
optimal_alternative = final_scores.idxmax()
st.subheader(f'The most optimal alternative is: {optimal_alternative} with a score of {final_scores[optimal_alternative]:.2f}')

# Explanation of the table
st.markdown("""
### Explanation of the Table
- The table shows the scores for each alternative (AS, MBR, AGS) for each criterion.
- The 'Final Score' row represents the total score for each alternative, normalized to be within 0-5 range.
- The criteria are weighted based on the input costs, and the expert grades are multiplied by these weights to get the scores.
""")
