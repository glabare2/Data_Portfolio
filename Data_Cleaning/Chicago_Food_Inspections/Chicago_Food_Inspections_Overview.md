<h1>Chicago Food Inspections Dataset</h1>
<h4>Project Report</h4>
<h4>IS 537: Theory and Practice of Data Cleaning</h4>
<h4>Peter Tubbs (pmtubbs2)</h4>
<h4>Gabrielle LaBare (glabare2)</h4>
<h4>Raw dataset: https://uofi.box.com/s/884d0m5ch8mcjlhbj3a83zwe00mxv02o</h4>
<h4>Cleaned dataset: https://uofi.box.com/s/ba11f908w2ti2gx4mjom1jff4n2cavej</h4>

<h2>Overview and Initial Assessment of the Dataset</h2>

Project Overview
For our project we chose to use a fictitious scenario to supply our use cases. Peter Tubbs and Gabby LaBare are two administrators for the Big Data 2022 conference at the Fairmont Chicago Hotel in Millenium Park. In addition to being administrators, we will also be giving a presentation on data cleaning. To save time and labor we decided to integrate some of their admin work into their presentation. During the week-long conference, only nine meals will be provided to attendees, so the administrators are obligated to provide a curated list of restaurants in the area surrounding the hotel. We want to ensure that our attendees are given the best possible options. To this end, we selected a dataset containing the Chicago Department of Public Health’s restaurant and food service inspection reports with the assumption that it could be used to seek out the safest restaurants in the vicinity of the conference venue.

Use Cases
We settled on the following use cases for our project:
Produce a report and Google map of the “best” (or, at any rate, safest) restaurants for conference-goers in roughly a three mile radius surrounding the Fairmont Hotel.
The goal of the data cleaning presentation will be to produce a report of restaurants to avoid for our conference presentation given the data we have cleaned in OpenRefine. 
Prepare dataset for use in future projects via a searchable web interface. 

Initial Dataset Assessment
Our dataset was collected from the City of Chicago’s open data portal. The current, up-to-date version of the dataset can be viewed and downloaded here. The Department of Public Health for the City of Chicago owns this dataset. The City of Chicago updates many Data Portal datasets daily (“About”), but the Chicago Food Inspections list is updated every assFriday. Records in this dataset represent every inspection conducted between 2010 and the present, and include basic information about the establishment, its location, and the outcome of the inspection. The version of the dataset we used for this project was collected at 9:44 p.m. on March 29, 2021 (City of Chicago, 2019). There are seventeen columns in this dataset: Inspection ID, DBA Name, AKA Name, License #, Facility Type, Risk, Address, City, State, Zip, Inspection Date, Inspection Type, Results, Violations, Latitude, Longitude, and Location. Overall, the dataset contains 217,846 rows, each representing a unique inspection event. 
Our initial inspection determined it would be feasible to use the dataset to differentiate establishments with positive inspection outcomes with roughly a three mile radius of the Fairmont Hotel.
During our initial assessment of the data we chose to use OpenRefine to manipulate the primitive operators (clustering, parsing, etc.) and planned to subsequently use SQL to check the domain, entity, and referential integrity. We planned to use OpenRefine to cluster and merge variants of descriptor names in order to more accurately represent the data while preserving the intentions of the inspectors. Finally we planned to use SQL to select all pertinent information to display our results, which ment purging or filtering records not relevant to our project that did not fit the scope of our use cases. 
Below are two tables that outline our assessment of data quality issues found in each attribute in the dataset and how we initially planned to go about cleaning it. The first table consists of issues that can be cleaned using OpenRefine, the second contains issues that will be addressed using SQL.

