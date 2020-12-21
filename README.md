`METHOD4` 2.2.2
============

Method4 is a PL/SQL application to run dynamic SQL in SQL.

## Examples

**DYNAMIC_QUERY** Run queries generated by queries. With a single SQL statement you can solve challenging problems such as counting the number of rows of every table.

    select * from table(method4.dynamic_query(
        q'[
            select replace(
                q'!
                    select '#TABLE_NAME#' table_name, count(*) row_count from #TABLE_NAME#
                !', '#TABLE_NAME#', table_name) sql_statement
            from user_tables
            where table_name like 'TEST%'
        ]'
    ));
    
    TABLE_NAME                 ROW_COUNT
    -------------------------  ---------
    TEST                           19765
    TEST1                              1
    TEST2                              1
    TEST3                              1
    ...


**PIVOT** Dynamically transpose rows into columns. The last column defines the values and the second-to-last column defines the column names. The default aggregation function is `MAX`, which works well for common entity-attribute-value queries like the one below:

    select * from table(method4.pivot(
        q'[
            select 1 user_id, 'Date of Birth', '2010-05-27' from dual union all
            select 1 user_id, 'Name',          'Elliott'    from dual union all
            select 2 user_id, 'Name',          'Oliver'     from dual union all
            select 2 user_id, 'Gender',        'Male'       from dual
        ]'
    ));

    USER_ID  Date of Birth  Gender  Name
    -------  -------------  ------  -------
          1  2010-05-27             Elliott
          2                 Male    Oliver

Use the second parameter to change the aggregation function. For example, the below query counts and compares the number of objects per object type, per user:

    select * from table(method4.pivot(
        q'[
            select owner, object_type, object_name
            from all_objects
            where owner like 'SYS%'
        ]',
        'count'))
    order by owner;

    OWNER   CLUSTER  CONSUMER GROUP  DESTINATION  DIRECTORY  EDITION  ...
    ------  -------  --------------  -----------  ---------  -------  ...
    SYS          10              18            2         13        1  ...
    SYSTEM        0               0            0          0        0  ...


**QUERY** This mode simply returns the results of a string literal:

    select * from table(method4.query('select * from dual'));
    
    D
    -
    X


All modes convert LONGs to CLOBs, which can be helpful when querying the data dictionary. The string-literal-mode is a good way to understand how Data Cartridge and the ANY types work together. Compare the types `method4_ot` with `method4_dynamic_ot` to see how the query string can be intercepted and converted to do something interesting.

These queries are powerful but they can also be confusing because of all the quotation marks required to build strings inside strings. Simplify your queries with the alternative quoting syntax (the "q" strings) and templating (use REPLACE instead of concatenating strings).

## Installation

Click the "Download ZIP" button, extract the files, CD to the directory with those files, connect to SQL*Plus, and run these commands:

1. Install Method4:

        @install

2. Uninstall Method4:

        @uninstall

3. Install unit tests (optional, only useful for development):

        @tests/install_unit_tests

4. Uninstall unit tests (optional, only useful for development):

        @tests/uninstall_unit_tests

## Notes

Method4 is based on the Dictionary Long Application, (c) Adrian Billington www.oracle-developer.net. Much of this code contains advanced methods thoroughly discussed on his website, http://www.oracle-developer.net/display.php?id=422

Method4 is a simpler, more generic version of that application. It can be useful for adhoc queries in highly dynamic environments. For example, an application where the schemas, tables, and columns are table-driven and only known at run time.

Use this program with caution. Few programs need to be this dynamic. This package will be slower and buggier than regular SQL.

## License

This project uses the MIT License.
