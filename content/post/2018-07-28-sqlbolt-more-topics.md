---
title: SQLBolt 课程学习笔记五 (番外篇)
author: Jackie
date: '2018-07-28'
slug: sqlbolt-more-topics
categories:
  - PostgreSQL
tags:
  - PostgreSQL
  - Code
  - 基础
disable_comments: no
---

## SQL Topic: Subqueries

子查询

You might have noticed that even with a complete query, there are many questions that we can't answer about our data without additional post, or pre, processing. In these cases, you can either make multiple queries and process the data yourself, or you can build a more complex query using SQL subqueries.

一次查询经常回答不了很多问题，除非我们事先对数据有预处理，或者查询后再处理。这时候要么多次查询，要么搞个复杂的查询。例子：

> #### Example: General subquery
> Lets say your company has a list of all Sales Associates, with data on the revenue that each Associate brings in, and their individual salary. Times are tight, and you now want to find out which of your Associates are costing the company more than the average revenue brought per Associate.
>
> First, you would need to calculate the average revenue all the Associates are generating:
>
```sql
SELECT AVG(revenue_generated)
FROM sales_associates;
```
>And then using that result, we can then compare the costs of each of the Associates against that value. To use it as a subquery, we can just write it straight into the WHERE clause of the query:
>
```sql
SELECT *
FROM sales_associates
WHERE salary > 
   (SELECT AVG(revenue_generated)
    FROM sales_associates);
```
>As the constraint is executed, each Associate's salary will be tested against the value queried from the inner subquery.

例子懒得翻译了。

A subquery can be referenced anywhere a normal table can be referenced. Inside a **FROM** clause, you can **JOIN** subqueries with other tables, inside a **WHERE** or **HAVING** constraint, you can test expressions against the results of the subquery, and even in expressions in the **SELECT** clause, which allow you to return data directly from the subquery. They are generally executed in the same logical order as the part of the query that they appear in, as described in the last lesson.

子查询可以跟一个普通的表格一样使用。**FORM** 从句里可以使用子查询 **JOIN** 其他表格，**WHERE** 或者 **HAVING** 语句里也可以对子查询的结果做判断，子查询甚至可以用在 **SELECT** 语句里，这时候子查询直接返回数据。（晕了晕了，没搞清楚这个 **SELECT** 说的哪种情况）