Table 1. Summary of OpenRefine data assessment and cleaning steps by attribute.
|Attributes| OpenRefine assessment steps| OpenRefine cleaning steps|
| :------- | :------- | :-------|
|Inspection ID | Transform to number, create numeric facet | None needed|
|DBA Name | Transform to text | Transform to uppercase |
|AKA Name | Transform to text | Transform to uppercase|
|License # | Transform to number, create numeric facet | None needed|
|Facility Type | Transform to text, create text facet | Create text facet; use clustering to merge values. Transform to uppercase|
|Risk | Transform to text, create text facet | Split into two rows labeled Risk Level and Risk, remove parenthesis using regex|
|Address | None needed (refer to narrative for more information) | Transform to uppercase|
|City | Transform to text, create text facet | Create text facet; use clustering to merge values. Transform to uppercase |
|State | transform to text, create text facet | None needed |
|Zip | Transform to number, create numeric facet | None needed|
|Inspection Date | Confirm dates are within expected range of dataset (January 2010 through March 29, 2021, the date the dataset was retrieved)| Text transformation using GREL statement to transform to ISO date: value.todate(dd/MM/yyyy) GREL statement to trim time and time zone (for compatibility with MySQL):
value.substring(0, -7)|
|Inspection Type | Transformation uppercase, text facet- cluster | Transform to uppercase|
|Results | Transform to text, create text facet | None needed|
|Violations | Separate violations into rows using delimiter; separate comments from violations; use text facet to examine violations and comments independently | Prepare for use in a separate table during SQL phase. Separate violations into rows using delimiter; separate comments from violations. See narrative for further detail|
|Latitude |Transform to number, create numeric facet|None needed|
|Longitude|Transform to number, create numeric facet|None needed|
|Location|None needed (refer to narrative for more information)|None needed|


Table 2. Summary of planned SQL data cleaning steps by attribute.
|Attributes|SQL|
| :--- | :--- |
|Inspection ID | Unique, not null, auto increment|
|DBA Name | Not null, has matching License # (1:1)|
|AKA Name |None |
|License # | Not null, has matching DBA Name (1:1) Should be non-zero|
|Facility Type | Not null|
|Risk | Not null|
|Address| Not null|
|City | Not null|
|State | None|
|ZIP |Not null, limit to only City of Chicago ZIP Codes|
|Inspection Date | Not null |
|Inspection Type | Not null |
|Results | Not null |
|Violations| Not null|
|Latitude | Within the boundaries of the City of Chicago|
|Longitude |Within the boundaries of the City of Chicago|
|Location | Should match latitude and longitude|

In summary, we determined the attributes Inspection_ID, Risk, Zip, Longitude, Latitude, and License Number would not require OpenRefine cleaning steps (beyond requiring a few minor transformations to numeric or text facets during the initial assessment). Every attribute other than Inspection ID (which, we assume, serves as the primary key) included null values, so any use case where each record must be completely populated/not missing any values might pose challenges, or might not be feasible. For example, a null longitude or latitude would prevent us from identifying the business’s location, but could be found if we were able to search the map and pinpoint the location on an additional resource. Further details on the steps we took to assess each of these are below. 

**Inspection ID**

Inspection ID was inspected with a numeric facet. No obvious anomalies were detected, and no cleaning steps were planned in OpenRefine.  

**DBA Name and AKA Name**

According to the City of Chicago’s description of the dataset’s elements, DBA Name contains “the legal name of the establishment” and AKA name contains “the name the public would know the establishment as.” Since these values are highly varied and are not expected to conform to any pattern--and are also very likely to be entered by an inspector--the only cleaning step we performed was conversion to uppercase. This was intended to eliminate differences due only to variations in capitalization.

**License #**
We expected every license number to contain a value, though there was no expectation it would be unique within the dataset (as a single establishment could be subjected to multiple inspection events). We also expected the values to be relatively similar and evenly distributed, as we guessed they were assigned serially. After converting the column to the number type, we created a numeric facet, which allowed us to visualize the distribution of values, which made several anomalies visible. First, we noticed there were a large number of records with a license number of 0 (1). Second, there were 16 blank/null values (2). Finally, there were a small number of values that appeared to be far outside the range occupied by the vast majority of the approximately 172,000 values (3). 

According to the City of Chicago, the License #  “is a unique number assigned to the establishment for the purposes of licensing by the Department of Business Affairs and Consumer Protection.” Thus, we would expect a one-to-one relationship between License# and the legal business name held in the DBA name field.

**Facility Type**
Like City, Facility Type contained variations in spelling, punctuation/spacing, and capitalization. Initially, we planned to limit our use to establishments with the RESTAURANT Facility Type. However, after reviewing the values in this column, we determined that there were a large proportion of the 343 Facility Types other than RESTAURANT which would be reasonably interpreted as a type of restaurant by a conference attendee: for example, “BAKERY/RESTAURANT,” “BAR/GRILL,” “RESTAURANT/BAR,” “TAVERN,” etc. As a result, we decided to do a more comprehensive review and cleaning of this attribute. 

