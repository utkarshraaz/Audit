import pandas as pd

def create_input_dataframe(merge_logic, beh_inputs, fin_inputs, loss_inputs, cost_inputs, beh_id):
    # Common columns
    columns = [
        'Year', 'unique_key', 'Scenario', 'Balance', 'Revolving_rate',
        'Promo_revolve_rate', 'APR', 'Sales', 'Interchange_Rate', 'Reward_rate',
        'Annual_Fees', 'Other_Fees', 'PD', 'BCR', 'CostOfFundRate',
        'Operational_Exp', 'Tax_rate', 'Equity_Rate', 'Discount_Rate', 'Decay_rate',
        'Year_1', 'Year_2', 'Year_3', 'Year_4', 'Year_5'
    ]
    
    # Detecting the segment columns dynamically
    segment_columns = [col for col in beh_inputs.columns if col.startswith('Seg_var')]
    columns = ['Year', 'unique_key'] + segment_columns + columns[2:]
    
    # Initialize the input dataframe with dynamic columns
    inputs_df = pd.DataFrame(columns=columns)
    
    # Retrieve the keys from merge_logic based on beh_id
    beh_key = merge_logic['beh_key'][beh_id]
    fin_key = merge_logic['fin_key'][beh_id]
    loss_key = merge_logic['loss_key'][beh_id]
    cost_key = merge_logic['cost_key'][beh_id]
    
    # Helper function to extract values based on keys and year
    def extract_values(input_df, key):
        return input_df[input_df['key'] == key].reset_index(drop=True)
    
    # Extracting input values
    beh_values = extract_values(beh_inputs, beh_key)
    fin_values = extract_values(fin_inputs, fin_key)
    loss_values = extract_values(loss_inputs, loss_key)
    cost_values = extract_values(cost_inputs, cost_key)
    
    for i in range(5):  # 5 years span
        row = {}
        row['Year'] = f'Year_{i + 1}'
        row['unique_key'] = beh_key
        
        # Extracting behavioral inputs
        for seg_col in segment_columns:
            row[seg_col] = beh_values[seg_col][0]
        
        row.update({
            'Scenario': beh_values['Scenario'][0],
            'Balance': beh_values[f'bal_y{i + 1}'][0],
            'Revolving_rate': beh_values[f'revolve_rate_y{i + 1}'][0],
            'Promo_revolve_rate': beh_values[f'promo_revolve_rate_y{i + 1}'][0],
            'Sales': beh_values[f'sales_y{i + 1}'][0],
        })
        
        # Extracting financial inputs
        row.update({
            'APR': fin_values[f'apr_y{i + 1}'][0],
            'Interchange_Rate': fin_values[f'interchange_rate_y{i + 1}'][0],
            'Reward_rate': fin_values[f'reward_rate_y{i + 1}'][0],
            'Annual_Fees': fin_values[f'annual_fee_y{i + 1}'][0],
            'Other_Fees': fin_values[f'other_fee_y{i + 1}'][0],
            'CostOfFundRate': fin_values[f'cost_of_fund_rate_y{i + 1}'][0],
            'Tax_rate': fin_values[f'tax_rate_y{i + 1}'][0],
            'Equity_Rate': fin_values[f'equity_rate_y{i + 1}'][0],
            'Discount_Rate': fin_values[f'discount_rate_y{i + 1}'][0],
            'Decay_rate': fin_values['decay_rate'][0],
        })
        
        # Extracting loss inputs
        row.update({
            'PD': loss_values[f'pd_y{i + 1}'][0],
            'BCR': loss_values[f'bcr_y{i + 1}'][0],
        })
        
        # Extracting cost inputs
        row['Operational_Exp'] = cost_values[f'operational_cost_y{i + 1}'][0]
        
        inputs_df = inputs_df.append(row, ignore_index=True)
    
    return inputs_df
