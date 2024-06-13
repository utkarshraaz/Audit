import pandas as pd

def create_input_dataframe(merge_logic, beh_inputs, fin_inputs, loss_inputs, cost_inputs, beh_id):
    # Common columns
    columns = [
        'Year', 'unique_key', 'Scenario', 'Balance', 'Revolving_rate',
        'Promo_revolve_rate', 'APR', 'Sales', 'Interchange_Rate', 'Reward_rate',
        'Annual_Fees', 'Other_Fees', 'PD', 'BCR', 'CostOfFundRate',
        'Operational_Exp', 'Tax_rate', 'Equity_Rate', 'Discount_Rate', 'Decay_rate'
    ]
    
    # Detecting the segment columns dynamically
    segment_columns = [col for col in beh_inputs.columns if col.startswith('Seg_var')]
    columns = ['Year', 'unique_key'] + segment_columns + columns[2:]
    
    # Detecting the year indexes dynamically
    year_indexes = [col for col in beh_inputs.columns if col.startswith('bal_y')]
    year_count = len(year_indexes)
    
    # Initialize the input dataframe with dynamic columns
    inputs_df = pd.DataFrame(columns=columns)
    
    # Retrieve the keys from merge_logic based on beh_id
    beh_key = merge_logic['beh_key'][beh_id]
    fin_key = merge_logic['fin_key'][beh_id]
    loss_key = merge_logic['loss_key'][beh_id]
    cost_key = merge_logic['cost_key'][beh_id]
    
    # Helper function to extract values based on keys
    def extract_values(input_df, key):
        return input_df[input_df['key'] == key].reset_index(drop=True)
    
    # Extracting input values
    beh_values = extract_values(beh_inputs, beh_key)
    fin_values = extract_values(fin_inputs, fin_key)
    loss_values = extract_values(loss_inputs, loss_key)
    cost_values = extract_values(cost_inputs, cost_key)
    
    for i in range(year_count):  # Dynamic year span
        row = {}
        year_suffix = f'y{i + 1}'
        row['Year'] = f'Year_{i + 1}'
        row['unique_key'] = beh_key
        
        # Extracting behavioral inputs
        for seg_col in segment_columns:
            row[seg_col] = beh_values[seg_col][0]
        
        row.update({
            'Scenario': beh_values['Scenario'][0],
            'Balance': beh_values[f'bal_{year_suffix}'][0],
            'Revolving_rate': beh_values[f'revolve_rate_{year_suffix}'][0],
            'Promo_revolve_rate': beh_values[f'promo_revolve_rate_{year_suffix}'][0],
            'Sales': beh_values[f'sales_{year_suffix}'][0],
        })
        
        # Extracting financial inputs
        row.update({
            'APR': fin_values[f'apr_{year_suffix}'][0],
            'Interchange_Rate': fin_values[f'interchange_rate_{year_suffix}'][0],
            'Reward_rate': fin_values[f'reward_rate_{year_suffix}'][0],
            'Annual_Fees': fin_values[f'annual_fee_{year_suffix}'][0],
            'Other_Fees': fin_values[f'other_fee_{year_suffix}'][0],
            'CostOfFundRate': fin_values[f'cost_of_fund_rate_{year_suffix}'][0],
            'Tax_rate': fin_values[f'tax_rate_{year_suffix}'][0],
            'Equity_Rate': fin_values[f'equity_rate_{year_suffix}'][0],
            'Discount_Rate': fin_values[f'discount_rate_{year_suffix}'][0],
            'Decay_rate': fin_values['decay_rate'][0],
        })
        
        # Extracting loss inputs
        row.update({
            'PD': loss_values[f'pd_{year_suffix}'][0],
            'BCR': loss_values[f'bcr_{year_suffix}'][0],
        })
        
        # Extracting cost inputs
        row['Operational_Exp'] = cost_values[f'operational_cost_{year_suffix}'][0]
        
        inputs_df = inputs_df.append(row, ignore_index=True)
    
    return inputs_df
