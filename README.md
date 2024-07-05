**DOCUMENTATION OF SQL QUERIES AND EXPLANATION OF LOGICS USED
Description of the database schema and tables used**
This database schema comprises of a table designed to store and manage smartphone data. The table in the schema contains columns that represent different properties of the data being stored. This table stores detailed information about various smartphone models.
 Below is a description of the table and it’s columns:
Table name : Smartphone_cleaned
Columns:
• brand_name (varchar): The brand name of the smartphone.(e.g Apple, Samsung)
 • model (varchar): The model name or number of the smartphone. 
• price (int): The price of the smartphone in RUPEES. 
• rating (float): The rating of the smartphone out of 100. 
• has_5g (Bit) : Indicates whether the smartphone supports 5G connectivity (TRUE/FALSE). 
• has_nfc (Bit): Indicates whether the smartphone has NFC capability (TRUE/FALSE). 
• has_ir_blaster (Bit): Indicates whether the smartphone has an IR blaster (TRUE/FALSE). 
• processor_name (varchar): The name of the processor used in the smartphone. 
• processor_brand (varchar): The brand of the processor used in the smartphone. 
• num_cores (varchar): The number of cores in the processor. 
• processor_speed (float): The speed of the processor in GHz. 
• battery_capacity (int): The battery capacity of the smartphone in mAh. 
• fast_charging (int): The fast-charging capability of the smartphone in watts. 
• ram_capacity (int): The RAM capacity of the smartphone in GB. 
• internal_memory (int): The internal storage capacity of the smartphone in GB. 
• refresh_rate (int): The refresh rate of the smartphone display in Hz.
 • resolution (int): The resolution of the smartphone display. 
• num_rear_cameras (int): The number of rear cameras on the smartphone.
 • num_front_cameras (int): The number of front cameras on the smartphone. 
• os (varchar): The operating system used in the smartphone. 
primary_camera_rear (varchar): The resolution of the primary rear camera in megapixels.
 • primary_camera_front (varchar): The resolution of the primary front camera in megapixels.
 • extended_memory (bit): The capacity of extended memory (e.g., microSD card support) in the smartphone, if applicable

**Explanation of SQL query logic and any data transformations performed**
**1.	Data cleaning**
**a)	A copy of the table was made and named “Smartphone_new” to avoid tampering with the original table, it was done using the query below :**
Select *
into Smartphone_new
from smartphone_cleaned_v2
**b)	A check was conducted to get rid of duplicates**
With SmartPhoneCTE AS (
Select *,
ROW_NUMBER() OVER(
	PARTITION BY model
	ORDER BY 
	brand_name) row_num
from Smartphone_new 
)
SELECT *
from SmartPhoneCTE
where row_num > 1
**c)	Checking for nulls**
SELECT
    SUM(CASE WHEN brand_name IS NULL THEN 1 ELSE 0 END) AS    brand_name_nulls,
   SUM(CASE WHEN model IS NULL THEN 1 ELSE 0 END) AS model_nulls,
   SUM(CASE WHEN price IS NULL THEN 1 ELSE 0 END) AS price_nulls
FROM Smartphone_new

**d)	The new table was altered by updating the null values in the OS column with the two distinct OS as introduced “android” and “ios”**

 **--Dealing with nulls in OS columns**
 select distinct(brand_name), os
from Smartphone_new
where os is null

**--Filling the nulls in the OS column with thier respective Operating system**
update Smartphone_new
set os = case when os is null and brand_name != 'apple' then 'android'
		else 'ios'
		end

update Smartphone_new
	set os = case when brand_name = 'apple' and os = 'ios' then 'ios'
	else 'android'
	end

  		
**e)	An altercation was done on the “Processor_speed” column by rounding up the values into 2 decimal place to have a uniform form of values in the column.**

 update Smartphone_new
set processor_speed = ROUND((processor_speed),2)

**f)	Replacement of nulls contained in the “primary front camera” and the “num of front camera” as both have the same number of nulls indicating that the phone models have no front camera, so it was replaced with ‘0’**

update Smartphone_new
set primary_camera_front = case when primary_camera_front is null then '0'
								else primary_camera_front
								end
update Smartphone_new
		set num_front_cameras = case when num_front_cameras is null then '0'
								else primary_camera_front
								end