Because subqueries can be nested, each subquery must be fully enclosed in parentheses in order to establish proper hierarchy. Subqueries can otherwise reference any tables in the database, and make use of the constructs of a normal query (though some implementations don't allow subqueries to use  **LIMIT** or **OFFSET**).

子查询可以嵌套组合，每个子查询必须用括号包围以保证正确的层次关系。子查询可以引用数据库中的任何表，并使用普通查询的构造（尽管某些实现不允许子查询使用 **LIMIT** 或 **OFFSET**）。

### Correlated subqueries

A more powerful type of subquery is the correlated subquery in which the inner query references, and is dependent on, a column or alias from the outer query. Unlike the subqueries above, each of these inner queries need to be run for each of the rows in the outer query, since the inner query is dependent on the current outer query row.

更为强大的子查询是内查询引用了或者依赖与外查询的数据列或别名的这种相关联的子查询。和上面的子查询不同之处在于，这时候内查询依赖与外查询数据行，因此内查询必须在外查询每一行上都执行一次。

说得有点绕，意思大概清楚，翻译得有点词不达意。看例子吧：

> #### Example: Correlated subquery
> Instead of the list of just Sales Associates above, imagine if you have a general list of Employees, their departments (engineering, sales, etc.), revenue, and salary. This time, you are now looking across the company to find the employees who perform worse than average in their department.
>
> For each employee, you would need to calculate their cost relative to the average revenue generated by all people in their department. To take the average for the department, the subquery will need to know what department each employee is in:
```sql
SELECT *
FROM employees
WHERE salary > 
   (SELECT AVG(revenue_generated)
    FROM employees AS dept_employees
    WHERE dept_employees.department = employees.department);
```

These kinds of complex queries can be powerful, but also difficult to read and understand, so you should take care using them. If possible, try and give meaningful aliases to the temporary values and tables. In addition, correlated subqueries can be difficult to optimize, so performance characteristics may vary across different databases.

这种复杂的查询十分强大，但同时也降低了可读性，提高了理解难度，所以用的时候应该多加小心并且对临时的值和表格使用合适的别名。另外，关联子查询的优化也很困难，因为不同数据库的性能表现可能也不尽相同。


### Existence tests

When we introduced **WHERE** constraints in [Lesson 2: Queries with constraints](https://sqlbolt.com/lesson/select_queries_with_constraints), the **IN** operator was used to test whether the column value in the current row existed in a fixed list of values. In complex queries, this can be extended using subqueries to test whether a column value exists in a dynamic list of values.

在第二节课介绍带限制条件的 **WHERE** 查询时，我们用 **IN** 来判断某一列中当前行的值是否在一个固定值列表中。在复杂的查询中，这一用法可以拓展为用子查询判断某一列的值是否存在于一个动态值列表中。语法：

```sql
-- Select query with subquery constraint
SELECT *, …
FROM mytable
WHERE column
    IN/NOT IN (SELECT another_column FROM another_table);
```

When doing this, notice that the inner subquery must select for a column value or expression to produce a list that the outer column value can be tested against. This type of constraint is powerful when the constraints are based on current data.

可以注意到，此时子查询需要选定一列值或者一个由表达式产生的值列表给外查询提供判断依据。这种类型的限制条件在限制条件本身也作用于当前数据时十分强大。

## SQL Topic: Unions, Intersections & Exceptions

When working with multiple tables, the **UNION** and **UNION ALL** operator allows you to append the results of one query to another assuming that they have the same column count, order and data type. If you use the **UNION** without the **ALL**, duplicate rows between the tables will be removed from the result.

同时操作多个表格的时候，**UNION** 和 **UNION ALL** 可以在多个查询的结果具有相同的列数、列的顺序和数据类型时把它们连接到一起。只使用 **UNION** 不加 **ALL** 的时候，结果中的重复行会被移除。

```sql
-- Select query with set operators
SELECT column, another_column
   FROM mytable
UNION / UNION ALL / INTERSECT / EXCEPT
SELECT other_column, yet_another_column
   FROM another_table
ORDER BY column DESC
LIMIT n;
```

In the order of operations as defined in [Lesson 12: Order of execution](https://sqlbolt.com/lesson/select_queries_order_of_execution), the **UNION** happens before the  **ORDER BY** and **LIMIT**. It's not common to use **UNIONs**, but if you have data in different tables that can't be joined and processed, it can be an alternative to making multiple queries on the database.

在第 12 节课提到的执行顺序里，**UNION** 的执行早于 **ORDER BY** 和 **LIMIT**。使用 **UNION** 并不是很常见，但是如果你的数据分散在不同的表格里并且不允许处理或者合并的话，这确实是一种避免多次查询的办法。

Similar to the **UNION**, the **INTERSECT** operator will ensure that only rows that are identical in both result sets are returned, and the **EXCEPT** operator will ensure that only rows in the first result set that aren't in the second are returned. This means that the **EXCEPT** operator is query order-sensitive, like the **LEFT JOIN** and **RIGHT JOIN**.

和 **UNION** 类似，**INTERSECT** 会返回多个结果间共有的行，**EXCEPT** 则只会返回第一个结果中有而第二个结果中没有的行。所以 **EXCEPT** 是查询顺序敏感的操作，和 **LEFT JOIN** 和 **RIGHT JOIN** 一样。

Both **INTERSECT** and **EXCEPT** also discard duplicate rows after their respective operations, though some databases also support **INTERSECT ALL** and **EXCEPT ALL** to allow duplicates to be retained and returned.

**INTERSECT** 和 **EXCEPT** 也都会去掉结果中的重复行，但有的数据库支持通过  **INTERSECT ALL** 和 **EXCEPT ALL** 保留重复行。

THE END

------

这次真的结束了，终于。
但是这一篇翻译得很不走心，几乎是字面翻译。主要是没有例子，但是理论性的。以后使用得多了知道说的什么了有空再来来改改吧，先这样。 PEACE。