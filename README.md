
# Spotify Analysis Dashboard 2024 (Power BI)

I developed a Power BI dashboard for Spotify analysis of year 2024, integrating Python for API calls to fetch data directly from Spotify for Developers, including cover URLs. The dashboard incorporates various visual graphs to highlight key insights, with a special focus on a heatmap created using Python visuals with Matplotlib and Seaborn. 
The dashboard integrates interactive visualizations and insights, making it easy to explore patterns in streaming data. By leveraging advanced analytics, it provides a comprehensive view of the evolving music landscape and user preferences, offering actionable insights for music enthusiasts and industry professionals alike.

![Screenshot 2024-12-10 234337](https://github.com/user-attachments/assets/5dfa2c28-427f-4da2-b1ee-e765a16c0405)





## Process for creating Dashboard
- Data Collection

 Downloaded the dataset from Kaggle.

 
- Data Enrichment

 Used Spotify's Developer API to fetch album cover URLs and added them to the dataset in Excel.
 
(This workflow efficiently integrates Python with Spotify's API to enrich a dataset by fetching album cover URLs. Using an access token, tracks are searched by name and artist to retrieve unique IDs, and the URLs are fetched through API calls. The data is updated in a pandas DataFrame and exported for analysis.)

```bash
import requests
import pandas as pd

#Function to get Spotify access token
def get_spotify_token(client_id, client_secret):
auth_url = 'https://accounts.spotify.com/api/token'
auth_response = requests.post(auth_url, {
'grant_type': 'client_credentials',
'client_id': client_id,
'client_secret': client_secret,
})
auth_data = auth_response.json()
return auth_data['access_token']

#Function to search for a track and get its ID
def search_track(track_name, artist_name, token):
query = f"{track_name} artist:{artist_name}"
url = f"https://api.spotify.com/v1/search?q={query}&type=track"
response = requests.get(url, headers={
'Authorization': f'Bearer {token}'
})
json_data = response.json()
try:
first_result = json_data['tracks']['items'][0]
track_id = first_result['id']
return track_id
except (KeyError, IndexError):
return None

#Function to get track details
def get_track_details(track_id, token):
url = f"https://api.spotify.com/v1/tracks/{track_id}"
response = requests.get(url, headers={
'Authorization': f'Bearer {token}'
})
json_data = response.json()
image_url = json_data['album']['images'][0]['url']
return image_url

#Your Spotify API Credentials
client_id = 'your_client_id'
client_secret = 'your_client_secret'

#Get Access Token
access_token = get_spotify_token(client_id, client_secret)

#Read your DataFrame (replace 'your_file.csv' with the path to your CSV file)
df_spotify = pd.read_csv('your_file.csv', encoding='ISO-8859-1')

#Loop through each row to get track details and add to DataFrame
for i, row in df_spotify.iterrows():
track_id = search_track(row['track_name'], row['artist_name'], access_token)
if track_id:
image_url = get_track_details(track_id, access_token)
df_spotify.at[i, 'image_url'] = image_url

#Save the updated DataFrame (replace 'updated_file.csv' with your desired output file name)
df_spotify.to_csv('updated_file.csv', index=False)
```
    


- Dashboard Design

 Created a custom canvas background for the dashboard using Figma.

- Data Visualization

 Developed various visualizations for the enhanced dataset, including a heatmap created using Python with Matplotlib and Seaborn.

 Code for Heatmap

```bash
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.patches import FancyBboxPatch
from matplotlib.colors import Normalize, LinearSegmentedColormap
import matplotlib.cm as cm

# Simulate data
data = pd.DataFrame({
    "day_of_week": np.random.choice(["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"], 365),
    "month": np.random.choice(["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"], 365),
    "orders": np.random.randint(1, 100, 365)
})

# Pivot table for heatmap
heatmap_data = data.pivot_table(
    values="orders",
    index="day_of_week",
    columns="month",
    aggfunc="sum"
)

# Ensure the correct order of days and months
days_order = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"]
months_order = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
heatmap_data = heatmap_data.reindex(index=days_order, columns=months_order)

# Create a custom colormap using #1db954
custom_cmap = LinearSegmentedColormap.from_list("CustomGreen", ["#212121", "#1db954"])

# Normalize the data for coloring
norm = Normalize(vmin=heatmap_data.min().min(), vmax=heatmap_data.max().max())

# Create the figure with a transparent background
fig, ax = plt.subplots(figsize=(14, 9))
fig.patch.set_alpha(0)  # Set figure background to transparent
ax.set_facecolor((0, 0, 0, 0))  # Set axes background to transparent

# Set block dimensions and padding
block_width = 0.8
block_height = 0.8
padding = 0.1  # Space between blocks

# Draw heatmap with separated rounded blocks and numerical values
for i, day in enumerate(heatmap_data.index):
    for j, month in enumerate(heatmap_data.columns):
        value = heatmap_data.loc[day, month]
        if not pd.isna(value):  # Only draw for valid values
            color = custom_cmap(norm(value))  # Use custom colormap for valid values
            rect = FancyBboxPatch(
                (j + padding, i + padding),  # Add padding for separation
                block_width, block_height,  # Width and height of blocks
                boxstyle="round,pad=0.1,rounding_size=0.29",
                linewidth=0,
                facecolor=color
            )
            ax.add_patch(rect)
            # Add numerical value as text
            ax.text(j + 0.5, i + 0.5, f"{int(value)}",  # Center the text
                    color="White" if norm(value) > 0.5 else "white",  # Adjust text color for visibility
                    ha="center", va="center", fontsize=18)

# Adjust axis and labels
ax.set_xticks(np.arange(len(months_order)) + 0.5)
ax.set_yticks(np.arange(len(days_order)) + 0.5)
ax.set_xticklabels(months_order, rotation=45, ha="right", fontsize=20, color="white")  # Change tick labels to white
ax.set_yticklabels(days_order, fontsize=20, color="white")  # Change tick labels to white

# Remove the outer black rectangular box (spines)
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)
ax.spines["left"].set_visible(False)
ax.spines["bottom"].set_visible(False)

# Add color bar
sm = cm.ScalarMappable(cmap=custom_cmap, norm=norm)
sm.set_array([])
cbar = fig.colorbar(sm, ax=ax, orientation="vertical", shrink=0.8, aspect=30)
cbar.ax.yaxis.set_tick_params(color="white")  # Change tick color to white
plt.setp(cbar.ax.get_yticklabels(), color="white")  # Change color of color bar tick labels

# Set limits and aspect
ax.set_xlim(-padding, len(months_order))
ax.set_ylim(-padding, len(days_order))
ax.invert_yaxis()  # Ensure the days start from the top

# Final adjustments
plt.tight_layout()
plt.show()

```
    


 # Hi, I'm Anush! 👋

 

## 🚀 About Me
I am a pre-final year student at JECRC University with a strong passion for Machine Learning, Data Science, and Data Analytics. My foundation in Data Structures and Algorithms (DSA) in C++ has sharpened my problem-solving skills and further fueled my interest in the evolving tech landscape.

 
## 🛠 Skills
Programming Languages: Python, R, C++, SQL
- Tools: VSCode, Github Codespaces, PyCharm, Jupyter Notebook, PowerBI, Github, MySQL, Machine Learning, Figma
- Frameworks: NumPy, Pandas, Matplotlib, SciPy, Scikit-learn, TensorFlow
- UI and UX 



## 🔗 Links

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/anush-sharma13//)