**Risk**
The Risk field contained both a numeric risk level (“Risk 1”, “Risk 2,” etc.) and a description of the risk level (“Low,” “High”). Per the City of Chicago’s description, the numeric risk level should correspond to a specific text description (“Risk 1 (High)”). We planned to separate the field in OpenRefine with the intention of subsequently testing to confirm that this pattern was followed in the SQL phase.

**Address**
We initially assumed Address values could be reliably used to determine the location of the business in question. In our use case, this might take the form of a street-based location lookup, map, or retrieving directions from Google Maps. Our initial assumption was that the address data had been drawn from or validated with an external data source with valid street addresses. On closer examination, however, we noticed that the address values contained frequent spelling errors in street names, suggesting that the addresses were entered by hand or transcribed from a handwritten form. For example, in Inspection ID 67738, Michael’s on Main has an address value “8750 W BRYN WAWR AVE”. The correct address is 8750 W Bryn Mawr Ave.

While it might have been possible (though time consuming) to correct street name errors in OpenRefine by splitting the field and using a text facet and clustering, their presence suggests that similar errors and typos will be present in the numeric portion of the address. Since Chicago’s streets often span the entire length or breadth of the city, an incorrect digit could lead to an address miles from the actual location. These errors would be impossible to correct without comparing the address to an external data source, and we concluded this was beyond the scope of our project, and that therefore no further cleaning would be attempted.
As a result, while we retained the addresses for use in our handouts, we decided to rely on the latitude and longitude values to determine location in the Google map we produced for our use case. 

**City**
Examination of a text facet revealed multiple variations in most values in the City column. For instance, the City of Chicago was represented by Chicago (285 rows), chicago (89 rows), CHicago (11 rows), CHICAGOI (3 rows), 312CHICAGO (2 rows), and others (including CHICAGOCHICAGO). Thus, we planned to use a text facet and clustering to merge variants. 

**State**
Examination with a text facet revealed 28 instances of null values in the state field. However, since all establishments are within the state of Illinois, and we were already planning to use latitude and longitude to map our establishments, this was not considered to be a meaningful deficiency in the dataset. We did not plan to conduct any further cleaning. 



**Violations**
The most challenging attribute to assess was Violations. All violations for a given inspection are present in a single cell containing the violation number, description, and the inspector’s comments, if any were included. We initially conducted processing steps in OpenRefine to investigate the violation number, delimiters separating multiple violations, and associated comments. Based on this, we determined cleaning this attribute for our use cases would require similar processing steps, which are detailed in the “Data cleaning with OpenRefine” section.
To inspect the violation codes separately from their comments, we determined that they were separated by a delimiter, which we used to separate the violations into rows. Next, the comments were separated using an invariant prefix (“ - Comments: ”).
 The text of the violations were examined via a text facet, which revealed clean and consistent entries with no apparent data entry errors (e.g. misspellings). This suggests violation codes were derived from a controlled vocabulary of violations, and multiple entries were assembled into a single text string via an automated process. Conversely, there were spelling errors in the comments associated with each violation, which would be expected in data typed in by an inspector. 
For example, in Inspection ID 67738, the number and text of each violation (“34. FLOORS: CONSTRUCTED PER CODE, CLEANED, GOOD REPAIR, COVING INSTALLED, DUST-LESS CLEANING METHODS USED”) is consistent with every other instance of this violation, but the comments contain spelling errors, typos, and other idiosyncrasies (for instance, it appears that “baseboard” was misspelled “basenoard”). Due to this, though we manipulated the data to prepare it for use in MySQL, we did not need to conduct any cleaning operations on the violation codes themselves.


**Date**
In order to examine the date data, we converted the Inspection Date field to ISO using the GREL expression:

value.toDate("dd/MM/yyyy")

Once the Date column had been converted, we created a timeline facet. This allowed us to confirm that the dates present in the dataset were within the stated limits of the dataset (1/1/2010 through March 29, 2021, the date on which the dataset used in our project was downloaded from the City of Chicago’s website). The facet revealed significant dropoff in inspection events in 2020, possibly due to the pandemic and/or a lag in data entry (1). There also did not appear to be any inspection events after July 4, 2020. We were not able to determine if this was a normal lag in data availability, or if it was caused by the pandemic. Based on our examination, we determined no further cleaning steps were necessary beyond converting to ISO format. 

**Results **
Breaking the patterns we saw in previous attributes, which appeared to be entered or transcribed by hand, Results only contained seven values:

