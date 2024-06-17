# pandas-challenge-1

## Module 4

## Part 1: Date

    import pandas as pd

    df = pd.read_csv('Resources/client_dataset.csv')
    df.head()

## View the column names in the data

    df.columns
    print(df.columns)

## Use the describe function to gather some basic statistics

df.describe()
statistics = df.describe().round(2)
print(statistics)

## Use this space to do any additional research

## and familiarize yourself with the data

df.info()
df.count()

## What three item categories had the most entries?

df['category']= df['category'].astype(str)
top_three_categories = df['category'].value_counts().head(3)
print(top_three_categories)

## For the category with the most entries, which subcategory had the most entries?

most_common_category = df['category'].value_counts().idxmax()
filtered_df = df[df['category'] == most_common_category]
subcategory_counts = filtered_df['subcategory'].value_counts()
print(subcategory_counts)
top_subcategory = subcategory_counts.idxmax()
print(top_subcategory, "has the most subcategory entries within consumables")

## Which five clients had the most entries in the data?

top_five_clients = df['client_id'].value_counts()
top_five_clients = top_five_clients.head(5)
print(top_five_clients)

## Store the client ids of those top 5 clients in a list

top_five_clients_ids = top_five_clients.index.tolist()
print("Top 5 clients by quantity:")
print(top_five_clients_ids)

## How many total units (the qty column) did the client with the most entries order?

client_id_qty = 33615
client_id_qty_total = df.loc[df['client_id'] == client_id_qty, 'qty'].sum()
print(client_id_qty_total)

## Part 2: Transform the Data

## Do we know that this client spent the more money than client 66037? If not, how would we find out? Transform the data using the steps below to prepare it for analysis

## Create a column that calculates the subtotal for each line using the unit_price and the qty

df['subtotal'] = df['unit_price'] * df['qty']
print(df['subtotal'].head())

## Create a column for shipping price

## Assume a shipping price of $7 per pound for orders over 50 pounds and $10 per pound for items 50 pounds or under

def calculation_shipping_price(row):
    item_weight = row['unit_weight']
    if item_weight > 50:
        shipping_price = item_weight *7
    else:
        shipping_price = item_weight* 10
    return shipping_price
df['shipping_price'] = df.apply(calculation_shipping_price, axis=1)
print(df[['unit_weight', 'shipping_price']].head())

## Create a column for the total price using the subtotal and the shipping price along with a sales tax of 9.25%

sales_tax_rate = 0.0925
df['total_price'] = df['subtotal'] + df['shipping_price'] + df['subtotal'] * sales_tax_rate
print(df['total_price'])

## Create a column for the cost of each line using unit cost, qty, and

## shipping price (assume the shipping cost is exactly what is charged to the client)

df['line_cost'] = (df['unit_cost'] * df['qty']) + df['shipping_price']
print(df[['unit_cost', 'qty', 'shipping_price', 'line_cost']].head())

## Create a column for the profit of each line using line cost and line price

df['line_profit'] = df['total_price'] - df['line_cost']
print(df[['total_price', 'line_cost', 'line_profit']].head())

## Part 3: Confirm your work

You have email receipts showing that the total prices for 3 orders. Confirm that your calculations match the receipts. Remember, each order has multiple lines.

Order ID 2742071 had a total price of \$152,811.89
Order ID 2173913 had a total price of \$162,388.71
Order ID 6128929 had a total price of \$923,441.25

## Check your work using the totals above

order1_total_price = df[df['order_id'] == 2742071]['total_price'].sum().round(2)
order2_total_price = df[df['order_id'] == 2173913]['total_price'].sum().round(2)
order3_total_price = df[df['order_id'] == 6128929]['total_price'].sum().round(2)
print("Total Price for Order ID 2742071:", order1_total_price)
print("Total Price for Order ID 2173913:", order2_total_price)
print("Total Price for Order ID 6128929:", order3_total_price)

## Part 4: Summarize and Analyze

Use the new columns with confirmed values to find the following information.

## How much did each of the top 5 clients by quantity spend? Check your work from Part 1 for client ids

top_five_clients_spending = df[df['client_id'].isin(top_five_clients_ids)].groupby['client_id']('total_price').sum()
print("Total spending for the top 5 clients:")
print(top_five_clients_spending)

## Create a summary DataFrame showing the totals for the for the top 5 clients with the following information

## total units purchased, total shipping price, total revenue, and total profit

top_five_totals = df[df['client_id'].isin(top_five_clients_ids)].groupby('client_id').agg(
    total_units_purchased=('qty', 'sum'),
    total_shipping_price=('shipping_price', 'sum'),
    total_revenue=('total_price', 'sum'),
    total_profit=('line_profit', 'sum')
    )
print("Summary DataFrame for Top 5 Clients:")
print(top_five_totals)

## Format the data and rename the columns to names suitable for presentation

top_five_clients_totals_data = pd.DataFrame(top_five_totals)

## Define the money columns

money_columns = ('total_shipping_price', 'total_revenue', 'total_profit')

## Define a function that converts a dollar amount to millions

def convert_to_millions(amount):
    if abs(amount) >= 1_000_000:
        return '${:,.2f}M'.format(amount / 1_000_000)
    else:
        return '${:,.2f}'.format(amount)

## Apply the currency_format_millions function to only the money columns

for col in money_columns:
    top_five_clients_totals_data[col] = top_five_clients_totals_data[col].apply(convert_to_millions)

## Rename the columns to reflect the change in the money format.
renamed_columns = top_five_clients_totals_data.rename(columns={
    'total_units_purchased': 'Total Units Purchased (Millions)',
    'total_shipping_price': 'Total Shipping Price (Millions)',
    'total_revenue': 'Total Revenue (Millions)',
    'total_profit': 'Total Profit (Millions)'})
print(renamed_columns)

## Sort the updated data by "Total Profit (millions)" form highest to lowest and assign the sort to a new DatFrame

sorted_data = renamed_columns.sort_values(by='Total Profit (Millions)', ascending=False)

## Print the sorted DataFrame

print(sorted_data)
