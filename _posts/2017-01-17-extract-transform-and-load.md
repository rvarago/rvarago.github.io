---
layout:	"post"
title:	"Extract Transform and Load"
---

* * *

Hello,

In this article, I'm going to present a key part of Business Intelligence (BI)
process: Extract, Transform and Load (ETL).

ETL is the ordered sequence of steps which retrieves data from production
database, prepare it to be useful for analysis and finally store it in a
separated database for decision making tasks.

### Motivation and Description

The competition for market share between companies has increased, which
demands improvements on business processes and strategies.

Without it, the strive to survive will be useless, hence the company has a
considerable chance of bankrupt. One manner to achieve this improvement is by
the introduction of information systems, which automates business processes.
For example, instead of having a physical store to sell clothes, a company
might develop its e-commerce to do it at internet. Through it, the customer
can buy clothes, share his purchases with friends in social networks,
therefore spreading the company's name and market value too.

In the beginning of these systems, information was viewed as a result of the
productive process and had the only purpose to support it. In the e-commerce
example, the clothes purchased by a customer were only used by the logistic
system to deliver them. Hence it hadn't any value for business planning.

This conception has changed in the recent years and information is now an
asset and should be treated and used as such.

Returning to our e-commerce example, the registered purchases can now be used
to analyse customers preferences, thereby developing specific services, like
special offers, focused always on customers needs. The use of information for
business planning, is known as BI.

However, this objective of using information as a guide for decision making,
depends on having the proper systems to consume this information obtained from
productive process and manipulate it, generating valuable knowledge, which can
be studied by the business's planners.

In this context, comes up Data Analytics as a relatively new field of study,
which aims to extract useful knowledge for decision making from the huge
amount of data produced by daily productive process. Nonetheless, first we
need retrieve the data from somewhere, clean it and store in the data
warehouse which is a separate database for the use of the data in BI.

### Extract, Transform and Load

Most of the time, we need adapt our resources to our needs, this is not
different in Information Technology related fields.

In this way, database are commonly made to perform well in the daily tasks
processed by information systems: maintain consistency in data, avoid
redundancy to save disk space etc. The most important kind or arrangement to
achieve it is the well known Online Transaction Processing (OLTP) model.
Basically, a database which follows OLTP, is very good to execute a huge
amount of queries that return few registers, mainly find by primary key.

Meanwhile, in BI we're looking for execution of few queries which returns a
huge amount of registers. The generally considered standard model for this
kind of task is Online Analytical Processing (OLAP).

Therefore, we need to translate the OLTP data made in production process to
the OLAP data necessary to business planning process. Here, arises the ETL,
which consists on three sequential steps:

  1.  **Extract  **-- Fetch data from production database, applying filters based on the project needs
  2.  **Transform  **-- Clean the data, denormalize it and convert between representation
  3.  **Load  **-- Store the the transformed data into the data warehouse, which is a separate database, preferably isolated from production data base

For the ETL, there are some tools which help in the process, one of them is
the Pentaho Data Integration (Kettle) [2] and it's the chosen one for
illustrate this article.

### Example

For this example, I intend to introduce the Kettle as a tool for ETL, but to
keep it simple, the example will not interact with any relational database.
Instead, it'll fetch data from a Comma Separated Value (CSV) file, filter it,
aggregate and write the result in another CSV file.

I've tested this example using the Kettle at version 4.2.0-stable, but you can
try it in a more recent version and make the necessary modifications.

The scenario is:

We have a CSV of the amount of sales made by some sellers represented by
codes, these sales are separated by years. Our goal is to summarise the amount
of sales by year for the sellers A and B. This scenario easily scales for a BI
typical task which is consolidate huge amount of sales data by period in
dashboards that permit drill down in more granular periods like months or
weeks.

Kettle is basically a Java tool that executes parallel processing of data.

Firstly, we have a transformation which is the group of elements built for the
data processing.

Secondly, the data flow like a wind flux from the beginning of the
transformation until its end.

Finally, every element in Kettle is one of two possible kinds, they're: steps
and hops. Steps are the ones responsible for processing the incoming data flow
and producing another data flow as output and two, or more, steps are
connected with each other by hops.

We have many built-in steps, like filters, database lookup, group by, sort and
in addition, a simple way to customize the transformation behaviour adding
Java or JavaScript code inside custom steps.

In the example, we have the transformation made by the steps:

  1.  **INPUT  **-- Reads the input CSV file of sales information
  2.  **FILTER_CODES  **-- Filters out rows which are not of codes A and B
  3.  **D**  -- Discards rows which were filtered by FILTER_CODES
  4.  **SORT_BY_YEARS  **-- Sorts the rows by years in descending manner
  5.  **AGGREGATE_YEAR**  -- Sums the amounts by years
  6.  **OUTPUT**  -- Generates the CSV output file

### Conclusion

In this article we've learned what ETL is, its importance in business and we
used Kettle as a tool for ETL.

The full example containing the input, transformation and output can be
obtained [here](https://github.com/rvarago/etl-example).

### Bibliography

[1] GROSSMANN, W. and RINDERLE-MA, S. Fundamentals of Business Intelligence.
1E. Published By Springer-Verlag Berlin Heidelberg.


***
*Originally published at https://medium.com/@rvarago*
