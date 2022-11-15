## World-electricity-analysis
* Analysis of electricity utilization by the percent of population.
* Generation of electricity by different sources (coal, nuclear, oil, renewable, etc).
* Part of electricity get lost due to distributiion and maintainance loss.

This includes all countries and group of countries.

# Data collection:- 
Data was collected from the [world bank website](https://data.worldbank.org/indicator/EG.ELC.ACCS.ZS), i.e. electricity related data like generation (coal, oil, renewable and nuclear), utilization and distribution loss by different segments of population like; urban, rural and whole population.

# Data Preprocessing:-
Collected data was imported into python with the help of pandas library and then pre-processed here by removal of some unnecessary columns and filling null values.
Exploratory data analysis suggest that data before year 2000 was inaccurate. That means more than 40% data was missing. So, only 2000 - 2020 years data was considered.
## Also we need to import certain python libraries;
    import pandas as pd
    import numpy as np
    import seaborn as sns
    import matplotlib.pyplot as plt
    
    
## Data Cleaning;
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
    
    
# Data analysis:-
data is prepared, now it's ready for analysis. 
So, I imported the cleaned data file to SQL and write some query to get the summarized report so that we can use excel for data visualization and create our own dashboard.

## SQL queries;
## 1.Comparison of access to electricity post 2000s in different countries
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

## 2.Every country’s performance with respect to the world average for every year. 
    -- Total
    select y_country_name, y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from Total as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.region is not Null or A.y_country_name like '%World%') as N
    
    -- Urban
    select y_country_name, y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from urban as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.region is not Null or A.y_country_name like '%World%') as N
    -- Rural
    select y_country_name, y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from rural_access as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.region is not Null or A.y_country_name like '%World%') as N
    
    
