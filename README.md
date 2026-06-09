import numpy as np
import pandas as pd
import plotly.graph_objects as go
from scipy.spatial import KDTree

# 1. Load Datasets
gps_df = pd.read_csv('drone_gps_no_timestamp.csv')
radar_df = pd.read_csv('radar_no_timestamp.csv')

# 2. Origin Reference Constants
lat_0, lon_0, alt_0 = 37.7749, -122.4194, 10.0  
R_e = 6378137.0
lat_0_rad = np.radians(lat_0)

# 3. Transform GPS
gps_x = R_e * np.radians(gps_df['longitude'] - lon_0) * np.cos(lat_0_rad)
gps_y = R_e * np.radians(gps_df['latitude'] - lat_0)
gps_z = gps_df['altitude'] - alt_0

# 4. Transform Radar
az_rad = np.radians(radar_df['azimuth_deg'])
el_rad = np.radians(radar_df['elevation_deg'])
r = radar_df['range_m']
radar_x = r * np.cos(el_rad) * np.sin(az_rad)
radar_y = r * np.cos(el_rad) * np.cos(az_rad)
radar_z = r * np.sin(el_rad)

# 5. Geometrical Nearest Neighbor Matching
gps_points = np.vstack((gps_x, gps_y, gps_z)).T
radar_points = np.vstack((radar_x, radar_y, radar_z)).T
tree = KDTree(radar_points)
distances, closest_indices = tree.query(gps_points)

# 6. Build Plotly Figure
fig = go.Figure()

# Drone GPS Track (Blue line, dot markers)
fig.add_trace(go.Scatter3d(
    x=gps_x, y=gps_y, z=gps_z, mode='lines+markers',
    name='Drone GPS Track',
    marker=dict(size=3, color='#0033cc'),
    line=dict(color='#0033cc', width=2),
    hovertext=[f"Alt: {alt:.1f}m" for alt in gps_df['altitude']]
))

# Radar Track (Red dashed line, 'x' markers)
fig.add_trace(go.Scatter3d(
    x=radar_x, y=radar_y, z=radar_z, mode='lines+markers',
    name='Radar Track',
    marker=dict(size=4, symbol='x', color='#cc0000'),
    line=dict(color='#cc0000', width=1.5, dash='dash'),
    hovertext=[f"Range: {rng:.1f}m" for rng in radar_df['range_m']]
))

# Subtle background indicator lines mapping spatial drift between the tracks (every 4th point)
for idx in range(0, len(gps_points), 4):
    closest_radar_idx = closest_indices[idx]
    fig.add_trace(go.Scatter3d(
        x=[gps_x[idx], radar_x[closest_radar_idx]],
        y=[gps_y[idx], radar_y[closest_radar_idx]],
        z=[gps_z[idx], radar_z[closest_radar_idx]],
        mode='lines',
        line=dict(color='rgba(150,150,150,0.3)', width=1),
        showlegend=False,
        hoverinfo='skip'
    ))

# Layout Styling (Clean title and legends matching the reference image)
fig.update_layout(
    title=dict(text='3D Trajectory Comparison: Drone GPS vs Radar', x=0.5, y=0.95),
    scene=dict(
        xaxis=dict(title='East (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        yaxis=dict(title='North (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        zaxis=dict(title='Up (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        aspectmode='data'
    ),
    paper_bgcolor='white',
    plot_bgcolor='white',
    margin=dict(l=0, r=0, b=0, t=50),
    legend=dict(yanchor="top", y=0.95, xanchor="right", x=0.95)
)

fig.show()
import numpy as np
import pandas as pd
import plotly.graph_objects as go

# 1. Load GPS Dataset
gps_df = pd.read_csv('drone_gps_no_timestamp.csv')

# 2. Origin Reference Constants
lat_0, lon_0, alt_0 = 37.7749, -122.4194, 10.0  
R_e = 6378137.0  # Earth radius in meters
lat_0_rad = np.radians(lat_0)

# 3. Transform GPS to Local Cartesian Coordinates (ENU)
gps_x = R_e * np.radians(gps_df['longitude'] - lon_0) * np.cos(lat_0_rad)
gps_y = R_e * np.radians(gps_df['latitude'] - lat_0)
gps_z = gps_df['altitude'] - alt_0

# 4. Build Plotly Figure
fig = go.Figure()

# Drone GPS Track (Blue line, dot markers)
fig.add_trace(go.Scatter3d(
    x=gps_x, y=gps_y, z=gps_z, mode='lines+markers',
    name='Drone GPS Track',
    marker=dict(size=3, color='#0033cc'),
    line=dict(color='#0033cc', width=2),
    hovertext=[f"Alt: {alt:.1f}m" for alt in gps_df['altitude']]
))

# Layout Styling
fig.update_layout(
    title=dict(text='3D Trajectory: Drone GPS Track', x=0.5, y=0.95),
    scene=dict(
        xaxis=dict(title='East (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        yaxis=dict(title='North (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        zaxis=dict(title='Up (meters)', backgroundcolor="white", gridcolor="lightgray", showbackground=True),
        aspectmode='data'
    ),
    paper_bgcolor='white',
    plot_bgcolor='white',
    margin=dict(l=0, r=0, b=0, t=50),
    legend=dict(yanchor="top", y=0.95, xanchor="right", x=0.95)
)

fig.show()