Business Not Located (62)
Fail (33534)
No Entry (5148)
Not Ready (1226)
Out of Business (15227)
Pass (101168)
Pass w/ Conditions (16517)

The distribution revealed by a text facet did not reveal any outliers or other potential data quality problems. This suggests that the values were entered via a menu rather than free text entry. We determined no cleaning steps were needed. 


**Latitude and Longitude**

Numeric facets were created to examine the contents of latitude and longitude. 578 records from each value were blanks, but otherwise the facets provided no evidence that there were anomalous values when data was present. No cleaning steps in OpenRefine were deemed necessary.



**Location**

As Location contained a latitude and longitude pair, we concluded no further cleaning was necessary. Instead, we anticipated we would disregard Location, and decided to derive any needed Latitude and Longitude pairs by querying these values directly, as these values can be tested independently. Additionally, querying would give us more flexibility when selecting the order of the values. 



Data cleaning with OpenRefine
In this section we will discuss the steps taken to clean the dataset in detail. Since cleaning operations were chosen with our use cases in mind, we have separated the narrative into sections corresponding to different objectives reflected in our use cases: location, type of establishment, date of inspection, and outcome of inspection. For instance, we hoped to make it possible to use location as a criterion (both for the Google map, and, possibly, future use in a proposed web-based search interface), so we will discuss steps related to this under the heading “Location.” 
In order to prepare the data for SQL, we also used OpenRefine to prepare the dataset to be loaded into two separate tables, one containing all fields other than violations and comments, and a second containing only violations and comments. Since each row in the original dataset represented an inspection event, we planned to name the first table “inspections,” and to name the second table “violations.” The operation history representing the steps taken to clean the data in preparation for use in the inspections table is provided in the file operation_history_inspections.json. The operation history capturing the steps used to prepare the violations data for use in a separate table is represented by the file operation_history_violations.json. These steps are discussed in the “Final OpenRefine steps to prepare dataset for MySQL and operations history” section of the narrative.

Location (City, Address)
	To address the variations noted in the City values, we created a text facet for the City column to cluster and merge values. Clustering was very effective for city names, and since they represented selections from a limited, easily confirmed set of values, we were confident that we were able to locate and merge all variants. 



Though we opted not to attempt to clean the Address data by clustering, and did not have an application for it in the use cases we actively completed, we assumed that it might be useful in our proposed future web search tool. Due to this we decided to convert to uppercase in the interest of eliminating variability introduced by inconsistent capitalization. As most values were originally present in upper case, we were confident that doing so did not result in the loss of any meaningful information.

Type of Establishment (Facility Type)
When conducting clustering on the Facility Type column, we merged values only when there was only a variation in spelling, punctuation, or capitalization, or when the alternative representations had an unambiguously identical meaning. For example, we merged RESTAURANT/GROCERY STORE and RESTAURANT/GROCERY. 
Conversely, we did not attempt to interpret values where alternatives could convey a meaningful distinction between facility type. For example, we did not merge TAVERN, TAVERN/RESTAURANT and TAVERN/LIQUOR. Many similar clusters of similar but not identical descriptions were present. Additionally, some entries defied interpretation (for example, CAT/LIQUOR or SUMMER FEEDING). Unfortunately, despite cleaning steps, we were not confident enough in our ability to clean and normalize the values written by inspectors, and as a result these data may not be useful as keywords or search terms in our anticipated use as the data underlying a web- based search tool.
While completing this cleaning task, we clustered in three rounds, first using metaphone3, followed by cologne-phonetic, which found several appropriate clusters more successfully than metaphone3.
Like other text-based fields, we converted all Facility Type values to uppercase. 





Date of Inspection (Inspection Date)
Since this dataset encompasses inspections conducted over a relatively long period of time, it will be necessary to determine whether information about inspections and violations are current. After all, an establishment with egregious violations in 2011 may be perfectly safe and compliant in 2021.
In order to facilitate queries that include a date, we converted the Inspection Date field to ISO using the GREL expression:

value.toDate("dd/MM/yyyy")

This is the same expression we used during the assessment of the dataset. Though we had determined the date field did not require further cleaning, we subsequently determined that the ISO format yielded by the GREL expression was the cause of an error when loaded by MySQL. Specifically, because time and time zone information was not present in the original dataset, the GREL statement appended “blank” time and time zone information As a result, we determined it was necessary to use a second GREL statement to remove “T00:00:00Z” from each Inspection Date string:

