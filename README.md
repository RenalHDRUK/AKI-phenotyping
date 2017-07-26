# Aberdeen AKI phenotyping algorithm guide


|Index	|	|
|------------- | -------------|
What does the Aberdeen AKI phenotyping algorithm code do?	|1|
|What prerequisite data are needed?	|1|
|What are the caveats?	|1|
|How should my data be organised?	|2|
|Which of the generated variables should I use?	|3|
|Are there any lines of code I should edit prior to use?	|4|
|How do I know if I have interpreted the output correctly?	|4|
|Guide to Mock data outputs	|5|
|Mock data outputs: dataset details	|6|
|Mock data outputs: AKI events and phenotypes	|7|

## What does the Aberdeen AKI phenotyping algorithm code do?
The “do” file code flags blood tests that are consistent with AKI based on a comparison with previous tests. It also arranges the flagged tests so that start points of discrete episodes can be identified and each episode can be phenotyped with respect to baseline, severity, recovery and recurrence. 

## What prerequisite data are needed?
Your dataset should contain as a minimum the following variables named as below:


|Name | Description | Format |  
|------------- | -------------|------------|  
studyid |		unique id number for each individual|(integer)|
dos 	|		date of each sample| (numerical)|
stcreat |		idms aligned serum creatinine (micromol/L)|(numerical)|
mdrd 	|		4 variable MDRD eGFR (ml/min/1.73m2)|(numerical)|
sex 	|		male=1 female=0| (binary)|
age 	|		age in years at time of sample|(integer)|
Optional:|	|
location_code| 	inpatient/outpatient/community sample |(categorical)|

## What are the caveats?

Ultimately the code will only provide reasonable outputs if the variables are recorded, extracted and prepared correctly. Outputs should be interpreted in light of the data source’s limitations, and therefore an absence of event flags should not be considered confirmation of condition absence.

For a given patient, all serum creatinine values taken from all locations and all time points should be included. If a patient is consistently having blood tests drawn that are not captured from one location this may result in missing either baseline data, or periods of illness – both of which may lead to underestimation of AKI. 
When presenting outputs, the extent of data capture should be reported; the completeness of inpatient, outpatient and community samples; and whether samples taken as an emergency (e.g. with a temporary id) have been reconciled to the correct individual by the biochemistry department.

There are inherent ascertainment biases in data capture in any routine dataset. Each blood sample represents an informative event done for a clinical reason (acute illness, chronic disease monitoring). Reasons and sampling patterns may vary over time and with respect to the case-mix of the population under investigation. Differences in AKI rates therefore do not necessarily mean differences in either health of care quality between time periods or locations.

Any changes in creatinine assay should be explored with the biochemistry department who provide the results. Changes in assay could lead to a step change in function across the population (even if adjustment factors are used) – which should be checked and reported.
Note that changes in body physiology (e.g. pregnancy) and composition (increase in muscle mass) can also lead to the appearance of AKI in the absence of a change in excretory function. This should be borne in mind for samples that come from maternity units.

## How should data be organised?

The dataset should be organised with all serum creatinine values and test dates in “long format” on sequential rows by “study id” and “sample date”. The recommended order of processing is provided below.

All initial variables should be “numerical” rather than “string”. Not that some values may need to be corrected. E.g. if MDRD is only reported as “>60” it would be better to apply the 4vMDRD equation to generate an actual eGFR value.

Null/missing creatinine values and values believed to be incorrect should be excluded as all will lead to false positive AKI flags. Note that Stata interprets a missing value as a very large number. For example this may include samples labelled as “non-compliant with biochemistry acceptance policy”, or samples where serum potassium and sodium has been measured, but not creatinine.

Ideally blood tests from people already receiving long term RRT by the date of the sample should be excluded because these tests are neither a true reflection of kidney function, nor represent AKI when fluctuations occur. In a population of 500,000 one would expect ~300-500 people to have blood tests excluded.

Only one creatinine for each sample date should be provided. This is because the timing of sampling is not necessarily reliable, particularly for community samples. We would suggest retain the highest creatinine on a given day if more than one sample is taken. This maximises the ascertainment of impairment, and minimises the inclusion of “dilutional” samples.

## Which of the generated variables should I use and how?

The algorithm “do” file contains lines of explanation embedded within it. We also recommend first working with the mock data file provided before analysing test data. This is a fictitious dataset manually created to ensure that the algorithm code file is used and interpreted correctly.

A series of new indicator variables are created by the algorithm to identify and arrange tests with specific properties. Key variables are described below.

