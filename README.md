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

Pig
---
1. Create an EMR cluster and start pig.
2. Copy s3://zoltanctoth/ceu/signup.log into your own bucket using `s3cmd`.

**If you consider yourself as a non-technical person:**

1. Answer this question using pig:

 What are the different amounts that were payed by users from the Netherlands (country code NL)? DUMP the file on the screen. STORE these amounts in a file called `amounts.txt` in your bucket (you will need to use the STORE command with the path `s3://<yourbucketname>/amounts.txt`). (If you do it correctly, the file will contain two lines:
```
59
99
```
2. Put your final set of commands you executed in pig in a text editor on your personal computer, and use the AWS Management Console to upload this file into your bucket, using the name `amounts.pig`.

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
