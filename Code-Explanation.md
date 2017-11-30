# How the Code Works
_Lisa Pugh & Quentin Lehn_


## Importing and Status Checker
<pre><code>from bs4 import BeautifulSoup
import requests
import pandas as pd
import folium

def getHtml():
    url = "https://en.wikipedia.org/wiki/All-time_Olympic_Games_medal_table"
    response = requests.get(url)
    if response.ok:
        print(response)
    else:
        print("Error:", response.status_code)

getHtml()</code></pre>
> The program begins by importing all of the libraries that will be used.
> Then there is a status checker to determine if the Wikipedia page we will be using is valid page.
> An error will be return with the error type if that Wiki page cannot be loaded.

## Grabbing the Data
<pre><code>url = "https://en.wikipedia.org/wiki/All-time_Olympic_Games_medal_table"
data = pd.read_html(url)

# Gets the title of the Wiki table using BeautifulSoup
def get_title():
    source_code = requests.get('https://en.wikipedia.org/wiki/All-time_Olympic_Games_medal_table')
    soup = BeautifulSoup(source_code.content, "lxml")
    table_title = soup.select('#firstHeading')
    header = table_title[0].text
    return header

# Creates a Dataframe using Webscraping with the Wiki table data
def get_df():
    total_medals = pd.DataFrame()
    total_medals = total_medals.append(data[2], ignore_index=True)
    total_medals = total_medals.drop(total_medals.index[[0,1]])
    total_medals = total_medals.drop(total_medals.index[[-1]])
    del total_medals[1]
    del total_medals[6]
    del total_medals[11]
    total_medals.columns = ['Countries', 'Gold1', 'Silver1', 'Bronze1', 'Summer Total', 'Gold2', 'Silver2', 'Bronze2', 'Winter Total', 'Gold3','Silver3','Bronze3', 'Combined Total']
    return total_medals

# Formats the Dataframe and output it
# Makes a new column using the values
# from 'Countries' without the 3-letter Country Codes
header = "Dataframe #1: " + get_title()
print("{: ^125s}".format(header))
tm = get_df()
country_code_list=[]
for country in tm['Countries'].astype('str'):
    country_code = country.split("(")[0]
    country_code_list.append( country_code.strip() )
se = pd.Series(country_code_list)
tm['Country'] = se.values
tm
</code></pre>
> In this cell, web-scraping with BeautifulSoup is briefly being used to extract the title of the Wiki table to be used.
> The Wiki webpage is being read in at the top.

> get_df() function:

>> A dataframe is being built using the Wiki table located at data[2].
>> Rows and columns from that Wiki table that are not necessary for this program are dropping.
>> Then we assign our own column names to each column.

> In the code body following the get_df() function:

>> A new list <code>country_code_list=[]</code> is created to be filled with the values from the <code>tm['Countries']</code>
>> column. This list will become a column on the <code>country_code_list</code> dataframe. The new column is to list
>> the countries but without their 3-letter Country Code to later be used as a common column to merge with the next dataframe.
>> The dataframe is outputted with its contents.

## Second Dataframe
<pre><code>url2 = "https://developers.google.com/public-data/docs/canonical/countries_csv"
data2 = pd.read_html(url2)

# Creates second dataframe from developers.google.com
def get_df2():
    coords_df = pd.DataFrame()
    coords_df = coords_df.append(data2[0], ignore_index=True)
    coords_df.sample()
    coords_df.columns = ['Code', 'Lat', 'Long', 'Country']
    coords_df = coords_df.drop(coords_df.index[[0,1]])
    return coords_df

# formats title and output first 10 elements of dataframe
print("{: ^50s}".format("Dataframe #2: Country Coordinates"))
cdf = get_df2()
cdf.head(10)</code></pre>
> This cell creates the next dataframe from a different webpage to get the list of countries including their
> coordinates and country names.
> This dataframe is then outputted after labeling the columns and dropping the rows we did not need to display.