g)	Updating the nulls in the rating column with the average rating of each brandnames…. E.g if the average of Apple brand is ‘77’, then all apple brands having nulls as their rating will be ‘77’. It was effected by firstly creating a Temporary table using the WITH_CTE for storing the average rating and then an update was carried out on the ‘rating’ column using the query below

WITH CTE_AVG_RATING AS (
  		  SELECT 
      	 	 brand_name, 
     		   AVG(rating) AS avg_rating
    		FROM 
       	 Smartphone_new
   		 WHERE 
       	 rating is not null
   		 GROUP BY 
     		   brand_name
)
UPDATE Smartphone_new
SET rating = CASE 
                WHEN rating IS NULL THEN CTE.avg_rating
                ELSE rating
             END
FROM Smartphone_new AS smart
JOIN CTE_AVG_RATING AS CTE
ON smart.brand_name = CTE.brand_name

h)	Populating nulls in the other columns like the processor brand, processor name and num cores with unknown. The data wasn’t provided and the nulls couldn’t be ignored. Since the Processor brand wasn’t provided, it was normal to have the same number of nulls for both the processor brand and processor name. below is the sql query to replace them

update Smartphone_new
set processor_brand = case when processor_brand is null then 'unknown'
						else processor_brand
						end,
		 processor_name = case when processor_brand is null then 'unknown'
					else processor_name
					end,
			processor_speed = case when processor_speed is null then '0'
						else processor_speed
					end,
		num_cores = case when num_cores is null then 'unknown'
				else num_cores
					end,
		battery_capacity = case when battery_capacity is null then ' '
						else battery_capacity
							end,
		internal_memory = case when internal_memory is null then ' '
					else internal_memory
					end,
		rating = case when rating is null then ' '
			else rating
			end

**i)	Replacing the ‘1’ and ‘0’ with ‘Y’ for Yes and ‘N’ for No respectively in the “has_5g”, “has_nfc”, “has_ir_blaster” columns.**

a)	The first step was to alter the table by creating new columns for the three columns… This was done using the query below
Alter table Smartphone_new
add Uses_5g varchar,
				Ir_blaster varchar,
				Near_field_comm varchar

b)	The second step was to update the new columns with values from the old columns transforming those values to either “Y” or “N” with respect to the old columns.

update Smartphone_new
set Uses_5g = case when has_5g = '1' then 'Y'
				else 'N'
				end,
				Ir_blaster = case when has_ir_blaster = '1' then 'Y'
					else 'N'
					end,
			Near_field_comm = case when has_nfc = '1' then 'Y'
						else 'N'
						end

c)	The old columns were deleted from the new table.

alter table Smartphone_new
 		    drop column has_5g,
					has_ir_blaster,
	has_nfc


**2) DATA ANALYSIS**

a)	Top 10 brand names by the average rating. As shown below, the AVG function was used to calculate the average rating of each brand and the round function was to make sure the values are uniform. The “as” was used to create a new name for the new column containing the new average values for each brand. The group by function categorized the brand name uniquely.

Select top 10 (brand_name), round(AVG(rating),0) as AverageRating
from Smartphone_new
group by brand_name
order by AverageRating desc

**b)	Top 20 Average price of phones for each brand**

Generating the data for the top 20 brand by the average prices of each 
brand using the AVG function and the ORDER BY function does the sorting of the result. Desc which means the resulting table will be sorted by the Average price ranking from the highest to the lowest value. 

Select top 20 (brand_name), AVG(price) as AveragePrice
from  Smartphone_new
group by brand_name
order by AveragePrice desc

**c)	Top 10 Brand names with the highest number of phones with 5g properties**

Count – This counts the number of brands that supports the use of 5g.
Where – the where function serves as filter function returning the brands that only supports the use of 5g


select top 10 (brand_name), count(Uses_5g) as count_of_5g
from Smartphone_new
where Uses_5g = 'Y'
group by brand_name
order by count_of_5g desc

****d)	What is the most common processor brand ****

Select top 1 processor_brand, COUNT(*) AS count_of_brand
from Smartphone_new
group by processor_brand
order by count_of_brand DESC
Top 1 – returns the most common processor brand according to the dataset
Count * - counts the number of brand uniquely
AS – The AS creates an alias for the new table 



**e)How many mobile phones runs on android **

Select count(model) as count_of_phones, os
from Smartphone_new
where os = 'android'
group by os

