# Table of Contents
1. [Introduction](README.md#introduction)
2. [Details](README.md#details)
3. [Implementation details](README.md#implementation-details)
4. [Input files](README.md#input-files)
5. [Output file](README.md#output-file)
6. [Example](README.md#example)
7. [Approach](README.md#approach)
8. [Run Instructions](README.md#Run-Instructions)
9. [Contact](README.md#contact)

# Introduction

Many investors, researchers, journalists and others use the Securities and Exchange Commission's Electronic Data Gathering, Analysis and Retrieval (EDGAR) system to retrieve financial documents, whether they are doing a deep dive into a particular company's financials or learning new information that a company has revealed through their filings. 

The SEC maintains EDGAR weblogs showing which IP addresses have accessed which documents for what company, and at what day and time this occurred.

The data engineering part is to build a pipeline to ingest that stream of data and calculate how long a particular user spends on EDGAR during a visit and how many documents that user requests during the session. 


# Details

For the purposes of this problem, an IP address uniquely identifies a single user. A user is defined to have visited the EDGAR system if during the visit, the IP address requested one or more documents. 

Also, the amount of time that elapses between document requests is used to determine when a visit, also referred to as a session, begins and ends. 

A single user session is defined to have started when the IP address first requests a document from the EDGAR system and continues as long as the same user continues to make requests. The session is over after a certain period of time has elapsed -- which is provided externally -- and the user makes no requests for documents. 

In other words, this period of inactivity helps to determine when the session is over and the user is assumed to have left the system. 

The duration of any particular session is defined to be the time between the IP address' first request and the last one in the same session prior to the period of inactivity. If the user returns later to access another document requests, that subsequent request would be considered the start of a new session.

# Implementation details

The program expects two input files:

* `log.csv`: EDGAR weblog data
* `inactivity_period.txt`: Holds a single value denoting the period of inactivity that should be used to identify when a user session is over

As we process the EDGAR weblogs line by line, the moment we detect a user session has ended, the program writes a line to an output file, `sessionization.txt`, listing the IP address, duration of the session and number of documents accessed.

The value found in `inactivity_period.txt` is used to determine when a session has ended and when a new session has possibly started. However, once we reach the end of the `log.csv`, that last timestamp should signal the end of all current sessions regardless of whether the period of inactivity has been met.

## Input files

### `log.csv`

The SEC provides weblogs stretching back years and is [regularly updated, although with a six month delay](https://www.sec.gov/dera/data/edgar-log-file-data-set.html). 

It is assumed that the data is being streamed into our program in the same order that it appears in the file with the first line (after the header) being the first request and the last line being the latest. It is assumed the data is listed in chronological order.

While you're welcome to run your program using a subset of the data files found at the SEC's website, you should not assume that we'll be testing your program on any of those data files.

Below are the data fields we'll want to pay attention to from the SEC weblogs:

* `ip`: identifies the IP address of the device requesting the data. While the SEC anonymizes the last three digits, it uses a consistent formula that allows you to assume that any two `ip` fields with the duplicate values are referring to the same IP address
* `date`: date of the request (yyyy-mm-dd) 
* `time`:  time of the request (hh:mm:ss)
* `cik`: SEC Central Index Key
* `accession`: SEC document accession number
* `extention`: Value that helps determine the document being requested

There are other fields that can be found in the weblogs. Our program ignores those other fields.

It is assumed that the combination of `cik`, `accession` and `extention` fields uniquely identifies a single web page document request.

### `inactivity_period.txt`
This file will hold a single integer value denoting the period of inactivity (in seconds) that your program should use to identify a user session. The value will range from 1 to 86,400 (i.e., one second to 24 hours)

## Output file

The program identifies the start and end of a session, it gathers the following fields and writes them out to a line in the output file, `sessionization.txt`:

* IP address of the user exactly as found in `log.csv`
* date and time of the first webpage request in the session (yyyy-mm-dd hh:mm:ss)
* date and time of the last webpage request in the session (yyyy-mm-dd hh:mm:ss)
* duration of the session in seconds
* count of webpage requests during the session


# Example

Suppose your input files contained only the following few lines. Note that the fields we are interested in are in **bold** below but will not be like that in the input file. There's also an extra newline between records below, but the input file won't have that.

**`inactivity_period.txt`**
> **2**

**`log.csv`**

>**ip,date,time**,zone,**cik,accession,extention**,code,size,idx,norefer,noagent,find,crawler,browser

>**101.81.133.jja,2017-06-30,00:00:00**,0.0,**1608552.0,0001047469-17-004337,-index.htm**,200.0,80251.0,1.0,0.0,0.0,9.0,0.0,

>**107.23.85.jfd,2017-06-30,00:00:00**,0.0,**1027281.0,0000898430-02-001167,-index.htm**,200.0,2825.0,1.0,0.0,0.0,10.0,0.0,

>**107.23.85.jfd,2017-06-30,00:00:00**,0.0,**1136894.0,0000905148-07-003827,-index.htm**,200.0,3021.0,1.0,0.0,0.0,10.0,0.0,

>**107.23.85.jfd,2017-06-30,00:00:01**,0.0,**841535.0,0000841535-98-000002,-index.html**,200.0,2699.0,1.0,0.0,0.0,10.0,0.0,

>**108.91.91.hbc,2017-06-30,00:00:01**,0.0,**1295391.0,0001209784-17-000052,.txt**,200.0,19884.0,0.0,0.0,0.0,10.0,0.0,

>**106.120.173.jie,2017-06-30,00:00:02**,0.0,**1470683.0,0001144204-14-046448,v385454_20fa.htm**,301.0,663.0,0.0,0.0,0.0,10.0,0.0,

>**107.178.195.aag,2017-06-30,00:00:02**,0.0,**1068124.0,0000350001-15-000854,-xbrl.zip**,404.0,784.0,0.0,0.0,0.0,10.0,1.0,

>**107.23.85.jfd,2017-06-30,00:00:03**,0.0,**842814.0,0000842814-98-000001,-index.html**,200.0,2690.0,1.0,0.0,0.0,10.0,0.0,

>**107.178.195.aag,2017-06-30,00:00:04**,0.0,**1068124.0,0000350001-15-000731,-xbrl.zip**,404.0,784.0,0.0,0.0,0.0,10.0,1.0,

>**108.91.91.hbc,2017-06-30,00:00:04**,0.0,**1618174.0,0001140361-17-026711,.txt**,301.0,674.0,0.0,0.0,0.0,10.0,0.0,

The single line on `inactivity_period.txt` tells us that once two seconds have elapsed since a user made a document request, we can assume that user's particular visit has ended. Any subsequent requests would be considered a new session.

The first day and time listed in the input file is 2017-06-30 and the time is 00:00:00. That means at that date and time, the following ip addresses initiated a visit to EDGAR:

* **101.81.133.jja** made a request for cik: **1608552.0**, accession: **0001047469-17-004337** and extention: **-index.htm**
* **107.23.85.jfd** made a request for cik: **1027281.0**, accession: **0000898430-02-001167** and extention: **-index.htm**
* **107.23.85.jfd** made a request for cik: **1136894.0**, accession: **0000905148-07-003827** and extention: **-index.htm**

So for the first second of data that your program has encountered, it knows one user has accessed one document and a second user has requested two:

![First second illustration](images/first_second.png)


When your program reads in the input file's fourth line, it should detect that the day and time has advanced by one second. So now, this is what we know:

![Second second illustration](images/second_second.png)


Then when it reaches the sixth and seventh line:

![Third second illustration](images/third_second.png)

When it first reads the eighth line, it should detect that the time is now `2017-06-30 00:00:03`. For one user, `101.8.33.jja`, its session has ended because two seconds of inactivity have passed for that user. Because there was only one request, only one web page document was accessed. 

![End of third second illustration](images/end_of_third.png)

At that point, the output file `sessionization.txt` should contain the following line:

    101.81.133.jja,2017-06-30 00:00:00,2017-06-30 00:00:00,1,1

After processing the eighth line of the input file and as we examine the timestamp in the ninth line of the input file, we detect that the time has progressed to `2017-06-30 00:00:04`. For a second user, `108.91.91.hbc`, we now see that two seconds of inactivity has elapsed and we can identify a second session:

![Fourth second illustration](images/fourth_second.png)

The output file `sessionization.txt` should now consist of the following data:

    101.81.133.jja,2017-06-30 00:00:00,2017-06-30 00:00:00,1,1
    108.91.91.hbc,2017-06-30 00:00:01,2017-06-30 00:00:01,1,1

Finally, after your program processes the ninth and 10th line, it should detect that the end of file has been reached and there are no more requests for any users. At this point, it should identify all sessions regardless of the period of inactivity:

![End of file illustration](images/end_of_file.png)

At that point, it should write the results to the output file, and the entire content of `sessionization.txt` should be:

    101.81.133.jja,2017-06-30 00:00:00,2017-06-30 00:00:00,1,1
    108.91.91.hbc,2017-06-30 00:00:01,2017-06-30 00:00:01,1,1
    107.23.85.jfd,2017-06-30 00:00:00,2017-06-30 00:00:03,4,4
    106.120.173.jie,2017-06-30 00:00:02,2017-06-30 00:00:02,1,1
    107.178.195.aag,2017-06-30 00:00:02,2017-06-30 00:00:04,3,2
    108.91.91.hbc,2017-06-30 00:00:04,2017-06-30 00:00:04,1,1

Notice from the above output that the first two lines were the ones we had already written. 

The third line details the session for `107.23.85.jfd` next because its first document request came at `2017-06-30 00:00:00`, which is earlier than any of the other remaining sessions. 

The fourth line belongs to IP address, `106.120.173.jie` because that user's first document request came at `2017-06-30 00:00:02`. The first document request from `107.178.195.aag` also comes at the same time but it is listed after `106.120.173.jie` in the input file so that is why it is listed on the fifth line.

The second session detected for `108.91.91.hbc` concludes the `sessionization.txt` file.

# Approach

The program first identifies the column headers and it's locations. It then read the input file line by line.
We have a list for storing all ip addresses and a log dictonary containing the ip as the key and corresponding start time,end time, duration and web page count of it.

If an ip is read for the first time,
We simply update the dictionary with it's values, having start and end time as the same, duration count as 1 second and webpage count as 1
Following that we append that ip into out list of ips.
For each of the line we read, we go through the list of ips and check if the inactivity period exceeds the session duration.
If yes, then we call writer function, remove ip from the list and also from the log dictionary
If not, we simply update the values in the log dictionary for the corresponding key(ip)

At the end of the file, we end all sessions and append the remaining logs in the log dictionary into the output file

# Run Instructions

Running the run.sh file in the project root directory will run the script and produce the results in the specified format in specified folder

# Contact

sbm4394@gmail.com
