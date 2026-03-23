---
title: "Add Athena Partition for ELB Access Logs"
date: 2018-07-31T17:15:00-05:00
draft: false
categories: ["Build Log"]
tags: ["athena", "aws", "cloudwatch", "elastic-load-balancing", "github", "lambda"]
description: "If you've worked on a load balancer, then at some point you've been witness to the load balancer taking the blame for an application problem (like a rite of ..."
image: ""
---

If you've worked on a load balancer, then at some point you've been witness to the load balancer taking the blame for an application problem (like a rite of passage). This used to be difficult to exonerate, but with [AWS Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/) you can capture Access Logs (Classic and Application only) and very quickly identify whether the load balancer contributed to the problem.

Much like any log analysis, the volume of logs and frequency of access are key to identify the best log analysis solution. If you have a large store of logs but infrequently access them, then a low-cost option is [Amazon Athena](https://aws.amazon.com/athena/). Athena enables you to run SQL-based queries against your data in S3 without an ETL process. The data is durable and you only pay for the volume of data scanned per query. AWS also includes documentation and templates for querying [Classic Load Balancer logs](https://docs.aws.amazon.com/athena/latest/ug/elasticloadbalancer-classic-logs.html) and [Application Load Balancer logs](https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html).

This is a great model, but with a potential flaw--as the data set grows in size, the queries become slower and more expensive. To remediate, Amazon Athena allows you to [partition your data](https://docs.aws.amazon.com/athena/latest/ug/partitions.html). This restricts the amount of data scanned, thus lowering costs and increasing speed of the query.

ELB Access Logs store the logs in S3 using the following format:

```
`s3://bucket[/prefix]/AWSLogs/{{AccountId}}}}/elasticloadbalancing/{{region}}/{{yyyy}}/{{mm}}/{{dd}}/{{AccountId}}_elasticloadbalancing_{{region}}_{{load-balancer-name}}_{{end-time}}_{{ip-address}}_{{random-string}}.log`
```

Since the prefix does not pre-define partitions, the partitions must be created manually. Instead of creating partitions *ad-hoc*, create a CloudWatch Scheduled Event that runs daily targeted at a Lambda function that adds the partition. To simplify the process, I created [buzzsurfr/athena-add-partition](https://github.com/buzzsurfr/athena-add-partition).

[![](https://theodorejsalvo.files.wordpress.com/2021/11/athena-add-partition-architecture.png?w=641)](https://theodorejsalvo.files.wordpress.com/2021/11/athena-add-partition-architecture.png)

This project is both the Lambda function code and a CloudFormation template to deploy the Lambda function and the CloudWatch Scheduled Event. Logs are sent from the Load Balancer into a S3 bucket. Daily, the CloudWatch Scheduled Event will invoke the Lambda function to add a partition to the Athena table.

Using the partitions requires modifying the SQL query used in the [Athena console](https://console.aws.amazon.com/athena/home). Consider the basic query to return all records: `SELECT * FROM logs.elb_logs`. Add/append to a `WHERE` clause including the partition keys with values. For example, to query only the records for July 31, 2018, run:

```
SELECT *
FROM logs.elb_logs
WHERE
  (
    year = '2018' AND
    month = '07' AND
    day = '31'
  )
```

This query with partitions enabled restricts Athena to only scanning

```
`s3://bucket/prefix/AWSLogs/{{AccountId}}/elasticloadbalancing/{{region}}/2018/07/31/`
```

instead of

```
`s3://bucket/prefix/AWSLogs/{{AccountId}}/elasticloadbalancing/{{region}}/`
```

resulting in a significant reduction in cost and processing time.

Using partitions also makes it easier to enable other Storage Classes like Infrequent Access, where you pay less to store but pay more to access. Without partitions, every query would scan the bucket/prefix and potentially cost more due to the access cost for objects with Infrequent Access storage class.

This model can be applied to other logs stored in S3 that do not have pre-defined partitions, such as [CloudTrail logs](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html), [CloudFront logs](https://docs.aws.amazon.com/athena/latest/ug/cloudfront-logs.html), or for other applications that export logs to S3, but don't allow modifications to the organizational structure.