Select – Returns the selected columns to return
From – specifies the database and table to generate the result from.
Where – filters the data to return from the database. In the query above, the models needed was the models that runs on the android os thus the where condition that was introduced into the query.


**f)Find smartphones with a refresh rate of at least 120 Hz and a ram capacity of 8 GB or more.**

Select model, refresh_rate, ram_capacity
From Smartphone_new
Where refresh_rate >= 120 AND ram_capacity >= 8

Select – Allows you select the columns to return
From – specifies the database and table to generate the result from.
Where – filters the data to return from the database. In the query above, the condition to be satisfied was returning smartphone models that has a refresh rate greater than or equal to 120hz and ram capacity greater than or equal to 8gb ram.


**g)	 Total number of smartphones per brand**

select brand_name, COUNT(model) AS num_of_models
from Smartphone_new
group by brand_name
order by num_of_models desc

Select – Returns the columns to return
From – specifies the database and table to generate the result from.
Group by – groups the brand uniquely
Order by – Sorts the count. This function was introduced in order to get the brand with the highest count at the top followed by the subsequent brands according to their values.


**h)	 The phone with the fastest processor_speed**

Select model, processor_speed 
From Smartphone_new
order by processor_speed desc

Select – Returns the the columns to return
From – specifies the database and table to generate the result from.
Order by – Sorts the ‘Processor speed’ column by ranking from the largest to the smallest value


**i)	 Brands that have the highest number of phones with rear camera greater than 3**


Select brand_name, count(model) as count_of_model
from Smartphone_new
where num_rear_cameras > 3
group by brand_name
order by count_of_model desc

Select – Returns the columns of interest.
Count – counts the number of values present in the dataset. In this case the count was initiated to return the number of mobile models that supports the Where condition grouping each brand name uniquely.
Where – Works as a filter for specific conditions to be met.
Order by – sorts the data according to the way either by descending or ascending order. In this case, the counts was sorted in descending order to return the brands that has the highest count according to the condition.


**j)	Brands with the highest number of mobile phones that does not have Ir blaster but has Near field communication property**


Select brand_name, count(brand_name) as num_of_brand
from Smartphone_new
where Ir_blaster = 'N' and Near_field_comm = 'Y'
group by brand_name
order by num_of_brand desc

Select – Allows you select the columns to return
From – specifies the database and table to generate the result from.
Where – filters the data to return from the database. In the below   query, the brand name that does not have the Ir_blaster but does have the Near field comms. The “and” combines two or more conditions to be satisfied in order to return the desired data.
Group by – groups the brand uniquely


**k)	 Checking to see the top 5 brands with the fastest charging rate**


    select top 5 brand_name, MAX(fast_charging) as Fastest_charging_brand
from Smartphone_new
group by brand_name
order by Fastest_charging_brand desc

Select – returns the desired columns.
Top 5 – returns the best 5 brands with the highest charging rate.
From – specifies the database and table to generate the result from.
Max – this function returns the maximum value contained in the column as in this case the brand with the highest charging rate.
Group by – groups the brand uniquely
Order by – Arranges the results in descending order(Desc) to speicify the values at the top


**l)	checking for processor brand that supports extended memory up to 1TB and has processor_speed greater than or equal to 2.5**

select processor_brand, count(processor_brand) as num_of_pro_brands
from Smartphone_new
where processor_speed > 2.5 and extended_memory >= '1TB'
group by processor_brand
order by num_of_pro_brands desc


Select – Allows you select the columns to return
From – specifies the database and table to generate the result from.
Where – filters the data to return from the database. In the below   query, the mobile phones with processor speed greater than or equal to 2.5 and also supports extended memory up to 1TB
Group by – groups the brand uniquely



**m) Checking for brands with battery capacity greater than '5500' and has fast charging < '200'**


select brand_name, count(battery_capacity) as count_of_brands
from Smartphone_new
where fast_charging < 200 and battery_capacity > 5500
group by brand_name
order by count_of_brands desc


Select – Returns selected columns 
From – specifies the database and table to generate the result from.
Where – filters the data to return from the database. In the below   query, the mobile phones with battery capacity greater than 5500 and fast charging less than 200kw.
Group by – groups the brand uniquely.




NOTE

Dealing with the average was something I came up with and the right thing should have been assuming it was a client’s data to reach out and verify if it could be removed or corrected. Thank you.



