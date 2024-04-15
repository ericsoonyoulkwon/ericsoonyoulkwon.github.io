# String Reconciliation in Excel

#### Category: MS Excel & VBA

---

### Comparing string data for differences

Cleansing data is essential. Although it's still not impossible to use data with compromised integrity to draw meaningful analytical conclusions, it is always preferable to maintain the cleanness of data. I found it helps to improve the usefulness of data for further analysis as well as to reduce transactional errors.

## Why am I doing this task of string data reconciliation?

I have a dataset with ownership information related to real estate properties that has been accumulated throughout many years of operation. This impacts the AR and collection process so the potential risk is that we may send a bill to the wrong customer if the ownership information is inaccurate. When I suggested the need for proactive cleaning of the ownership information at least once a while before such an error is brought to us when a customer's complaint is received, the response was the team already tried the mass cleanup but ended up giving it up a few years ago. Wait a minute, so there was an attempt and the project was halted? Not sure why but whenever I hear such challenges, they just trigger my excitement and solving the abandoned problem becomes my interest all of a sudden.

## Assessing the situation

After interviewing the team, below were the challenges that held back the attempt because the previous analysis resulted in 99% mismatches between the ownership information we have in our system and the dataset we are being provided from a confidential source that is reliably up-to-date and more accurate. That were just too many mismatches to be reviewed by our representatives.

- Large volume. Approximately 140,000 properties
- Heterogeneous data structure
  - Ownership information is saved in six different columns in our database
  - It is saved in one table in the data we are provided with for comparison. Even worse, co-owners separated by semi-colons are stored in one column.
- Heterogenous format of the naming convention used in two datasets compared
  - 'Lastname,Firstname' format in our data
  - 'Firstname Lastname' format in the reference data provided
  - Even worse, the rules have not always been followed and prone to human errors such as typos, extra commas, space, and sometimes different separators were used, etc.

I am pretty sure that the large volume of data was not even a challenge that led to the failure of an initial attempt as 140,000 rows is not an impossible scale in Excel (the maximum number of rows manageable in Excel is 220, slightly more than a million). Also, the number of columns to be compared is not a challenge either because it could simply be treated as a many-to-many comparison after splitting a column into multiple columns. The many-to-many comparison is achieved using nested IFs in the OR statement (this is IFs inside OR and not to be confused with the nested OR inside IF). The name convention could also have been resolved by replacing the comma with a space, splitting and reversing the order of the first and last names for comparison. It is always the inconsistent application of various formats in the column that makes the transformation for analysis harder.

## Actions I took

Among all the steps that I put in place to achieve the goal of obtaining a list of mismatches in ownership information between two datasets, I am going to focus on sharing the details of the steps that I took to address the inconsistent formats and prioritize the records based on the likeliness of the errors.

### Splitting a column into many columns

In our database, there are 6 columns where the ownership information is stored in. The COALESCE () function in SQL could have been used when I pulled the data if only one of the 6 is occupied but it was common that more than one column was occupied with the owner-like names. So, I decided to include all 6 columns in the analysis.

In the reference data that we were provided with, there was only one column for the ownership information. However, the data in the column was a concatenation of many, up to 6, owners separated by ‘; ‘ (a semi-colon and a space). The column was split by either using the text-to-columns wizard in Excel or using the formula as follows. I preferred the latter because the work is reproducible and can be templatized.

![name-split-formula](/images/name-split-formula.png)

### Overcoming the inconsistency in the naming convention

When there was a formatting inconsistency having sneaked into the names and it exaggerated the list of mismatches, I thought anything other than the true component of the name itself should not be accounted for in the comparison. For example, I didn’t want my name ‘Eric Kwon’ entered as ‘Eric K Won’ or ‘Eric-Kwon’ to be included in the list to be reviewed as a result of my analysis. So, I came up with the idea of counting the number of each alphabet occurred in the name. Assigning natural numbers 1 to 26 to A to Z and summing them up would result in a comparable number that the effect of such minor variations is eliminated. ‘Eric Kwon’ would result in 5+18+9+3+11+23+15+14 = 98 and the variations such as ‘Eric K Won’ or ‘Eric-Kwon’ would result in the same. But then I was concerned about the false-negative that the result incorrectly indicates the absence of mismatching conditions when the variations result in the same total by chance (the result for Bob and Do is 19). Although it is less likely the case, I just wanted to eliminate the possibility. So, I decided to assign prime numbers instead of natural numbers. If the product, the result of the multiplication is used, such coincidence will not happen as the prim numbers are the numbers that are divided by 1 and itself, therefore, the product of prime numbers does not coincide with another prime number. Instead of the product of multiplication, for the sake of saving computing effort, I decided to use summation of prime numbers assigned to each letter as summation would do enough job to eliminate such false-negative. Please note that I assigned prime numbers from Z to A, and 1 was assigned to Z even though 1 is not a prime number considering Z would not frequently occur in the names.

![text-rec](/images/text-rec.png)

The IF statement on the first line was used to skip the calculation and avoid unnecessary computation if not all 6 owners columns are filled. The logic in the formula is counting the occurrence of a certain letter by measuring the length of characters decreased when the letter was replaced by a blank. Then assigned prime numbers were multiplied. The same logic was applied to the owner names from two different datasets for each property. As mentioned above, the 36 IF statements inside OR were used for many-to-many comparisons. Not to make the explanation too long, each dataset contained up to 6 names per property so 6*6=36 was the number of the Cartesian product.

### Prioritizing works

After I applied the strategy of converting letters to prime numbers, the number of records that require our representatives’ review has come down from 140k to 65k. So, I wanted to narrow it down further. because it was still too many records for manual review although the result was reduced by more than half.

I noticed that there were some partial matches that could have been excluded from the final result of mismatches if the analysis opted out of the change of last name over time, missing/additional middle name in one data, missing/additional dot, the inclusion of operating-as name in case the entity is business or industrial, and other changes or variations that were made in one data but not updated in the other data.

So, I added an additional search that checks if the part of the name in one dataset is included in the name in the other dataset. The types of searches I added are as follows but I won’t discuss these here in detail.

  - word before the 1st comma
  - word after the 1st comma (if there is no, last word)
  - word before the 1st space
  - the word after the 1st space

## Result

When I excluded these partial matches, I was able to come up with only 3,800 records that are required for review. There could be very nominal false-positive sneaked into the result but won’t significantly impact the total hour spent for the review. Our team can now prioritize working on only 2.5% of the entire records to maintain the data accurately. One may suggest that the team perform the full review of the entire 65k of mismatches but I would say that would be less cost-effective. There are obviously false negatives not captured in the final list but they can be corrected on an item-by-item basis when the complaint is received. We’ve not done the mass cleanup for several years and made a significant improvement once the 3,000 records were corrected. It wouldn’t really be worth it to spend tons of staff hours reviewing all 65k records. I am not saying good enough is good enough. I just value my staff’s time and believe they can contribute in a better way with the saved time and energy. That was the whole point of and in line with narrowing down the list for review rather than performing the full review of the entire dataset.

## My 2 cents

We hope that the data is maintained consistently from the beginning of the input stage during the whole business process. However, when that is not always the real-life case, I would advise being creative and prioritizing tasks to select and focus on where there are more risk factors. Plus, never give up, as seen from the example I presented, discrepancies from 99% to 2.5% are a great improvement and shortlisting down to 3,000 records out of 140,000 made the review a doable task when delegated to the team for review to achieve reasonable assurance regarding data integrity. Well actually, by the time I am writing this, the review of the shortlisted has already been done.