## 3. Income wise countries segmentation for electricity utilization 
    -- Total
    select incomegroup, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002, avg(y_2003) as y_2003, avg(y_2004) as y_2004,
    avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007, avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010,
    avg(y_2011) as y_2011, avg(y_2012) as y_2012, avg(y_2013) as y_2013, avg(y_2014) as y_2014, avg(y_2015) as y_2015, avg(y_2016) as y_2016,
    avg(y_2017) as y_2017, avg(y_2018) as y_2018, avg(y_2019) as y_2019, avg(y_2020) as y_2020 from
    (select A.*, B.incomegroup from Total as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by incomegroup
    order by avg(y_2000) Desc


    -- Rural
    select incomegroup, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002, avg(y_2003) as y_2003, avg(y_2004) as y_2004,
    avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007, avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010,
    avg(y_2011) as y_2011, avg(y_2012) as y_2012, avg(y_2013) as y_2013, avg(y_2014) as y_2014, avg(y_2015) as y_2015, avg(y_2016) as y_2016,
    avg(y_2017) as y_2017, avg(y_2018) as y_2018, avg(y_2019) as y_2019, avg(y_2020) as y_2020 from
    (select A.*, B.incomegroup from rural_access as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by incomegroup
    order by avg(y_2000) Desc

    -- Urban
    select incomegroup, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002, avg(y_2003) as y_2003, avg(y_2004) as y_2004,
    avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007, avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010,
    avg(y_2011) as y_2011, avg(y_2012) as y_2012, avg(y_2013) as y_2013, avg(y_2014) as y_2014, avg(y_2015) as y_2015, avg(y_2016) as y_2016,
    avg(y_2017) as y_2017, avg(y_2018) as y_2018, avg(y_2019) as y_2019, avg(y_2020) as y_2020 from
    (select A.*, B.incomegroup from urban as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by incomegroup
    order by avg(y_2000) Desc
    
    
## 4 A chart to depict the increase in the count of countries with greater than 75% electricity access in rural areas across different year
    -- Total
    select SUBSTRING(years, 3,4) as years, sum(new) as country_count from
    (select years, case when electricity > 75 then 1 else 0 end new from
    (select A.*, B.region, B.incomegroup, B.specialnotes from Total as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null) as N
    unpivot
    (electricity for years in (y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) m) o
    group by SUBSTRING(years, 3,4)
    order by SUBSTRING(years, 3,4)

    -- Urban
    select SUBSTRING(years, 3,4) as years, sum(new) as country_count from
    (select years, case when electricity > 75 then 1 else 0 end new from
    (select A.*, B.region, B.incomegroup, B.specialnotes from urban as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null) as N
    unpivot
    (electricity for years in (y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) m) o
    group by SUBSTRING(years, 3,4)
    order by SUBSTRING(years, 3,4)

    -- Rural
    select SUBSTRING(years, 3,4) as years, sum(new) as country_count from
    (select years, case when electricity > 75 then 1 else 0 end new from
    (select A.*, B.region, B.incomegroup, B.specialnotes from rural_access as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null) as N
    unpivot
    (electricity for years in (y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020)) m) o
    group by SUBSTRING(years, 3,4)
    order by SUBSTRING(years, 3,4)
    

## 5 A way/KPI to present the evolution of nuclear power presence grouped by Region and income group.
    -- income-wise electricity utilization by nuclear energy
    select incomegroup, avg(y_1990) as y_1990, avg(y_1991) as y_1991, avg(y_1992) as y_1992, avg(y_1993) as y_1993,
    avg(y_1994) as y_1994, avg(y_1995) as y_1995, avg(y_1996) as y_1996, avg(y_1997) as y_1997, avg(y_1998) as y_1998,
    avg(y_1999) as y_1999, avg(y_2000) as y_2000, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002,
    avg(y_2003) as y_2003, avg(y_2004) as y_2004, avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007,
    avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010, avg(y_2011) as y_2011, avg(y_2012) as y_2012,
    avg(y_2013) as y_2013, avg(y_2014) as y_2014 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from nuclear as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by incomegroup
    order by avg(y_2000) Desc
    
    -- region-wise electricity utilization by nuclear energy
    select region, avg(y_1990) as y_1990, avg(y_1991) as y_1991, avg(y_1992) as y_1992, avg(y_1993) as y_1993,
    avg(y_1994) as y_1994, avg(y_1995) as y_1995, avg(y_1996) as y_1996, avg(y_1997) as y_1997, avg(y_1998) as y_1998,
    avg(y_1999) as y_1999, avg(y_2000) as y_2000, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002,
    avg(y_2003) as y_2003, avg(y_2004) as y_2004, avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007,
    avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010, avg(y_2011) as y_2011, avg(y_2012) as y_2012,
    avg(y_2013) as y_2013, avg(y_2014) as y_2014 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from nuclear as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by region
    order by avg(y_2000) Desc
    
    -- income-wise electricity utilization by oil
    select incomegroup, avg(y_1990) as y_1990, avg(y_1991) as y_1991, avg(y_1992) as y_1992, avg(y_1993) as y_1993,
    avg(y_1994) as y_1994, avg(y_1995) as y_1995, avg(y_1996) as y_1996, avg(y_1997) as y_1997, avg(y_1998) as y_1998,
    avg(y_1999) as y_1999, avg(y_2000) as y_2000, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002,
    avg(y_2003) as y_2003, avg(y_2004) as y_2004, avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007,
    avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010, avg(y_2011) as y_2011, avg(y_2012) as y_2012,
    avg(y_2013) as y_2013, avg(y_2014) as y_2014 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from oil as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by incomegroup
    order by avg(y_2000) Desc
    
    -- region-wise electricity utilization by oil
    select region, avg(y_1990) as y_1990, avg(y_1991) as y_1991, avg(y_1992) as y_1992, avg(y_1993) as y_1993,
    avg(y_1994) as y_1994, avg(y_1995) as y_1995, avg(y_1996) as y_1996, avg(y_1997) as y_1997, avg(y_1998) as y_1998,
    avg(y_1999) as y_1999, avg(y_2000) as y_2000, avg(y_2000) as y_2000, avg(y_2001) as y_2001, avg(y_2002) as y_2002,
    avg(y_2003) as y_2003, avg(y_2004) as y_2004, avg(y_2005) as y_2005, avg(y_2006) as y_2006, avg(y_2007) as y_2007,
    avg(y_2008) as y_2008, avg(y_2009) as y_2009, avg(y_2010) as y_2010, avg(y_2011) as y_2011, avg(y_2012) as y_2012,
    avg(y_2013) as y_2013, avg(y_2014) as y_2014 from
    (select A.*, B.region, B.incomegroup, B.specialnotes from oil as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null) as N
    where incomegroup is not null
    group by region
    order by avg(y_2000) Desc


## 6 A chart to present the production of electricity across different sources (nuclear, oil, etc.) as a function of time
    select y_country_name, y_indicator_name ,y_1971, y_1972, y_1973, y_1974, y_1975, y_1976, y_1977, y_1978, y_1979, y_1980,y_1981, y_1982, y_1983, y_1984, y_1985, y_1986, y_1987, y_1988, y_1989,
    y_1990,y_1991, y_1992, y_1993, y_1994, y_1995, y_1996, y_1997, y_1998, y_1999,
    y_2000, y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015 from
    (select * from nuclear
    union
    select * from oil
    union
    select *, 8.25 as y_2015 from power_loss) as N
    
## Average percent of the population utilize electricity (groups of countries)
    select A.y_Country_Name, y_2000,
    y_2001, y_2002, y_2003, y_2004, y_2005, y_2006, y_2007, y_2008, y_2009, y_2010,
    y_2011, y_2012, y_2013, y_2014, y_2015, y_2016, y_2017, y_2018, y_2019, y_2020, B.specialnotes from Total as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and region is null
    
    
## new table with all three composite data (rural, urban and total)
    select * into composite from
    (select A.*, B.region, B.incomegroup from rural_access as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null
    union
    select A.*, B.region, B.incomegroup from Urban as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null
    union
    select A.*, B.region, B.incomegroup from Total as A
    full outer join metadata_country as B on A.y_country_code = B.country_code
    where B.tablename is not Null and B.region is not null) as N


# Export the output data from SQL server:-
## For exporting the data from the SQL server which the output of the above SQL query, I've connected my python with SQL server with the help of Pandas and pyodbc libraby;
    import pyodbc
    conn = pyodbc.connect('Driver={SQL Server};' 
                   'Server=PREDATOR\SQLSERVER;' #name of the SQL server, my SQL server name is PREDATOR\SQLSERVER
                   'Database=Project1;' #name of the DATAbase in which the data is stored, I've created the database- Project1 and imported all my datas here.
                   'Trusted_Connection=no;') 
                   
## Once the connection is successful, we can directly write our sql query inside the read_sql of pandas and then export in csv, excel or any other file. Example;
    df = pd.read_sql("select * from Total", conn) #this code help to read the sql query once the connection is established with the help of above code
    df.to_csv("C:/Users/batman/Desktop/total.csv", index = False) #export the output data into csv file from the above output
    df.head() #display the top 5 rows from the table to show the preview of data structure

# As per the summarized report, I've created the dynamic dashboard in MS excel and presentation in ppt format.
## PPT (world electricity annalysis presentation);
   [final ppt.pptx](https://github.com/sahuashish147/World-electricity-analysis/files/10014544/final.ppt.pptx)

## MS excel dashboard;
   [final project.xlsx](https://github.com/sahuashish147/World-electricity-analysis/files/10014547/final.project.xlsx)
   
## SQL query file for data preproccesing;
   [SQL queries.pdf](https://github.com/sahuashish147/World-electricity-analysis/files/10014578/SQL.queries.pdf)


  
