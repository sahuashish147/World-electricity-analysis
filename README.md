## World-electricity-analysis
* Analysis of electricity utilization by the percent of population.
* Generation of electricity by different sources (coal, nuclear, oil, renewable, etc).
* Part of electricity get lost due to distributiion and maintainance loss.

This includes all countries and group of countries.

## Data collection:- 
Data was collected from the world bank website, i.e. electricity related data like generation (coal, oil, renewable and nuclear), utilization and distribution loss by different segments of population like; urban, rural and whole population.

## Data Preprocessing:-
Collected data was imported into python with the help of pandas library and then pre-processed here by removal of some unnecessary columns and filling null values.
Exploratory data analysis suggest that data before year 2000 was inaccurate. That means more than 40% data was missing. So, only 2000 - 2020 years data was considered.
Also we need to import certain python libraries;
    import pandas as pd
    import numpy as np
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    
Data Cleaning;
    #this code is only for electricity utization by rural population
    rural_access = pd.read_json("C:/Users/batman/Desktop/my territory/Projects/complete data analysis/world_bank_electricity_report/access_to_elec_rural__world/API_EG.ELC.ACCS.RU.ZS_DS2_en_csv_v2_4364070.json") #path for the downloaded data  
    rural_access.drop(0, inplace =True)
    list1 = rural_access.iloc[0,:].to_list() 
    new_lst = []
    for i in list1:
        new_lst.append("y_"+str(i)) #year column name has integer datatype which is readable in SQL, so here I'm change the column name by appending y_.
    rural_access.columns = new_lst
    rural_access.drop(1, inplace = True)
    rural_access.dropna(axis =1, thresh = 1, inplace = True)
    rural_access.drop_duplicates(inplace = True)   #drop dupplicate values
    rural_access.drop("y_", axis =1, inplace = True)
    rural_access.replace("",np.nan, inplace = True)   #replace the space with null values
    rural_access.to_csv("C:/Users/batman/Desktop/my territory/Projects/complete data analysis/exported from python/rural_access.csv",index = False) 
    #exported cleaned data
    rural_access.head() #preview top 5 rows to check the table structure
    #same can be done for urban, total population, electricity generation and loss. IPYNB file also attached
    
    #To check the null values;
    null = rural_access.isnull().sum()[4:]
    plt.subplots(figsize=(17,9.5))
    sns.barplot( x= null.values, y = null.index, palette = 'cool')
    
    
Data analysis:-
data is prepared, now it's ready for analysis. 
So, I imported the cleaned data file to SQL and write some query to get the summarized report so that we can use excel for data visualization and create our own dashboard.

SQL queries;
Comparison of access to electricity post 2000s in different countries
--Rural
  select y_country_name, avg(electricity) as rural_average_usage from
  (select y_country_name, electricity from rural_access
  unpivot (electricity for years in (y_2000, y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
  y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) as N) as M
  group by y_country_name

-- Urban
  select y_country_name, avg(electricity) as urban_average_usage from
  (select y_country_name, electricity from urban
  unpivot (electricity for years in (y_2000, y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
  y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) as N) as M
  group by y_country_name

-- Total
  select y_country_name, avg(electricity) as total_average_usage from
  (select y_country_name, electricity from total
  unpivot (electricity for years in (y_2000, y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
  y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) as N) as M
  group by y_country_name
