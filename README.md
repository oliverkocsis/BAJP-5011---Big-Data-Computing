# BAJP-5011---Big-Data-Computing
## AWS/S3/Unix
### Bucket: ceu2016kocsiso

### EMR cluster: ceu2016kocsiso

1. Private-Public key pair is created:  ceu2016kocsiso
2. Inbound SSH rule is added to ElasticMapReduce-master security group

### s3cmd
```
chmod 600 ceu2016kocsiso.pem
ssh -i ~/ceu2016kocsiso.pem hadoop@ec2-52-29-64-59.eu-central-1.compute.amazonaws.com
sudo pip install s3cmd
```

### Download
```
[hadoop@ip-172-31-5-23 ~]$ s3cmd ls
2016-04-14 19:15  s3://ceu2016kocsiso
[hadoop@ip-172-31-5-23 ~]$ s3cmd ls s3://zoltanctoth/ceu/
2016-03-28 19:23      1115   s3://zoltanctoth/ceu/satisfaction.txt
2016-03-28 19:21     43479   s3://zoltanctoth/ceu/signup-sample.log
2016-03-28 19:20  46241691   s3://zoltanctoth/ceu/signup.log
2016-03-28 19:23     54368   s3://zoltanctoth/ceu/web.log
[hadoop@ip-172-31-5-23 ~]$ s3cmd get s3://zoltanctoth/ceu/signup.log signup.log
download: 's3://zoltanctoth/ceu/signup.log' -> 'signup.log'  [1 of 1]
download: 's3://zoltanctoth/ceu/signup.log' -> 'signup.log'  [1 of 1]
 46241691 of 46241691   100% in    7s     6.29 MB/s  done
```
### Registration sources
Using the `head` function the file can be reviewed: 
```
[hadoop@ip-172-31-5-23 ~]$ head signup.log 
2000-01-08 00:00:01 registration 124 organic
2000-01-08 00:00:02 registration 125 organic
2000-01-08 00:00:03 registration 126 organic
2000-01-08 00:00:04 registration 127 organic
2000-01-08 00:00:05 registration 128 display
2000-01-08 00:00:06 registration 129 organic
2000-01-08 00:00:07 registration 130 organic
2000-01-08 00:00:08 registration 131 organic
2000-01-08 00:00:09 registration 132 sem
2000-01-08 00:00:09 payment 132 DE 59
```
The uniq sources can be retreived via

1. reading the file by `cat signup.log`
2. then filetring for registration raws by `grep registration`
3. selecting only the 5 column separated by spaces by `cut -d" " -f5`
4. sorting the result `sort`
5. keeping only the uniq values by `uniq`
6. finally writing the result into a file by `>`

```
[hadoop@ip-172-31-5-23 ~]$ cat signup.log | grep registration | cut -d" " -f5 | sort | uniq > sources.txt
[hadoop@ip-172-31-5-23 ~]$ cat sources.txt 
blog
display
organic
sem
```

### Upload
```
[hadoop@ip-172-31-5-23 ~]$ s3cmd put sources.txt s3://ceu2016kocsiso/sources.txt
upload: 'sources.txt' -> 's3://ceu2016kocsiso/sources.txt'  [1 of 1]
 25 of 25   100% in    0s    51.86 B/s  done
upload: 'sources.txt' -> 's3://ceu2016kocsiso/sources.txt'  [1 of 1]
 25 of 25   100% in    0s   498.48 B/s  done
upload: 'sources.txt' -> 's3://ceu2016kocsiso/sources.txt'  [1 of 1]
 25 of 25   100% in    0s   338.19 B/s  done
```
### Commands
```
[hadoop@ip-172-31-5-23 ~]$ nano unixsolution.txt
[hadoop@ip-172-31-5-23 ~]$ s3cmd put unixsolution.txt s3://ceu2016kocsiso/unixsolution.txt
upload: 'unixsolution.txt' -> 's3://ceu2016kocsiso/unixsolution.txt'  [1 of 1]
 80 of 80   100% in    0s   170.85 B/s  done
upload: 'unixsolution.txt' -> 's3://ceu2016kocsiso/unixsolution.txt'  [1 of 1]
 80 of 80   100% in    0s     4.30 kB/s  done
upload: 'unixsolution.txt' -> 's3://ceu2016kocsiso/unixsolution.txt'  [1 of 1]
 80 of 80   100% in    0s  1302.13 B/s  done
```

