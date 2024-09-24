### Null Handling in Data Flow

In the ETL (Extract, Transform, Load) process, handling `NULL` values is a crucial task. There are several methods to achieve this:

- **Transformation during extraction**: You can convert `NULL` values in SQL using the `ISNULL` or `COALESCE` function. For example:
   ```sql
   SELECT ISNULL(column, 'Unknown') AS column
   FROM source_table
   ```

- **Transformation during the ETL process**: During the transformation phase, you can add a transformation step to replace `NULL` values with a default value. In SSIS, you can use the `Derived Column` component to replace `NULL` values with 'Unknown'.

- **Adding an Unknown column**: For each column that may have `NULL` values, add a default or "Unknown" value to that column in the table and set the ID of that column in the `Derived Column`. This way, if a `NULL` value is encountered, it will take that value.

In cases where a row in the Fact table has a record that does not correspond to one of the Dimension columns, you can create default records for each Dimension. This method allows you to create exception reports to track the number of Facts assigned to such rows.

For example, if the `Dim Employee` table has a nullable customer column, you can add an Unknown column to it and set the ID of that unknown record in the `Derived Column`.

For instance, if a date column has a `NULL` value, you can create a default record in your Dimension table and reference the default record when a `NULL` value is found. Similarly, if a data column is empty or `NULL`, you can convert these values to a specific default value.