|Name | Meaning |  
|------------- | -------------| 
newAKI 	|	Indicates that this test satisfies one of the AKI criteria|
AKImonth |		Indicates that this test satisfies the 8-90 day AKI criterion|
AKIyear |		Indicates that this test satisfies the 91-365 day AKI criterion|
AKIweek |		Indicates that this test satisfies the 7 day AKI criterion|
AKIday 	|	indicates that this test satisfies the 2 day AKI criterion|
firstAKI |	The first AKI test that meets any AKI criteria for a given individual: “count if firstAKI==1”|
AKIcounter|	The AKI episode count (number of complete AKI episodes up to that point, including the one in progress): “tab AKIcounter if firstAKI==1”|
AKIspellcounter |	The AKI duration (number of AKI blood tests up to that point within the episode in progress). Can be used to count the number of AKI episodes in a time period: “count if AKIspellcounter==1 & year(dos)==2003”|
AKIrefGFR|	Reference eGFR for the first AKI episode in the index year for those with at least one AKI episode (combine with “if firstAKI==1 to isolate one result per individual). “sum AKIrefGFR if firstAKI==1”|
refGFRgroup	|	Categorical version of the reference eGFR|
AKIN|	AKI episode severity stage for the first AKI episode in the index year for those with at least one AKI episode: “tab AKINdx if firstAKI==1”|
AKINdx|	Initial AKI episode severity stage for the first AKI episode in the index year for those with at least one AKI episode when measured at the time of the first test meeting AKI criteria (i.e. using the first not the highest creatinine of the episode): “tab AKINdx if firstAKI==1”|
prevAKIcount|	Number of previous AKI episodes at the time of an AKI episode|
priorAKI1yr|	Presence of a prior AKI episode within one year (use “tab priorAKI 1yr if firstAKI==1” to identify those whose AKI episode is a recurrence within the past year.|
recovery|	Metric of renal recovery to within 1.2 x baseline creatinine (1=recovery 0=non-recovery 2=untested): “tab recovery if firstAKI==1”|

## Are there any lines of code I should edit prior to use?

The code is written to follow individuals from 1st January 2000, through an index year of 2003, describing the first AKI episode of 2003 for those who develop AKI”. To change the dates of follow up you will need to update lines 71 to 75. You may also wish to create a different variable to substitute “yr03”, which is an indicator variable to filter out any blood tests not from 2003 for certain calculations.

The code loops through all available blood tests for each individual and therefore must loop for the maximum number of tests for any individual. This value is the max value generated by line 110 (“sum protagonist, detail”). This value should be pasted into line 116 (currently showing 790).

## How do I know if I have interpreted the output correctly?

We strongly recommend that you first use the mock dataset before moving to your own data. The outputs are provided below.
 
# Guide to Mock data outputs

Missing creatinine values are present on lines 304 and 379, which should be removed in an initial data cleansing step

After removing missing creatinine values (stcreat), number of records in the index year (2012) should equal 864

Records should be removed which predate the initiation date of renal replacement therapy (RRTdate)

After removing post RRT values “dos>RRTdate & !missing(RRTdate)”, number of records in the index year (2012) should equal 793

For each day, the highest creatinine value is selected, and other values removed, after which records in the index year (2012) should equal 746

Please carefully confirm your interpretation of the outputs matches that described in the tables below
 
## Mock data outputs: dataset details

|Output	|Patient total	|% of total	|
|------------- | -------------|	
Adult serum creatinine samples analysed in index year	|864	|100|
Number of non-RRT samples in index year|	793|	91.78|
Number of samples in index year after the duplicate and non-RRT samples removed|	746|	81.60|
Number of (non rrt) individuals with a creatinine test in index year 	38||	100|
		
|Sex	|Patient total	|% of total	|
|------------- | -------------| -------------|
females	|19	|50	|
males	|19	|50	|
		
|Age	|Patient total	|% of total	|
|------------- | -------------| -------------|		
Age ≥70 years	|16	|42.1	|
Age 40-69 years	|19	|50	|
Age 15-39 years	|3	|7.9	|
		
|First eGFR value recorded (index year, ml/min/1.73m2)	|Patient total	|% of total	|
|------------- | -------------| -------------|
≥60	|29	|76.3	|
45-59	|6	|15.8	|
30-44	|3	|7.9	|
<30	|0	|0	|

 
## Mock data outputs: AKI events and phenotypes

|Output	|Total people	|
|------------- | -------------| 
People with AKI detected in index year	|22	|
Sex	|	|
females	|12	|
males	|10	|
Age|	|
Age ≥70 years	|10|
Age 40-69 years	|10|
Age 15-39 years	|2|
AKI severity|	|
AKI severity stage 1 (First AKI, index year)|	18|
AKI severity stage 2 (First AKI, index year)|	2|
AKI severity stage 3 (First AKI, index year)|	2|
Baseline eGFR (ml/min/1.73m2, first AKI of index year)|	|
≥60	|18|
45-59	|4|
30-44	|0|
<30	|0|
Prior AKI episodes detected in last 3 |	|
No prior episodes |	22|
1 prior episode|	0|
2 or more prior episodes|	0|
Renal recovery	|	|
To within 20% of baseline function|	4|
Non-recovery to baseline|	|
No repeat tests available to determine recovery|	6|