## Pig
### Amounts
#### Script
```
[hadoop@ip-172-31-5-23 ~]$ pig
grunt> log = LOAD 's3://ceu2016kocsiso/signup.log' USING PigStorage(' ') AS (date:chararray, 
grunt> payments = FILTER log BY action == 'payment';
grunt> neatherland= FILTER payments BY country == 'NL';
grunt> amounts= FOREACH neatherland GENERATE amount;
grunt> amount      = DISTINCT amounts;
grunt> DUMP amount;
(59)
(99)
grunt> STORE amount INTO 'amounts.txt' USING PigStorage(',');
grunt> quit
```
#### Upload result
```
[hadoop@ip-172-31-5-23 ~]$ hadoop fs -ls
Found 1 items
drwxr-xr-x   - hadoop hadoop          0 2016-04-14 20:35 amounts.txt
[hadoop@ip-172-31-5-23 ~]$ hadoop fs -get amounts.txt amounts.txt
[hadoop@ip-172-31-5-23 ~]$ s3cmd put amounts.txt/ s3://ceu2016kocsiso/amounts.txt
amounts.txt/  part-r-00000  _SUCCESS      
[hadoop@ip-172-31-5-23 ~]$ s3cmd put amounts.txt/part-r-00000 s3://ceu2016kocsiso/amounts.txt
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s    12.86 B/s  done
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s   296.31 B/s  done
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s    77.59 B/s  done
```
#### Commands
```
[hadoop@ip-172-31-5-23 ~]$ nano amounts.pig
[hadoop@ip-172-31-5-23 ~]$ s3cmd put amounts.pig s3://ceu2016kocsiso/amounts.pig
upload: 'amounts.pig' -> 's3://ceu2016kocsiso/amounts.pig'  [1 of 1]
 409 of 409   100% in    0s   837.04 B/s  done
upload: 'amounts.pig' -> 's3://ceu2016kocsiso/amounts.pig'  [1 of 1]
 409 of 409   100% in    0s    20.75 kB/s  done
upload: 'amounts.pig' -> 's3://ceu2016kocsiso/amounts.pig'  [1 of 1]
 409 of 409   100% in    0s     4.58 kB/s  done
```

### Satisfaction
#### web.log
```
[hadoop@ip-172-31-4-179 ~]$ head web.log
2015-08-12 07:37:43 registration organic dorothy@gmail.com 188.121.177.181
2015-08-12 07:37:43 payment dorothy@gmail.com US 99
2015-08-12 15:15:26 registration organic laura@hotmail.com 191.90.144.105
2015-08-12 22:53:09 registration organic barbara@hotmail.com 208.229.189.57
2015-08-13 06:30:52 registration organic gary@mail.com 230.45.226.70
2015-08-13 06:30:52 payment gary@mail.com IN 99
2015-08-13 14:08:35 registration organic michelle@lycos.com 125.164.174.190
2015-08-13 14:08:35 payment michelle@lycos.com IN 99
2015-08-13 21:46:18 registration organic carol@hotmail.com 28.187.253.57
2015-08-14 05:24:01 registration sem eric@aol.com 136.178.219.192
```
#### satisfaction.txt 
```
[hadoop@ip-172-31-4-179 ~]$ head satisfaction.txt 
barbara@hotmail.com 81
michelle@lycos.com 91
laura@aol.com 87
charles@gmail.com 72
ronald@gmail.com 78
sarah@inbox.com 65
karen@outlook.com 86
patricia@outlook.com 80
donna@hotmail.com 88
thomas@yahoo.com 57
```
#### Analysis
The average satisfcation can be calcuated using only `satisfaction.txt` file. The `web.log` is not required: 
1. The space seperated file is loaded into Pig (lazy evaluation)
2. The domain name is selected from the email address as a substring using the indexes of the `@` character and the last `.` character
3. The result is grouped by domain 
4. The mean score is calculated by domain
5. The result list is dumped to the screen and stored to a file order by the mean score in descending order
```
grunt> satisfaction = LOAD 's3://ceu2016kocsiso/satisfaction.txt' USING PigStorage(' ') AS (email:chararray, score:int);
grunt> domains = FOREACH satisfaction GENERATE SUBSTRING(email,INDEXOF(email,'@') + 1,LAST_INDEX_OF(email, '.')) as domain, score;
grunt> groups = GROUP domains BY domain;
grunt> mean = FOREACH groups GENERATE group as domain, AVG(domains.score) as mean;
grunt> mean = ORDER mean BY mean DESC;
grunt> dump mean;
(gmail,81.14285714285714)
(lycos,80.5)
(inbox,75.71428571428571)
(hotmail,74.28571428571429)
(gmx,74.0)
(outlook,73.85714285714286)
(aol,73.6)
(mail,69.7)
(yahoo,69.5)
grunt> STORE mean INTO 'happier.txt' USING PigStorage(',');
```

