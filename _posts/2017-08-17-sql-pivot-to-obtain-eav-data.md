---
layout: post
title: 'SQL Pivot to obtain EAV data'
categories: programming
tags: [SQL]
excerpt_separator: <!--more-->
---

Here is a brief post showing how we can use a SQL pivot to obtain EAV data.

Before I go into the actual code and briefly explain how it does it, I will explain the reasoning why such a model was developed.

<!--more-->

### Use Case
Let’s say you are developing a school management system and you are designing a schema to store attributes for a student. A student object will contain a number of attributes, such as their name, date of birth, ethnicity etc. Naturally, it would make sense to create a table, named students with a column for each of the attributes.

However, as the school management system will be sold globally, it becomes obvious that different schools in different countries may wish to store different attributes. For instance, here in the UK, we might store the students’ gender, if they are on free school meals, if they are in foster care, their UPNs etc. Some of these attributes might be of interest to some schools, but it would be unwise to try and foresee every possible attribute in advance, leading to an extremely wide entity with many of its columns being left empty.

The question is how can we store attribute data that describes a certain entity, without knowing how many or which attributes will be suitable?

### EAV (Entity Attribute Value)
The solution is to adopt an EAV model. EAV is based on the mathematical notation of a sparse matrix, which essentially enables the efficient storage for large entities that would likely be highly fragmented otherwise.

### T-SQL Example: 
Below is a table adhering to the EAV pattern:

| StudentID | AttributeName | AttributeValue  |
|-----------|---------------|-----------------|
| 1         | gender        | 'M'             |
| 1         | name          | 'Student1'      |
| 1         | ethnicity     | 'White British' |
| 2         | name          | 'Student2'      |
| 2         | age           | '19'            |
| 3         | name          | 'Student3'      |
| ...       | ...           | ...             |

We could store any number of entities. The data type of attributes could be all stored as nvarchar or we could set it to a SQL_Variant.

The problem with the data above is that its not all that readable. We could always parse it and build up the student object programmatically, but this would be a computationally costly operation, particularly if hundred of thousands of rows are returned. Fortunately for us, since SQL 2005, Pivots transform the data from rows into columns, so the data above will appear as:

| StudentID | Gender | Name       | Ethnicity       | Age  |
|-----------|--------|------------|-----------------|------|
| 1         | 'M'    | 'Student1' | 'White British' | null |
| 2         | 'F'    | 'Student2' | null            | 20   |
| 3         | 'M'    | 'Student3' | null            | null |
| ...       | ...    | ...        | ...             | ...  |

There are two types of pivots, a dynamic and non-dynamic pivot.

Here is how we could use a **non-dynamic pivot**:

```sql
select *
from 
(
  select StudentID, AttributeName, AttributeValue
  from Students
) data
pivot
(
  max(AttributeValue)
  for AttributeName in ([Gender], [Name], [Ethnicity], [Age], [...])
) piv;
```

However, there are scenarios in which we cannot know the attribute types and rather than hardcoding values, we can make use of a **dynamic pivot**:

```sql
DECLARE @cols AS NVARCHAR(MAX), @query AS NVARCHAR(MAX)

select @cols = STUFF((SELECT ',' + QUOTENAME(AttributeName) 
                    from Students
                    group by AttributeName
                    order by AttributeName
            FOR XML PATH(''), TYPE
            ).value('.', 'NVARCHAR(MAX)') 
        ,1,1,'')

set @query = 'SELECT StudentID,' + @cols + ' from 
             (
                select StudentID, AttributeName, AttributeValue
                from Students
             ) data
            pivot 
            (
                max(AttributeValue)
                for AttributeName in (' + @cols + ')
            ) piv '

execute(@query);
```

The result is identical, however, columns in the second example are dynamic.

Dynamic queries are great, but performance does suffer – In my real use case example, a dynamic pivot executed in approx 20seconds, compared to a non-dynamic pivot executing in 1/3 of a second.

So here we have it, a SQL Pivot! Do note that the EAV model is rather controversial and a quick google search will reveal it as an anti pattern. In my opinion, like in the scenario above, it provides with an elegant solution to otherwise a complex problem. However, should I’ve opted for the dynamic solution (which was, of course, my first choice), performance was completely unacceptable. Use wisely with well-defined queries!