# A Python Script to Visualize FreeStyle Libre 2 Sensor Data

FreeStyle Libre 2 is a continuous glucose monitor wearable that measures the glucose concentration in the interstitial fluid and estimates the blood glucose level. [Learn more about the device here](https://www.freestyle.abbott/).

The following code guides you through the steps to download and visualize the data daily.

### Important

The device's website states: "The FreeStyle Libre 2 sensor automatically captures the glucose concentration in the interstitial fluid every minute. It also automatically records the glucose concentration every 15 minutes, storing that data in a rolling 8-hour log."

This has implications! Check the plot below. The red dots represent the moments where the sensor is manually scanned by me, providing data for those specific times. The blue dots represent the normal 15-minute intervals that the device saves automatically. Without continuous scanning, the peak above 7.8 mmol/L would have been missed due to the sluggish recording rate of the sensor.

Therefore, to accurately capture the dynamics of the blood glucose level, one should manually scan the device at shorter intervals, especially during critical periods (e.g., after a meal).

### Visualize Your Data:

1. Visit this website and log in: [https://www.libreview.com/](https://www.libreview.com/). Ensure you use the same credentials as you do for the LibreLink app on your phone.
2. As shown in the picture below, find and download your data.
3. This will give you a CSV file containing several columns.
4. Open a Jupyter Notebook and copy-paste the following code. Make sure to replace `The-Name-Of-Your-Downloaded-File` with the actual filename and place it in the same location as your notebook.
5. Ensure the relevant packages are installed; otherwise, install them.  

**Important:** This code combines the values from both those recorded automatically every 15 minutes and those scanned manually to show a more comprehensive representation.

```Python 

import pandas as pd
import plotly.express as px
import ipywidgets as widgets
from IPython.display import display, clear_output

# Load data
data = pd.read_csv("The-Name-Of-Your-Downloaded-File.csv", header=1)

# Prepare a new DataFrame
df = pd.DataFrame()
df["timestamp"] = pd.to_datetime(data["Device Timestamp"], dayfirst=True)
df['glucose'] = data['Historic Glucose mmol/L'].combine_first(data['Scan Glucose mmol/L'])
df.dropna(inplace=True)
df.set_index("timestamp", inplace=True)
df.sort_index(inplace=True)

# Extract unique dates for the dropdown
dates = df.index.date
unique_dates = sorted(set(dates))

# Create dropdown for date selection
date_dropdown = widgets.Dropdown(
    options=[(date.strftime('%Y-%m-%d'), date) for date in unique_dates],
    description='Select Date:',
    disabled=False,
)

# Create output widget for the plot
output_plot = widgets.Output()

# Function to update plot based on selected date
def update_plot(change):
    selected_date = change['new']
    filtered_df = df.loc[df.index.date == selected_date]
    
    with output_plot:
        clear_output(wait=True)  # Clear the previous plot
        fig = px.line(filtered_df, y="glucose", markers="o", title=f"Glucose Levels on {selected_date.strftime('%Y-%m-%d')}")
        fig.add_hline(y=3.9, line_dash="dash", line_width=1.5, line_color="red")
        fig.add_hline(y=7.8, line_dash="dash", line_width=1.5, line_color="red")
        fig.show()

# Set up observer to update plot when dropdown value changes
date_dropdown.observe(update_plot, names='value')

# Initial call to update_plot to display the initial plot
update_plot({'new': unique_dates[0]})

# Display the widgets
display(date_dropdown, output_plot)

```
### Output Example
The output of the code will appear as shown below, where you can select your date using a widget. If you hover your mouse over it, you can see each specific value for each data point.