#### Upload

**If you consider yourself as a technical person:**

1. There is a registration/payment log with email addresses in `s3://zoltanctoth/ceu/web.log`.
2. The results of a customer satisfaction survey are stored in `s3://zoltanctoth/ceu/satisfaction.txt`.
3. We have a hypothesis that those users who have *gmail.com* email addresses are in general more satisfied with our services than those,
who come with an *outlook.com* email address. Is this right? What can you see in the data?
4. Upload the two solutions into your bucket with the name: `happier.pig` and `nl.pig`.
5. Upload your analysis (what you think and why) in text format into your bucket with the name `emailanalysis.txt`.

 You will need a few commands here which we have not covered. [The Pig Latin Reference manual](http://pig.apache.org/docs/r0.14.0/basic.html) is a great piece of documentation. Also, you can find [very similar pig scripts](https://github.com/zoltanctoth/bigdata-training/tree/master/pig/solutions) on my github.

Spark
---
**If you consider yourself as a non-technical person:**

1. Answer this question using Jupyter (iPython) Notebook on your virtual machine:

 What are the different amounts that were payed by users from the Netherlands (country code NL)? The input file is at `file:///home/bigdata/training/datasets/registration.log`. Show the results on the screen. Download the notebook (File -> Download as) in *HTML* fromat and upload it into your bucket with the name `spark-notebook.html`. 
 
 *Hint: If you have an RDD (called e.g. `rdd`) and you want to get rid of the duplicate values in it, simply use the `rdd.distinct()` transformation. I cover this in the recap video.*

**If you consider yourself as a technical person:**

1. There is a registration/payment log with email addresses in `s3://zoltanctoth/ceu/web.log`.
2. The results of a customer satisfaction survey are stored in `s3://zoltanctoth/ceu/satisfaction.txt`.
3. We have a hypothesis that those users who have *gmail.com* email addresses are in general more satisfied with our services than those,
who come with an *outlook.com* email address. Is this right? What can you see in the data? Use Spark on AWS.
4. Show the results on the screen. Download the notebook (File -> Download as) in *HTML* fromat and upload it into your bucket with the name `spark-advanced-notebook.html`. 
 You will need a few commands here which we have not covered. [The Spark programming guide manual](http://spark.apache.org/docs/latest/programming-guide.html#working-with-key-value-pairs) will help, especially the part with *Working with key value pairs*.

*Pro tip: You can avoid the key value pairs RDD part if you use dataframes and spark SQL as early as possible*


Finishing up
----------
1. Please go to the AWS Management Console and make sure that
I will be able access the files you uploaded (Select the files in the S3 browser and press *Actions* -> *Make Public*)
2. Send me an email to *zoltanctoth@gmail.com* that you are done.
