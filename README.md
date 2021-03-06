# Tabulator: Shell scripts for delimited data files

URL: https://github.com/stefan-schroedl/tabulator

Author: Stefan Schroedl

## Date
* 2015/04/03 release 1.2.1
* 2015/03/21 release 1.2
* 2014/10/12 release 1.1.2
* 2012/01/24 release 1.1.1
* 2011/11/25 release 1.1.0
* 2009/06/16 release 1.0.0


## Purpose


Unix/Linux comes with several tools, such as `cut,` `paste,` `join,` `sort,` to
process tabular text data files (a.k.a., *tab-delimited*, *csv*, *tsv*,
or *flat file* format). However, they have shortcomings that often requires
additional scripting and prevents them from being directly used in one-liners.
For example, having to count columns to pass as arguments is cumbersome and
error-prone; `join` needs presorted files, and works with a single key column;
there is no direct 'group-by' functionality.

One remedy would be to load the data into a relational database or noSQL system
(e.g., Hadoop/Pig) first, but these might not be available or be more
time-consuming for short, ad-hoc tasks.

`Tabulator` is a collection of command-line shell tools for Unix/Linux platforms
that build on native shell programs and can be used as filters, but make them more
easy and flexible to use. In particular, they

* allow to reference columns by names rather than position, as indicated by the
  first line in a file; this makes scripts more readable and robust to changes
  in the input data format.
* automate file format recognition (delimiter, compression etc).
* check file format (e.g., consistent number of columns).
* offer expanded functionality, such as SQL-like *relational join* and *group-by*
  operations.

## Installation

Installation is easy -- just unpack the tarball, add the unpacked directory to
your PATH.

## License

Tabulator is licensed under the MIT LICENSE, see LICENSE for details.

## Documentation

Here is a brief list of the programs together with their main functionality.
Each one provides more documentation and examples when called with the `-m` or
`-h` options. A common assumption is that the first line in input files contains
the column names.

* `tblcat:` concatenate files of the same data format without header repetition.
* `tblcmd:` execute a program on the body of a file (e.g., `sort`, `uniq`),
  without affecting the header.
* `tbldesc:` for each column, summarize its type (e.g., char, int, float),
  percentage of undefined values, min/max/mean/median/std, etc. Can also provide
  correlation coefficients with a target column.

> Example: Suppose `file` is

        name,house_nr,height,shoe_size
        arthur,42,6.1,11.5
        berta,101,5.5,8
        chris,333,5.9,10
        don,77,5.9,12.5

> Then `tbldesc file` prints:

        summarizing file_desc (4 lines, target column: shoe_size)
        field name     type char% uniq min max avg  std    mse  corr   prob%
        1 name       char 100    4 [arthur; berta; chris; don]
        2 house_nr   int    0    4 42   333 138   114    172   -0.287 71.25
        3 height     float  0    3 5.5  6.1 5.85  0.218  4.89   0.812 18.82
        4 shoe_size  float  0    4 8   12.5 10.5  1.7     0.0   1.0    0.00

* `tblmap:` simple line-wise ("map") computation similar to awk.

> Example: Compute ratio of columns `sales` and `clients` for lines where column
  `region` has value `us`:

    tblmap -s'region=="us"' -c'sales_per_client=sales/client' file

* `tblred:` compute ("reduce") aggregations (e.g., `sum`, `min`, `max`, `avg`,
   etc.) over groups of keys -- similar to the SQL `group by` operator.

> Example:

    tblred -k'region' 'sales_ratio=sales/sum(sales)'
> computes for each line proportion of column `sales` to total sales for all
> lines with the same value of column `region`.

* `tbljoin:` Relational join. In contrast to Unix `join`, input files don't
  need to be pre-sorted, and multiple join columns can be specified.

> Example: Suppose `file1` is

    name,street,house
    zorro,desert road,5
    john,main st,2
    arthur,pan-galactic bypass,42
    arthur,main st,15

> and `file2` is

    name,street,phone
    john,main st,654-321
    arthur,main st,121-212
    john,round cir,123-456

> Then `tbljoin file1 file2` gives

    name,street,house,phone
    arthur,main st,15,121-212
    john,main st,2,654-321

* `tblhist:` computation and ascii-plotting of the histogram of column values.
* `tblsplit:` split a file into several ones based on a column value.

> Example: Suppose `file` is

        continent,country
        americas,us
        americas,mx
        europe,de
        europe,fr

> Then `tblsplit -rk'continent' file` generates two files,
> `file.select.continent=americas`:

        country
        us
        mx
> and `file.select.continent=europe`:

        country
        de
        fr

* `tblsort:` interface for unix sort.
* `tbltex:` formatting for latex tables.
* `tbltranspose:` transposition of rows and columns.
* `tbluniq:` check for and cut out duplicate columns; also, discover functional
     value dependencies.
* `tblcolumn:` format columns in a more readable way, using the unix 'column'
     program, and aligning and shortening numbers.
* `tblless:` page through formatted column output (calls tblcolumn).

## Implementation Notes

* The scripts have been developed over time to help with various data
  processing tasks, and were not designed from the outset to be released in one
  package. Therefore, some scripts are implemented in Python, and some in Perl;
  and there is a small amount of overlapping functionality.
* For very large files, it is crucial to be able to process them in *streaming*
  mode, i.e., without keeping them entirely in main memory. This is the case for
  `tblmap` (since it translates into an `cut`/`awk` script) and for `tbljoin`
  (using `sort` and `join`), For `tblred`, presort the file first, then run it
  with option `-s`.
* `tblmap`, `tbljoin`, and `tblcmd` work by first translating the command into
  a standalone shell script.

## Limitations

* There is no special interpretation of block delimiters like `'` or `"`; it is
  the user's responsibility to ensure that the column delimiter cannot occur
  within column values.
* `tbljoin`, `tblred`, `tblhist`, `tbluniq`, `tblcat`, and `tbltex` have some
  restrictions when run as a filter (repeated reading is necessary in some cases).
