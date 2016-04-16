# BAJP-5011---Big-Data-Computing
> Olivér Kocsis

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
#### Upload
```
[hadoop@ip-172-31-5-23 ~]$ hadoop fs -ls
Found 1 items
drwxr-xr-x   - hadoop hadoop          0 2016-04-14 20:35 amounts.txt
[hadoop@ip-172-31-5-23 ~]$ hadoop fs -get amounts.txt amounts.txt
[hadoop@ip-172-31-5-23 ~]$ s3cmd put amounts.txt/part-r-00000 s3://ceu2016kocsiso/amounts.txt
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s    12.86 B/s  done
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s   296.31 B/s  done
upload: 'amounts.txt/part-r-00000' -> 's3://ceu2016kocsiso/amounts.txt'  [1 of 1]
 6 of 6   100% in    0s    77.59 B/s  done
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

Based on the result the mean satisfaction of Gmail users over 81% but it is below 74% for Outlook users. It could be for several reasons, i.e. the Gmail users are used to flat user interfaces, they could be also more tech friendly. The real reason might be revelaed via A/B testing. 

## Spark
### IPython
I have installed the IPython on the AWS EMR cluster using the script below: 
```
sudo yum install -y git
git clone https://github.com/zoltanctoth/bigdata-training.git /home/hadoop/training

sudo pip install s3cmd==1.6.1 ipython[notebook]==4.1.0 py4j==0.9.2

export PYSPARK_SUBMIT_ARGS="--master yarn"
export PYSPARK_DRIVER_PYTHON_OPTS='notebook --port=8889 --no-browser --ip='*''
export IPYTHON=1
pyspark
```
### Amounts
```
signup = sc.textFile('s3://ceu2016kocsiso/signup.log')
payments = signup.map(lambda x: x.split(' ')).filter(lambda x: x[2] == 'payment' and x[4] == 'NL')
amounts = payments.map(lambda x: x[5])
amounts.distinct().collect()
```

```
[u'59', u'99']
```

### Satisfaction
```
satisfaction = sc.textFile('s3://ceu2016kocsiso/satisfaction.txt')
emailscore = satisfaction.map(lambda x: x.split(" "))
domainscore = emailscore.map(lambda x: [x[0][x[0].index('@') + 1 : x[0].rindex('.')], int(x[1])])
df = domainscore.toDF(['domain','score'])
df.registerTempTable("scores")
result = sqlContext.sql("SELECT domain, AVG(score) AS score FROM scores GROUP BY domain ORDER BY score DESC")
result.show()
```

```
+-------+-----------------+
| domain|            score|
+-------+-----------------+
|  gmail|81.14285714285714|
|  lycos|             80.5|
|  inbox|75.71428571428571|
|hotmail|74.28571428571429|
|    gmx|             74.0|
|outlook|73.85714285714286|
|    aol|             73.6|
|   mail|             69.7|
|  yahoo|             69.5|
+-------+-----------------+
```