## Merge Two Dataframes
<pre><code>
# Merges both dataframes, formats header, then outputs both
combined_df = pd.merge(left=tm, right=cdf, how='inner', left_on='Country', right_on='Country')
print("{: ^125s}".format("Combined Dataframes"))
combined_df</code></pre>
> Both Dataframes are merge, combining the 'Country' column that both dataframes share Country name values in.
> The new combined dataframe is outputted with a formatted title above it.

# Medal Counter
<pre><code>#Quit Function
def quit():
    print("Exiting All-Time Olympic Medal Counter...")

#Main Program
print("All-Time Olympic Medal Counter")

while True:
    combined_df = pd.merge(left=tm, right=cdf, how='inner', left_on='Country', right_on='Country')
    print('=' * 55)
    season_type = input("Please enter (summer/winter/both) or quit: ")
    if season_type.lower() == 'quit':
        quit()
        break
    elif season_type.lower() == 'summer':
        medal_type = input("Please enter (bronze/silver/gold) or quit: ")
        print("")
        if medal_type.lower() == 'quit':
            quit()
            break
        elif medal_type.lower() == 'bronze':
            del combined_df['Countries'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Bronze1'] = combined_df['Bronze1'].astype('int')
            sb = combined_df.sort_values(['Bronze1'], ascending=False).head(5)
            print('All-Time Bronze Medals Awarded in Summer Olympics')
            print(sb.to_string(index=False, header=False))
        elif medal_type.lower() == 'silver':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Silver1'] = combined_df['Silver1'].astype('int')
            ss = combined_df.sort_values(['Silver1'], ascending=False).head(5)
            print('All-Time Silver Medals Awarded in Summer Olympics')
            print(ss.to_string(index=False, header=False))
        elif medal_type.lower() == 'gold':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Gold1'] = combined_df['Gold1'].astype('int')
            sg = combined_df.sort_values(['Gold1'], ascending=False).head(5)
            print('All-Time Gold Medals Awarded in Summer Olympics')
            print(sg.to_string(index=False, header=False))
        else:
            print('ERROR:', medal_type,'is not a medal type.')
    elif season_type.lower() == 'winter':
        medal_type = input("Please enter (bronze/silver/gold) or quit: ")
        print("")
        if medal_type.lower() == 'quit':
            quit()
            break
        elif medal_type.lower() == 'bronze':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Bronze2'] = combined_df['Bronze2'].astype('int')
            wb = combined_df.sort_values(['Bronze2'], ascending=False).head(5)
            print('All-Time Bronze Medals Awarded in Winter Olympics')
            print(wb.to_string(index=False, header=False))            
        elif medal_type.lower() == 'silver':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Silver2'] = combined_df['Silver2'].astype('int')
            ws = combined_df.sort_values(['Silver2'], ascending=False).head(5)
            print('All-Time Silver Medals Awarded in Winter Olympics')
            print(ws.to_string(index=False, header=False)) 
        elif medal_type.lower() == 'gold':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Gold2'] = combined_df['Gold2'].astype('int')
            wg = combined_df.sort_values(['Gold2'], ascending=False).head(5)
            print('All-Time Gold Medals Awarded in Winter Olympics')
            print(wg.to_string(index=False, header=False)) 
        else:
            print('ERROR:', medal_type,'is not a medal type.')
    elif season_type.lower() == 'both':
        medal_type = input("Please enter (bronze/silver/gold) or quit: ")
        print("")
        if medal_type.lower() == 'quit':
            quit()
            break
        elif medal_type.lower() == 'bronze':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Bronze3'] = combined_df['Bronze3'].astype('int')
            bb = combined_df.sort_values(['Bronze3'], ascending=False).head(5)
            print('All-Time Bronze Medals Awarded in the Olympics')
            print(bb.to_string(index=False, header=False))
        elif medal_type.lower() == 'silver':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Bronze3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Silver3'] = combined_df['Silver3'].astype('int')
            bs = combined_df.sort_values(['Silver3'], ascending=False).head(5)
            print('All-Time Silver Medals Awarded in the Olympics')
            print(bs.to_string(index=False, header=False))
        elif medal_type.lower() == 'gold':
            del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Bronze3'], combined_df['Silver3'], combined_df['Combined Total'], combined_df['Lat'], combined_df['Long']
            combined_df['Gold3'] = combined_df['Gold3'].astype('int')
            bg = combined_df.sort_values(['Gold3'], ascending=False).head(5)
            print('All-Time Gold Medals Awarded in the Olympics')
            print(bg.to_string(index=False, header=False))
        else:
            print('ERROR:', medal_type,'is not a medal type.')
    else:
        print('ERROR:', season_type,'is not a season type.')</pre></code>
> This is the actual main section of the program that involves user interaction and input.
> The program starts with printing the program title then progresses to a loop.
> The user can run the program as many times as desired until he or she enters 'quit' then the loop breaks.
> The program first prompts the user for an input of 4 valid options: summer/winter/both/quit.
> If the user enters anything else, the program will return an error message prompting the user to try again.
> Once summer/winter/both is entered, the program then prompts the user for a type of medal: bronze/silver/gold.
> Again, if the user enters quit- the program exits and the loop breaks. If the user enters anything other than the 
> given options of bronze/silver/gold/quit, the program will return an error message prompting the user to start over.
> When a valid olympic type and medal type are entered, the program will remove all of the columns of the new combined dataframe
> except for the column of medal counts that has been specified along with the country and country code. For example:
<pre><code>Please enter (summer/winter/both) or quit: summer
Please enter (bronze/silver/gold) or quit: bronze

All-Time Bronze Medals Awarded in Summer Olympics
705  United States  US
262         France  FR
232        Germany  DE
193          Italy  IT
187      Australia  AU</pre></code>
> The user entered <code>summer</code> then <code>bronze</code>, so the program returned the dataframe with the highest counts of
> bronze medals awarded to countries during the Summer Olympics. Only three columns are displayed while the rest are eliminated
> because they are not pertaining to this particular run of the program.
> The program then allows the user to go again and pull up other information (or the same information again) until they enter 'quit'.

# Mapping the Top 5 Overall Countries With Most Medals
<pre><code>
# Sorts 'Combined Total' column as integers
# Outputs it without row index or column titles
combined_df = pd.merge(left=tm, right=cdf, how='inner', left_on='Country', right_on='Country')
del combined_df['Countries'], combined_df['Bronze1'], combined_df['Silver1'], combined_df['Gold1'], combined_df['Summer Total'], combined_df['Bronze2'], combined_df['Silver2'], combined_df['Gold2'], combined_df['Winter Total'], combined_df['Gold3'], combined_df['Silver3'], combined_df['Bronze3']
combined_df['Combined Total'] = combined_df['Combined Total'].astype('int')
sorted_total = combined_df.sort_values(['Combined Total'], ascending=False).head(5)
print('All-Time Medals Awarded in the Olympics')
print(sorted_total.to_string(index=False, header=False))

# Creates Folium Map of World with 5 values from 'Combined Total'
CENTER_WORLD = (0,0)
country_geojson = 'world-countries.json'
map = folium.Map(location=CENTER_WORLD, zoom_start=2, tiles='Open Street Map')
for row in sorted_total.sample(5).to_records():
    pos = (row['Lat'],row['Long'])
    marker = folium.Marker(location=pos, 
                    popup="Medal Count: %s, %s (%s)" % (row['Combined Total'], row['Country'], row['Code'])
                          )
    map.add_child(marker)
map
All-Time Medals Awarded in the Olympics
2803  United States  US   37.09024  -95.712891
 824        Germany  DE  51.165691   10.451526
 824         France  FR  46.227638    2.213749
 691          Italy  IT   41.87194    12.56738
 638         Sweden  SE  60.128161   18.643501
</pre></code>
> The last part of the program uses Folium and a json of the world countries to create a world map with
> pin-points of the countries with the most total medals overall (in both the winter and summer olympics combined).
> The user can interact with the map. Each marker can be clicked on to reveal more information.
> The map is not appearing on GitHub but is visible when run using Jupyter-Notebook.


