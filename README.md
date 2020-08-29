`METHOD4` 2.1.9
============

Method4 is a PL/SQL application to run dynamic SQL in SQL.

## Examples

**DYNAMIC_QUERY** Method4 can run queries generated by queries.  This can solve challenging problems such as "count the rows of every table", all within a single SQL statement.

    select * from table(method4.dynamic_query(
        q'[
            select replace(
                q'!
                    select '#TABLE_NAME#' table_name, count(*) a from #TABLE_NAME#
                !', '#TABLE_NAME#', table_name) sql_statement
            from user_tables
            where table_name like 'TEST%'
        ]'
    ));
    
    TABLE_NAME                         A
    ------------------------- ----------
    TEST                           19765
    TEST1                              1
    TEST2                              1
    TEST3                              1
    ...

**QUERY** This mode simply returns the results of a string literal:

    select * from table(method4.query('select * from dual'));
    
    D
    -
    X

**POLL_TABLE** This mode periodically polls a table and returns new rows until a condition is met.  This can be useful for querying tables populated by an asynchronous process.  See the package for more information on the parameters.

    create table table1(a number) rowdependencies;
    insert into table1 values(1);
    commit;
    
    select * from table(method4.poll_table(
       p_table_name              => 'table1',
       p_sql_statement_condition => 'select 1 from dual',
       p_refresh_seconds         => 2
    ));
    
    Results:
             A
    ----------
             1

All modes convert LONGs to CLOBs, which can be helpful when querying the data dictionary.  The string-literal-mode is a good way to understand how Data Cartridge and the ANY types work together.  Compare the types `method4_ot` with `method4_dynamic_ot` to see how the query string can be intercepted and converted to do something interesting.

These queries are powerful but they can also be confusing because of all the quotation marks required to build strings inside strings.  Simplify your queries with the alternative quoting syntax (the "q" strings) and templating (use REPLACE instead of concatenating strings).

## Notes

Method4 is based on the Dictionary Long Application, (c) Adrian Billington www.oracle-developer.net.  Much of this code contains advanced methods thoroughly discussed on his website, http://www.oracle-developer.net/display.php?id=422

Method4 is a simpler, more generic version of that application.  It can be useful for adhoc queries in highly dynamic environments.  For example, an application where the schemas, tables, and columns are table-driven and only known at run time.

Use this program with caution.  Few programs need to be this dynamic.  This package will be slower and buggier than regular SQL.

## License

This project uses the MIT License.

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