value.substring(0, -7)

This statement returns a substring eight characters long, beginning with the first character of the string. This retains four digits for the year, one dash, two digits for the month, a second dash, and two digits for the day. More information on the date is presented in the “Loading the data into SQL tables” section. 

Result of inspection (Violations)
The Violations value for each record contains zero to many violations. Each individual violation name begins with a number, and is followed by a comment from the inspector preceded by the text “- Comments:”. Subsequent violations, if any exist, follow the same pattern, and are separated by a pipe character. 
Initially, we planned to separate individual violations and their comments to allow violation-level queries by either by separating each violation into individual rows (creating multi-row records for each Inspection ID), or by using regex to reduce the complete violation listing to code numbers, which could be corresponded to a data dictionary provided by the City of Chicago.
The maximum number of violations listed for any single inspection ID is twenty three. However, during inspection of this attribute, we also determined that some records included multiple instances of a single violation code with distinct comments. It is unclear if this is an artifact of the entry system (which could be the case if re-entering the violation code was the only way to append additional comment to the violation), or if each represents a unique violation. 
However, as we investigated further, we uncovered the first major challenge in cleaning this dataset. On July 18, 2018, the Chicago Department of Public Health’s Food Protection updated its food inspection violation categories, issuing the following statement (City of Chicago, 2018):

Structurally, this will not affect the Food Inspections dataset. The same columns will exist and continue to be populated. The Violations column will still contain the violation number, description, and comments; with separate violations delimited by the pipe character with a space on each side (i.e., “ | ”). However, the actual violations will change substantially so users are advised to test and potentially modify any processes that depend on this column.

Finally, we used OpenRefine’s SQL Exporter tool to prepare the data for the next step. We observed that data types we assigned in OpenRefine were not automatically honored when we set up the options for the SQL export. We selected the appropriate SQL Type for each field.  



Final OpenRefine steps to prepare data for MySQL and operations history
In the dataset provided by the City of Chicago, each row represented a unique inspection event. Since the Violations value contained a list of every violation found during the inspection event (from zero to a maximum of twenty three violations), and each had an associated comment, we decided it would be appropriate to build a separate table to hold violations and comments when we loaded the dataset into MySQL. This would allow us to address violation and comment pairs individually. 
To prepare, we used OpenRefine to separate each row into a multi-row record, with one row for each violation. Next, we separated the violations and their comments into two columns (Violation ID and Violation Comment). 
	The steps we followed were similar but not identical to the steps we took during the assessment phase. To separate the violations, “Edit Cells > Split multi-valued cells...” was used with the delimiter space-pipe-space (“ | “). At this point, “Collapse consecutive whitespace” was applied, as some records contained multiple spaces within the comment text (including spaces before the first word of the comment).



	To separate violations and comments, we applied the “Split into several columns...” command, using the invariant prefix “ - Comments: “ as a delimiter. This prefix was present before each comment. 



At this point, the process diverged from the steps taken during assessment. To prepare the violations for use as a separate table in MySQL, we used OpenRefine’s Copy Down command to copy the Inspection ID and License # value into each row in the multi-row record. Finally, we removed all columns other than the License #, Inspection ID, Violation ID, and  Violation Comment. These columns were respectively renamed License_Number, Inspection_ID, Violation_ID, Violation_Comment for ease of use in MySQL.



Finally, we exported the result using OpenRefine’s SQL Exporter tool, which produces a file containing SQL statements intended to create and populate a table with the data.
These operations constituted a distinct workflow from the main data cleaning steps, and are represented by operation_history_violations.json and operation_history_violations.pdf. 

Data cleaning with other tools
We did not conduct cleaning with tools other than OpenRefine. During several cleaning steps, OpenRefine became unresponsive, which led us to conclude it may not have been an appropriate choice due to the size of the dataset. We would investigate other options if we were to take on a similar task in the future. The problems we encountered were most evident while processing the violations for use as a separate SQL table--after the modifications described in the previous section, the violations dataset comprised approximately three times as many rows as the original dataset, as there was one row for each violation. 
We were able to mitigate this by editing the Info.plist file to update the variable controlling the maximum heap space available to the Java VM. Ultimately, we determined that 4 GB was sufficient for a dataset of this size (“Xmx4096M” in the figure below).
