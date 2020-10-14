---
layout: "post"
title:  "Extract Transform and Load"
tags:   etl db
---

> A critical component of Business Intelligence (BI) process: Extract, Transform and Load (ETL).

* * *

ETL is the ordered sequence of steps which retrieves data from production
database, prepare it to be useful for analysis and finally store it in a
separated database for decision-making tasks.

# Motivation

At the beginning of information systems, information was viewed as the result of the productive process and was solely responsible to support it. In an e-commerce example, clothes purchased by a customer were only used by the logistic system to deliver them. Hence it hadn't had any value for future business planning.

This concept has changed since then. Information is now considered a critical asset and is treated as such.

Back to the e-commerce example, records of purchases can now be used
to analyze customers' preferences, thereby providing services and recommendations focused on the customers' needs.

However, the goal of using the information as a guiding factor for decision making
depends on proper systems that are responsible for consuming the data obtained from
productive process, storing, and deriving knowledge from it, which can then
be used by the business's planners.

In this context, we have Data Analytics, a field that aims to extract knowledge from data, so that it
can be used as input for decision making.

As a first step, we need to retrieve the data, clean it up, and then store in a proper place, e.g. in a data warehouse.

# Extract, Transform and Load

Most of the time, we need to adapt our resources to our needs.

Usually, databases are optimized for a given set of goals: consistency, no redundancy, fast lookups, etc. For instance, by employing the Online Transaction Processing (OLTP) model.
A database that follows OLTP is pretty good at executing real-time processing, e.g. lookups by primary key.

Meanwhile, in BI systems, batch processing is usually acceptable, e.g by using the Online Analytical Processing (OLAP) model.

Therefore, sometimes we need to translate from OLTP to OLAP models, and that's when ETL comes in handy.

We can summarize ETL by three sequential steps:

  1.  **Extract** -- Fetchs data from the production database.
  2.  **Transform** -- Cleans the data, maybe denormalize and convert it to a more suitable representation.
  3.  **Load** -- Stores the transformed data into the data warehouse, which is a separate database, preferably isolated from the production database.

For ETL, there is plenty of tools which assist in the process, here I will be using Pentaho Data Integration (Kettle) [2].

# Example

I will introduce Kettle as a tool for ETL, without interacting with a proper relational database. Instead, it'll fetch some data from a Comma Separated Value (CSV) file, filter it, aggregate and then write the result into another CSV file.

I've tested this example using the Kettle 4.2.0-stable.

The use-case:

> We have a CSV containing the number of sales by salesperson, aggregated by years.
>
> Our goal is to summarize the number of sales by year made by the salesperson A and B.

Kettle is a Java tool for parallel data processing.

Firstly, we have a transformation which is groups elements built for the data processing.

Secondly, the data flows through the pipeline, from the beginning of the transformation towards its end.

Finally, every element in Kettle is one of two possible kinds: steps
or hops. Steps are responsible for processing the incoming data flow
and producing the output data flow. And steps are connected through hops.

We have many built-in steps, e.g. filters, database lookup, group-by, sort-by, folding, etc. Moreover, it's possible to
customize transformations by embedding Java or JavaScript code inside custom steps.

In the example, we have the transformation made by these steps:

  1.  **INPUT** -- Reads the input CSV file of sales information.
  2.  **FILTER_CODES** -- Filters out rows which are not of codes A and B.
  3.  **D** -- Discards rows which were filtered by FILTER_CODES.
  4.  **SORT_BY_YEARS** -- Sorts the rows by years in descending manner.
  5.  **AGGREGATE_YEAR** -- Sums the amounts by years.
  6.  **OUTPUT** -- Generates the CSV output file.

# Conclusion

In this article, we've learned what the basic principles of ETL are, its importance for BI, and we used Kettle as a tool to illustrate ETL.

The full example containing the input, transformation and output can be obtained [here](https://github.com/rvarago/etl-example).

# References

[1] GROSSMANN, W. and RINDERLE-MA, S. Fundamentals of Business Intelligence.
1E. Published By Springer-Verlag Berlin Heidelberg.

***
*Originally published at [https://medium.com/@rvarago](https://medium.com/@rvarago)*
