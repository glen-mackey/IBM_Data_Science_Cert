#preamble to run in terminal
#pip3 install pandas dash
#wget "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/datasets/spacex_launch_dash.csv"
#only need to run this the first time to get 'spacex_dash_app.py' if I haven't already created it
#wget "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/spacex_dash_app.py"


# Import required libraries
import pandas as pd
import dash
import dash_html_components as html
import dash_core_components as dcc
from dash.dependencies import Input, Output
import plotly.express as px

# Read the airline data into pandas dataframe
spacex_df = pd.read_csv("spacex_launch_dash.csv")
max_payload = spacex_df['Payload Mass (kg)'].max()
min_payload = spacex_df['Payload Mass (kg)'].min()

#print(spacex_df[spacex_df['Launch Site']=='CCAFS LC-40']['class'].value_counts())

# Create a dash application
app = dash.Dash(__name__)

# Create an app layout
app.layout = html.Div(children=[html.H1('SpaceX Launch Records Dashboard',
                                        style={'textAlign': 'center', 'color': '#503D36',
                                               'font-size': 40}),
                                # TASK 1: Add a dropdown list to enable Launch Site selection
                                # The default select value is for ALL sites
                                #uses [{}, *[{}]] to create an inner list of dictionaries and then explode that list so
                                #the dictionaries merge into the outer list of dictionaries, need the list comprehension
                                #use placeholder to get a baseline text value for when you do a clean run 
                                 dcc.Dropdown(id='site-dropdown',
                                              options=[  {'label':'All Sites', 'value': 'All'},
                                                       *[{'label': val, 'value': val} for val in spacex_df['Launch Site'].unique()]],
                                              value='All',
                                              placeholder='Select a launch site here',
                                              searchable=True),
                                html.Br(),

                                # TASK 2: Add a pie chart to show the total successful launches count for all sites
                                # If a specific launch site was selected, show the Success vs. Failed counts for the site
                                html.Div(dcc.Graph(id='success-pie-chart')),
                                html.Br(),

                                html.P("Payload range (Kg):"),
                                # TASK 3: Add a slider to select payload range
                                dcc.RangeSlider(id='payload-slider',
                                                min=min_payload, max=max_payload,
                                                step=1000, value=[min_payload, max_payload]),

                                # TASK 4: Add a scatter chart to show the correlation between payload and launch success
                                html.Div(dcc.Graph(id='success-payload-scatter-chart')),
                                ])

# TASK 2:
# Add a callback function for `site-dropdown` as input, `success-pie-chart` as output
@app.callback(Output(component_id='success-pie-chart', component_property='figure'),
              Input(component_id='site-dropdown', component_property='value'))
def get_pie_chart(entered_launch_site):
    #plots sucess rates in pie plot

    #copies df to filter down
    filtered_df=spacex_df.copy()

    if entered_launch_site=='All':
        #summarize all of the launch sites
        fig=px.pie(filtered_df, values='class',
                    names='Launch Site',
                    title='Fraction of Successful Launches by Site')
        
    else:
        #shows fraction of failures and successes for specified site
        #filters the df for the specified site
        filtered_df=filtered_df.loc[filtered_df['Launch Site']==entered_launch_site]
        
        fig=px.pie(filtered_df['class'].value_counts(), values='class',
                    names=filtered_df['class'].value_counts().index,
                    title='Fraction of Successful Launches for %s'%(entered_launch_site))

    #returns the figure
    return(fig)



# TASK 4:
# Add a callback function for `site-dropdown` and `payload-slider` as inputs, `success-payload-scatter-chart` as output
@app.callback(Output(component_id='success-payload-scatter-chart', component_property='figure'),
              [Input(component_id='site-dropdown', component_property='value'),
              Input(component_id='payload-slider', component_property='value')])
def get_scatter_chart(entered_launch_site,payload_slider):
    #plots scatter plot of payload on x, class (success true or false) on y, and booster version for color
    #there are often multiple launches at the same payload, and since y is 0 or 1, these points get stacked and
    #it is easy to loose track of them (scatter doesn't have an expansion when you hover over or click like folium does)

    #copies df to filter down
    filtered_df=spacex_df.copy()

    #mass filter
    filtered_df=filtered_df.loc[(filtered_df['Payload Mass (kg)']>=payload_slider[0]) &
                                (filtered_df['Payload Mass (kg)']<=payload_slider[1])]

    if entered_launch_site=='All':
        #summarize all of the launch sites
        fig=px.scatter(filtered_df, x='Payload Mass (kg)', y='class',
                            color='Booster Version Category',
                            title='Payload vs Success for All Sites by Booster Version')
        
    else:
        #shows fraction of failures and successes for specified site
        #filters the df for the specified site
        filtered_df=filtered_df.loc[filtered_df['Launch Site']==entered_launch_site]
        
        fig=px.scatter(filtered_df, x='Payload Mass (kg)', y='class',
                            color='Booster Version Category',
                            title='Payload vs Success at %s by Booster Version'%(entered_launch_site))

    #returns the figure
    return(fig)

# Run the app
if __name__ == '__main__':
    app.run_server()
