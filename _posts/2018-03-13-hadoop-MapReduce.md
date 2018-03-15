---
layout: post
title: Using Hadoop and MapReduce to analyze Seattle Public Library logs 
subtitle: Part 1 of Big Data series
image: /img/hadoopMapReduce/HadoopLogoMod.png
gh-repo: eskilan/hadoopWithPython
gh-badge: [star, fork, follow]
---

## Intro to Big Data
Welcome to the first post of the Big Data series. In this series of posts we will be going through what is known as Big Data technologies. 

What does the term "big data" mean? The term means data with high Volume, high Variety, and high Velocity; these are known as the three "V's" of big data. Large volumes of data refer to data sets so large that they won't fit in a regular hardrive and/or cannot be processed in a single PC. High variety data means that the data is composed of many different types of items. Some may be unstructured text, some may be blobs, some may be tables, etc. High velocity referes to data that is acquired more quickly than it can be processed or understood. For example, if an app is storing user behavior data for millions of people every minute, there is no simple way to store and process it, lets say, in a relational database with a fixed schema. Big data requires a particular approach when it comes to storage and processing, and in this post, we will be talking about Hadoop and the MapReduce algorithm.

## Hadoop
Hadoop (http://hadoop.apache.org/) is an open-source tool created to tackle the problem of storing and manipulating large data sets using a cluster of inexpensive distributed computers made of commodity hardware. The use of commodity hardware lowers the cost of data ownership and allows organizations to store much more data than used to be possible. Hadoop is composed of two separate but complimentary parts: HDFS (The Hadoop file system), and YARN (Yet Another Resource Negotiator). When we store data in HDFS, the data gets split into blocks which are copied and sent to different computers for storage. If one computer's hard drive fails, there are copies around stored in the other hard drives in the cluster. This redundancy makes data stored in HDFS very fault-tolerant. Meanwhile, YARN manages the resources of the cluster and schedules the jobs sent to it.

## What is MapReduce
MapReduce is the name of a computing paradigm where the computation is done in a distributed manner. Imagine you have the task of counting how many coins are stored in a large vault at a bank. Moreover, you need to count how many coins are worth 5 cents, how many are worth 10 cents and how many are worth 20 cents. Instead of trying to do the task all by yourself, you hire a bunch of people, maybe 100 workers, and have them take the coins and split them among themselves. Then you send each worker an instruction: "Count how many coins of each type, and relay the result back to me!" When you get the 100 results back, you add everything up in a spreadsheet and get your net results quickly.

![alt text](/img/hadoopMapReduce/coins.jpeg "Coins in a vault"){:height="30%" width="30%"}

In Hadoop the code is sent to where the data resides (each worker), the "map" part corresponds to each worker counting the coins of each type. The "reduce" is the operation where you take all intermediate results and join them into an overall result. This paradigm allows for the storage and analysis of really big data sets.
![alt text](/img/hadoopMapReduce/Mapreduce_Ville_Tuulos.png "By Ville Tuulos (Private Kommunikation) [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC-BY-SA-3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons" )

In scientific computing it is common practice to send data to be analyzed concurrently in multiple cores or multiple computers. When dealing with very large data sets with Hadoop and MapReduce, the data stays in place, and it is the code that travels to where the data "lives."

## Installing Hadoop on Ubuntu 16.04 LTS
If you wish to follow along with a Ubuntu machine, I'd recommend following the instructions provided at the following link https://data-flair.training/blogs/hadoop-2-with-yarn/
Installing Hadoop in pseudo-distributed mode will allow you to play with the system as if you were using a cluster. The process involves installing java, setting up an SSH connection to your own computer (localhost), and setting up some environment variables, among other things.

## Using HDFS
Once Hadoop is installed and running, we can use the hdfs command to access the filesystem. In many ways, HDFS is like the linux filesystem. To see what's inside a directory we type:
```bash
$ hdfs dfs -ls /myfolder
```
To display a file we can use the -cat option:
```bash
$ hdfs dfs -cat fileWithFullPath
```
To create a directory we can use he -mkdir option
```bash
$ hdfs dfs -mkdir pathForDirectoryCreation
```
To copy a file from the local filesystem into HDFS we use -put
```bash
$ hdfs dfs -put localFileNameWithFullPath hadoopFileNameWithFullPath
```
To take the results of a mapReduce job and place them in the local filesystem we use -getmerge
```bash
$ hdfs dfs -getmerge pathToDirectoryWithResults resultsFileNameLocal
```
These are also other commands such as -rm, -rm -r,-mv, etc.

## The data set
In this post I'll be using Hadoop MapReduce to find the most popular items checked out from the Seattle Public Library in 2017. I selected this data set for three reasons: 
1. The log files are large (5 million lines) 
2. The log files are not so large that I couldn't perform the analysis locally on my laptop (363.5 MB) 
3. I thought it'd be a cool project

## Taking a look at the data
The following is code written to a bash terminal in linux:

We examine the data:
```bash
$ head -10 Checkouts_By_Title_Data_Lens_2017.csv 

BibNumber,ItemBarcode,ItemType,Collection,CallNumber,CheckoutDateTime
2543647,0010063298235,accd,nacd,CD 782.42166 C6606So,01/02/2017 08:13:00 AM
3172300,0010087522552,acbk,namys,MYSTERY COTTERI 2016,01/02/2017 08:13:00 AM
2393405,0010054483200,acbk,camys,MYSTERY MAY2006,01/02/2017 08:24:00 AM
3199718,0010088153514,acdvd,nadvdnf,DVD 781.66092 M3347G 2013,01/02/2017 08:33:00 AM
3211526,0010089643810,accd,nacd,CD 782.42166 Sh75o,01/02/2017 08:33:00 AM
2743540,0010074812131,acbk,nanf,158.20874 M3167L 2011,01/02/2017 08:33:00 AM
3100439,0010085432515,accd,cacdnf,CD 617.48092 M353M 2015,01/02/2017 08:48:00 AM
2679201,0010072536583,accd,cacdnf,CD 616.994 M8968E 2010,01/02/2017 08:48:00 AM
3167465,0010088368476,accd,cacdnf,CD 327.1273 H3246P 2016,01/02/2017 08:48:00 AM
```

Before running anything on the real data, we create a small sample of 50 lines so that we can check our programs first:
```bash
$ head -50 Checkouts_By_Title_Data_Lens_2017.csv > testCSV.csv
```

## Mapping and Reducing in python

### mapper.py
This code processes a text file line by line. For each line, it selects the checked out item's bibNumber and outputs a list of form [bibNuber,1]. The "bibNumber" is the key and the "1" is the value.
For safety, it also performs checks on the line (how long it is and if the bibNumber is numerical).
```python
#!/usr/bin/python
# This code reads each line in the seattle library checkout log. The key is the bibnumber, and for each count outputs a 1
import sys
import csv

def mapper():

    reader = csv.reader(sys.stdin, delimiter=',')
    writer = csv.writer(sys.stdout, delimiter=',', quotechar='"', quoting=csv.QUOTE_ALL)
    for line in reader:

        # checking size
        if len(line)!=6:
            continue
        # check bibnumber
        if str.isdigit(line[0]) == False:
            continue
        
        keyValPair = []
        keyValPair.append(line[0])
        keyValPair.append(1)

        # Now print out the data that will be passed to the reducer
        writer.writerow(keyValPair)

def main():
    mapper()
    sys.stdin = sys.__stdin__
    
if __name__ == "__main__":
    main()
```

Running mapper.py on our small data sample is a simple as writing:
```bash
$ cat testCSV.csv | python2 mapper.py
```
In this command, the vertical line | is a pipe and sends the output from the first command (cat) to the following command (python2).

The output is:
```bash
"2543647","1"
"3172300","1"
"2393405","1"
"3199718","1"
"3211526","1"
"2743540","1"
"3100439","1"
"2679201","1"
"3167465","1"
"3172511","1"
"3216974","1"
"3216678","1"
"3221781","1"
"3132683","1"
"3183319","1"
"3199612","1"
"2384552","1"
"3146168","1"
"2699432","1"
"3043811","1"
"3198377","1"
"2686927","1"
"2969030","1"
"2450744","1"
"3148776","1"
"3015546","1"
"3172504","1"
"2614204","1"
"2918565","1"
"2839103","1"
"3148636","1"
"3082276","1"
"3111794","1"
"3056832","1"
"2758503","1"
"3091048","1"
"2303301","1"
"2959038","1"
"3165661","1"
"2839268","1"
"3182947","1"
"2883332","1"
"2969839","1"
"3089601","1"
"3125681","1"
"1742223","1"
"1698155","1"
"170113","1"
"3173410","1"

```

### reducer.py
After all computing nodes perform the mapping step, the outputs goes through a shuffle and sort phase and gets merged into a single file to be processed by the reducer. Because the data is now sorted by the key value, in our case by the bibNumber, we can read the file line by line and sum how many times each key is repeated, e.g., if we read four lines: [4341,1],[4341,1],[4341,1],[5931,1] we can tell there are three items of key 4341 and one of key 5931, simply by counting how many times the "current key" is repeated, and taking note of when it changes.

```python
#!/usr/bin/python
import sys
import csv

reader = csv.reader(sys.stdin, delimiter=',')
writer = csv.writer(sys.stdout, delimiter=',', quotechar='"', quoting=csv.QUOTE_ALL)

def outputCount(oldKey,countTotal):
    output = []
    output.append(oldKey)
    output.append(countTotal)
    writer.writerow(output)

countTotal = 0
oldKey = None

for data in reader:
    if len(data) != 2:
        # Something has gone wrong. Skip this line.
        continue

    thisKey = data[0]
    if oldKey and oldKey != thisKey: # key changed, output total
        outputCount(oldKey,countTotal)
        oldKey = thisKey
        countTotal = 0 # reset count

    oldKey = thisKey
    countTotal += 1
# last line
if oldKey != None:
    outputCount(oldKey,countTotal)
```
## Running mapReduce without Hadoop
When our data is small, we can test our mapper and reducer code without submitting a job to the cluster. This imitates the Hadoop mapReduce approach and allows you to debug problems before spending time and money on cluster processing.

First, we try our small sample of 50 lines:
```bash
$ cat testCSV.csv | python2 mapper.py | sort | python2 reducer.py
```
The output looks something like this:
```bash
"1698155","1"
"170113","1"
"1742223","1"
"2303301","1"
"2384552","1"
"2393405","1"
"2450744","1"
"2543647","1"
.
.
.
```
This doesn't give us much information because the outputs are all 1 still. The data is too small.

Since we know our full data isn't too large we can try running the whole data set and saving it locally into a csv file:
```bash
cat Checkouts_By_Title_Data_Lens_2017.csv | python2 mapper.py | sort | python2 reducer.py > expectedOutput.csv
```
We keep that and compare it later with the Hadoop output to verify that we get the same result.
## Running mapReduce with Hadoop Streaming
Hadoop is written in Java but it can run python scripts without issues. That functionality comes from Hadoop Streaming (https://hadoop.apache.org/docs/r2.7.0/hadoop-streaming/HadoopStreaming.html). However, the first thing we must to is to place the data in HDFS.
```bash
$ hdfs dfs -mkdir /seattleInput
$ hdfs dfs -ls /
18/03/07 20:00:31 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 5 items
drwxr-xr-x   - ilan supergroup          0 2018-03-06 20:47 /inputFolder
drwxr-xr-x   - ilan supergroup          0 2018-03-07 16:32 /outputFolder
drwxr-xr-x   - ilan supergroup          0 2018-03-07 19:59 /seattleInput
drwx------   - ilan supergroup          0 2018-03-07 16:22 /tmp
drwxr-xr-x   - ilan supergroup          0 2018-03-07 16:22 /user

```
Adding file:
```bash
$ hdfs dfs -put Checkouts_By_Title_Data_Lens_2017.csv /seattleInput
$ hdfs dfs -ls /seattleInput
Found 1 items
-rw-r--r--   1 ilan supergroup  363465870 2018-03-07 20:03 /seattleInput/Checkouts_By_Title_Data_Lens_2017.csv
```

Now that our input data is in our "cluster" we can run the streaming command. The streaming command I use is the following:
```bash
$HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar \
    -input /seattleInput/Checkouts_By_Title_Data_Lens_2017.csv \
    -output /seattleOutput \
    -mapper "python2 mapper.py" \
    -reducer "python2 reducer.py" \
    -file mapper.py \
    -file reducer.py
```
The "-file" option is necessary because the python scripts have to be attached to the job. Other required files can also be passed in that manner. The "python2" added to the mapper and reducer are necessary in my computer because my default python version is 3.x but hadoop streaming still uses python 2.x. This command runs mapReduce in a cluster, or in my case in my pseudo-cluster. 

When the job is finished we check the output folder in HDFS:
```bash
$ hdfs dfs -ls /seattleOutput
18/03/07 20:05:41 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 ilan supergroup          0 2018-03-07 20:05 /seattleOutput/_SUCCESS
-rw-r--r--   1 ilan supergroup    4912630 2018-03-07 20:05 /seattleOutput/part-00000
```

Then I extract the result to my local filesystem:
```bash
$ hdfs dfs -getmerge /seattleOutput/ hadoopOutput1.csv
```
## Post-processing
The result of mapReduce is only part of the story. I still don't know which were the most popular items in the Seattle Public Library in 2017. The results have to be sorted and interpreted. In the following postprocess.py code, I load the results (key,value) pairs into a pandas dataframe. I also load a library item reference called "Library_Collection_Inventory.csv" which contains the BibNumber,Title, Author, ISBN, and other information.
###postprocess.py
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
import re

mrDF = pd.read_csv('hadoopOutput1.csv',header=None,  names = ['BibNum','nCheckouts']).set_index('BibNum') # 321772 rows
libRefDF = pd.read_csv('Library_Collection_Inventory.csv').set_index('BibNum') # ~2.6 million rows
libRefDF = libRefDF[['Title','Author','ISBN','PublicationYear']].drop_duplicates()
# cross reference column1 from mrDF with BibNum of libRefDF
n = 100
topN = mrDF.nlargest(n, 'nCheckouts')

results = topN.join(libRefDF,how='inner').sort_values(by=['nCheckouts'], ascending=False)

splitTitle = results.Title.str.split('/')
pattern = re.compile('\[.*\]')
results['Title_1'] = splitTitle.str[0].str.replace(pattern,'')
results['Title_2'] = splitTitle.str[1]

def selectEarliestYear(myList):
    numericList = []
    for item in myList:
        if item.isnumeric():
            numericList.append(int(item))
    if len(numericList) > 0:
        return min(numericList)
    else:
        return np.nan

# let's clean up the publication year from libRefDF and results
def cleanYear(df):
    mySeries = df['PublicationYear'].str.split(',|\.|\[|\]|-|c|©')
    mySeries = mySeries.apply(selectEarliestYear)
    return mySeries

## Now we can compare the distribution of publication years between the overall library and our most checked out items
resultsYear = cleanYear(results)
overallYear = cleanYear(libRefDF.dropna().head(1000))

f, (ax1, ax2) = plt.subplots(2)

g1 = sns.distplot(resultsYear.dropna(),norm_hist=True, ax=ax1)
g2 = sns.distplot(overallYear.dropna(),norm_hist=True, ax=ax2)
g1.set(xlim=(1970,2017))
g2.set(xlim=(1970,2017))
g1.set_title('Top 100 most popular items')
g2.set_title('Sample of items in collection')

print(results[['nCheckouts','Title_1','Title_2']].head(20))
```

I output the top 20 most popular items checked out from the library:
```bash
         nCheckouts                                            Title_1  \
BibNum                                                                   
3030520        5461                  SPL HotSpot : connecting Seattle    
2919580        3631                        In Building Device Checkout   
3237829        3086                                           Arrival    
3245906        2494                                         Moonlight    
3177840        2427                The Underground Railroad : a novel    
3243783        2368                             Manchester by the Sea    
3223559        2361                                            Sully     
3259561        2360                                    Hidden figures    
3183534        2312  Hillbilly elegy : a memoir of a family and cul...   
3213429        2293                                     Finding Dory     
3217759        2189                          The secret life of pets     
3227098        2171                             The girl on the train    
3245681        2152                                             Moana    
3262198        2120                                        La La Land    
3219373        2105                                     Jason Bourne     
3230634        2102                                    The accountant    
3236824        2049                                     Hacksaw Ridge    
3243782        2003                                              Lion    
3255704        1994                                        Passengers    
3246153        1944           Fantastic beasts and where to find them    

                                                   Title_2  
BibNum                                                      
3030520                 [distributed by Verizon Wireless].  
2919580                                                NaN  
3237829   Paramount Pictures ; Filmnation Entertainment...  
3245906   A24 ; Plan B ; Pastel ; produced by Adele Rom...  
3177840                                  Colson Whitehead.  
3243783   Lionsgate ; Amazon Studios ; a Pearl Street F...  
3223559   Warner Bros. Pictures ; in association with V...  
3259561   20th Century Fox ; Fox 2000 Pictures ; a Cher...  
3183534                                        J.D. Vance.  
3213429   Walt Disney Pictures ; Pixar Animation Studio...  
3217759   Universal ; Illumination Entertainment ; prod...  
3227098   Dreamworks Pictures and Reliance Entertainmen...  
3245681   Disney ; Walt Disney Animation Studios ; prod...  
3262198   Summit Entertainment ; in association with Bl...  
3219373   Universal Pictures ; in association with Perf...  
3230634   Warner Bros. Pictures ; Ratpac ; an Electric ...  
3236824   Summit Entertainment ; Cross Creek Pictures ;...  
3243782   The Weinstein Company ; Screen Australia ; Se...  
3255704   Sony ; Columbia Pictures ; Start Motion Pictu...  
3246153   Warner Bros. Pictures ; a Heyday Films produc...
```

The results are quite interesting. The most checked out items are devices! Something called an SPL HotSpot from Verizon Wireless, as well as an "In Building Device Checkout." A quick google search clarifies this a little more. An SPL hotspot is a device that will give the user free internet connectivity and can be checked out for 21 days. Sounds like a great deal. I imagine the second item must be laptops or tablets checked out for use within the library. The next most popular items are films "Arrival" and award-winning "Moonlight." As a matter of fact, almost all most popular items are recent movies. A notable exception is "The Undergroung Railroad: a novel."

Since it seems the majority of popular items in 2017 have a recent publication date, I decided to test this by qualitatively comparing the histogram of the 100 most popular items binned by publication date, to a histogram of a sample of the entire library catalog. The result is below:

![alt text](/img/hadoopMapReduce/distribution.png "Comparing the most popular items to a sample from the library catalog.")

It is clear that the library catalog has a much larger spread in publication dates than the top 100 most popular items.

After doing this analysis I wanted to find out who were the most popular "Authors" in the library in 2017, as opposed to the most popular "Items." The problem is slightly more complex because first we would have to use bibNumbers as keys, perform mapReduce, then somehow cross-reference bibNumbers with Authors and perform mapReduce again with Authors as keys. It seems to me that doing so is rather cumbersome and limiting for more advanced analytics and that the complexity in handling multiple mapReduce jobs in series is what led to other projects like Apache Spark, and Hive.

Thank you for reading. Let me know if you have any comments!

Note: The Hadoop logo is an altered version of that by Apache Software Foundation - http://svn.apache.org/repos/asf/hadoop/logos/asf_hadoop/, Apache License 2.0, https://commons.wikimedia.org/w/index.php?curid=63919822