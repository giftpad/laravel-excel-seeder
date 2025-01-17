# Excel Seeder for Laravel
> #### Seed your database using CSV files, XLSX files, and more with Laravel

With this package you can save time seeding your database. Instead of typing out seeder files, you can use CSV, XLSX, or any supported spreadsheet file format to load your project's database. There are configuration options available to control the insertion of data from your spreadsheet files.

This project was forked from [laravel-csv-seeder](https://github.com/jeroenzwart/laravel-csv-seeder) and rewritten to support processing multiple input files and to use the [PhpSpreadsheet](https://github.com/PHPOffice/PhpSpreadsheet) library to support XLSX and other file formats.

### Features

- Support CSV, XLS, XLSX, ODF, Gnumeric, XML, HTML, SLK files through [PhpSpreadsheet](https://github.com/PHPOffice/PhpSpreadsheet) library
- Seed from multiple spreadsheet files per Laravel seeder class
- Generate text output version of XLS spreadsheet for determining changes to XLS when branch merging
- Automatically resolve CSV filename to table name.
- Automatically resolve XLSX worksheet tabs to table names.
- Automatically map CSV and XLSX headers to table column names.
- Automatically determine delimiter for CSV files, including comma `,`, tab `\t`, pipe `|`, and semi-colon `;`
- Skip seeding data columns by using a prefix character in the spreadsheet column header.
- Hash values with a given array of column names.
- Seed default values into table columns.
- Adjust Laravel's timestamp at seeding.

### Scale
This package has been used on CSV files with 5 million rows per file while maintaining flat memory usage (no memory leaks).

### Testing
This package has PHPUnit tests.  Tests are added as enhancements are made or as bugs are found and fixed.

This package is tested against the following Laravel versions
* Laravel 5.8
* Laravel 6.x
* Laravel 7.x
* Laravel 8.x

## Contents
- [Installation](#installation)
- [Simplest Usage](#simplest-usage)
- [Basic Usage](#basic-usage)
- [Seeding Individual Sheets](#seeding-individual-sheets)
- [Markdown Diffs](#excel-text-markdown-output-for-branch-diffs)
- [Configuration Settings](#configuration)
- [Conversion Details](#details)
- [Examples](#examples)
- [License](#license)
- [Changes](#changes)

## Installation
- Require this package directly by `composer require --dev bfinlay/laravel-excel-seeder`
- Or add this package in your composer.json and run `composer update`

    ```
    "bfinlay/laravel-excel-seeder": "^2.0"
    ```
## Simplest Usage
In the simplest form, you can use the `bfinlay\SpreadsheetSeeder\SpreadsheetSeeder`
as is and it will process all XLSX files in `/database/seeds/*.xlsx` and `/database/seeders/*.xlsx` (relative to Laravel project base path).

Just add the SpreadsheetSeeder to be called in your `/database/seeds/DatabaseSeeder.php` (Laravel 5.8, 6.x, 7.x) or `/database/seeder/DatabaseSeeder.php` (Laravel 8) class.

```php
use Illuminate\Database\Seeder;
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            SpreadsheetSeeder::class,
        ]);
    }
}
```

Place your spreadsheets into the path `/database/seeds/` (Laravel 5.8, 6.x, 7.x) or `/database/seeders/` (Laravel 8.x) of your Laravel project.

With the default settings, the seeder makes certain assumptions about the XLSX files:
* worksheet (tab) names match --> table names in database
* worksheet (tab) has a header row and the column names match --> table column names in database
* If there is only one worksheet in the XLSX workbook either the worksheet (tab) name or workbook filename must match a table in the database. 


An Excel example:

| first_name    | last_name     | birthday   |
| ------------- | ------------- | ---------- |
| Foo           | Bar           | 1970-01-01 |
| John          | Doe           | 1980-01-01 |

A CSV example:
```
    first_name,last_name,birthday
    Foo,Bar,1970-01-01
    John,Doe,1980-01-01
```


## Basic usage
In most cases you will need to configure settings.
Create a seed class that extends `bfinlay\SpreadsheetSeeder\SpreadsheetSeeder` and configure settings on your class.  A seed class will look like this:
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // By default, the seeder will process all XLSX files in /database/seeds/*.xlsx (relative to Laravel project base path)
        
        // Example setting
        $this->worksheetTableMapping = ['Sheet1' => 'first_table', 'Sheet2' => 'second_table'];
        
        parent::run();
    }
}
```
## Seeding Individual Sheets
By default, executing the `db:seed` Artisan command will seed all worksheets within a workbook.

If you want to specify individual sheets to seed, you may use the `xl:seed` Artisan command
with the `--sheet` option.  You may specify multiple `--sheet` options.

```
php artisan xl:seed --sheet=users --sheet=posts 
```

The above will run the `Database\Seeders\DatabaseSeeder` class, and for any SpreadsheetSeeders that are invoked
will only seed sheets named `users` and `posts`.  You may use the `--class` option to specify a specific seeder
class to run individually

```
php artisan xl:seed --class=MySpreadsheetSeederClass --sheet=users --sheet=posts
```

If you want to run the default `SpreadsheetSeeder` class, you can specify `--class=#`.  (The `#` resembles a spreadsheet.)

```
php artisan xl:seed --class=# --sheet=users --sheet=posts
```

For an easier syntax, you can also pass these as arguments and omit the --class and --seed.   When using arguments,  
the first argument must be the class, and subsequent arguments will be sheets.

```
php artisan xl:seed # users posts
```

Important note: as with seeding traditional seeder classes individually, when seeding individual sheets if the truncate option is true,
relations with cascade delete will also be deleted.

## Excel Text Markdown Output for Branch Diffs
After running the database seeder, a subdirectory will be created using
the same name as the input file.  A text output file will be created
for each worksheet using the worksheet name with an "md"
extension.  This text file contains a markdown text representation of each
worksheet (tab) in the workbook and can be used to determine
changes in the XLSX when merging branches from other contributors.

Check this file into the repository so that it can serve as a basis for
comparison.

You will have to merge the XLSX spreadsheet manually.

The file extension can be changed by setting the `textOutputFileExtension` setting.

TextOutput can be disabled by setting `textOutput` to `FALSE`

## Configuration
* [Aliases](#column-aliases) - (global) map column names to alternative column names
* [Batch Insert Size](#batch-insert-size) - number of rows to insert per batch
* [Date Formats](#date-formats) - configure date formats by column when Carbon cannot automatically parse date
* [Defaults](#defaults) - (global) map column names to default values
* [Delimiter](#delimiter) - specify CSV delimiter (default: auto detect)
* [Extension](#data-source-file-default-extension) - default file extension when directory is specified (default: xlsx)
* [File](#data-source-file) - path or paths of data source files to process (default: /database/seeds/*.xlsx)
* [Hashable](#hashable) - (global) array of column names hashed using Hash facade
* [Header](#header) - (global) skip first row when true (default: true)
* [Input Encodings](#input-encodings) - (global) array of possible encodings from input data source
* [Limit](#limit) - (global) limit the maximum number of rows that will be loaded from a worksheet (default: no limit)
* [Mapping](#column-mapping) - column "mapping"; array of column names to use as a header
* [Offset](#offset) - (global) number of rows to skip at the start of the data source (default: 0)
* [Output Encodings](#output-encodings) - (global) output encoding to database
* [Read Chunk Size](#read-chunk-size) - number of rows to read per chunk
* [Skipper](#skipper) - (global) prefix string to indicate a column or worksheet should be skipped (default: "%")
* [Tablename](#destination-table-name) - (legacy) table name to insert into database for single-sheet file
* [Text Output](#text-output) - enable text markdown output (default: true)
* [Text Output File Extension](#text-output-file-extension) - extension for text output table
* [Timestamps](#timestamps) - when true, set the Laravel timestamp columns 'created_at' and 'updated_at' with current date/time (default: true)
* [Truncate](#truncate-destination-table) - truncate the table before seeding (default: true)
* [Truncate Ignore Foreign Key Constraints](#truncate-ignore-foreign) - truncate the table before seeding (default: true)
* [Unix Timestamps](#unix-timestamps) - interpret date/time values as unix timestamps instead of excel timestamps for specified columns (default: no columns)
* [Validate](#validate) - map column names to laravel validation rules
* [Worksheet Table Mapping](#worksheet-table-mapping) - map names of worksheets to table names

### Column Aliases
`$aliases` *(array [])*

This is an associative array to map the column names of the data source
to alternative column names (aliases).

Note: this setting is currently global and applies to all files or
worksheets that are processed.  All columns with the same name in all files
or worksheets will have the same alias applied.  To apply differently to
different files, process files with separate Seeder instances.

Example: `['CSV Header 1' => 'Table Column 1', 'CSV Header 2' => 'Table Column 2']`

Default: `[]`

### Batch Insert Size
`$batchInsertSize` *(integer)*

Number of rows to insert in a batch.

Default: `5000`

### Date Formats
`$dateFormats` *(array [])*

This is an associative array mapping column names in the data source to
date format strings that should be used by Carbon to parse the date.
Information to construct date format strings is here:
[https://www.php.net/manual/en/datetime.createfromformat.php](https://www.php.net/manual/en/datetime.createfromformat.php)

When the destination column in the database table is a date time format,
and the source data is a string, the seeder will use Carbon to parse the
date format.  In many cases Carbon can parse the date automatically
without specifying the date format.

When Carbon cannot parse the date automatically, map the column name in
this array to the date format string.   When a source column is mapped,
Carbon will use the date format string instead of parsing automatically.

If column mapping is used (see [mapping](#mapping)) the column name should match the
value in the $mapping array instead of the value in the file, if any.

Note: this setting is currently global and applies to all files or
worksheets that are processed.  All columns with the specified name in all files
or worksheets will have the validation rule applied.  To apply differently to
different files, process files with separate Seeder instances.

Example:
```
[
  'order_date' => 'Y-m-d H:i:s.u+',  // parses "2020-10-04 05:31:02.440000000"
]
```

Default: `[]`

### Defaults
`$defaults` *(array [])*

This is an associative array mapping column names in the data source to
default values that will override any values in the datasource.

Note: this setting is currently global and applies to all files or
worksheets that are processed.  To apply differently to
different files, process files with separate Seeder instances.

Example: `['created_by' => 'seed', 'updated_by' => 'seed]`

Default: `[]`

### Delimiter
`$delimiter` *(string NULL)*

The delimiter used in CSV, tab-separate-files, and other text delimited
files.  When this is not set, the phpspreadsheet library will
automatically detect the text delimiter

Default: `null`

### Data Source File Default Extension
`$extension` *(string 'xlsx'*)

The default extension used when a directory is specified in $this->file

Default: `"xlsx"`

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // specify relative to Laravel project base path
        // feature directories specified
        $this->file = [
            '/database/seeds/feature1', 
            '/database/seeds/feature2',
            '/database/seeds/feature3', 
            ]; 
        
        // process all xlsx and csv files in paths specified above
        $this->extension = ['xlsx', 'csv'];
        
        parent::run();
    }
}
```

### Data Source File

`$file` *(string*) or *(array []*) or *(Symfony\Component\Finder\Finder*)

This value is the path of the Excel or CSV file used as the data
source. This is a string or array[] and is list of files or directories
to process, which can include wildcards.  It can also be set to an instance
of [Symfony Finder](https://symfony.com/doc/current/components/finder.html),
which is a component that is already included with Laravel.

By default, the seeder will process all XLSX files in /database/seeds (for Laravel 5.8 - 7.x)
and /database/seeders (for Laravel 8.x).

The path is specified relative to the root of the project

Default: `"/database/seeds/*.xlsx"`

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // specify relative to Laravel project base path
        $this->file = [
            '/database/seeds/file1.xlsx', 
            '/database/seeds/file2.xlsx',
            '/database/seeds/seed*.xlsx', 
            '/database/seeds/*.csv']; 
        
        parent::run();
    }
}
```

This setting can also be configured to an instance of
[Symfony Finder](https://symfony.com/doc/current/components/finder.html),
which is a component that is already included with Laravel.

When using Finder, the path is not relative to `base_path()` by default.
To make the path relative to `base_path()` prepend it to the finder path.
You could also use one of the other [Laravel path helpers](https://laravel.com/docs/8.x/helpers#method-base-path) .

Example:
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // specify relative to Laravel project base path
        $this->file =
            (new Finder)
                ->in(base_path() . '/database/seeds/')
                ->name('*.xlsx')
                ->notName('*customers*')
                ->sortByName();
        
        parent::run();
    }
}
```

### Hashable
`$hashable` *(array ['password'])*

This is an array of column names in the data source that should be hashed
using Laravel's `Hash` facade.

The hashing algorithm is configured in `config/hashing.php` per
[https://laravel.com/docs/master/hashing](https://laravel.com/docs/master/hashing)

Note: this setting is currently global and applies to all files or
worksheets that are processed.  All columns with the specified name in all files
or worksheets will have hashing applied.  To apply differently to
different files, process files with separate Seeder instances.

Example: `['password']`

Default: `[]`


### Header
`$header` *(boolean TRUE)*

If the data source has headers in the first row, setting this to true will
skip the first row.

Default: `TRUE`

### Input Encodings
`$inputEncodings` *(array [])*

Array of possible input encodings from input data source
See [https://www.php.net/manual/en/mbstring.supported-encodings.php](https://www.php.net/manual/en/mbstring.supported-encodings.php)

This value is used as the "from_encoding" parameter to mb_convert_encoding.
If this is not specified, the internal encoding is used.

Default: `[]`

### Limit
`$limit` *(int*)

Limit the maximum number of rows that will be loaded from a worksheet.
This is useful in development to keep loading time fast.

This can be used in conjunction with settings in the environment file or App::environment() (APP_ENV) to limit data rows in the development environment.

Example:
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class SalesTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/sales.xlsx';
        if (App::environment('local'))
            $this->limit = 10000;
        
        parent::run();
    }
}
```

Default: null


### Column "Mapping"
`$mapping` *(array [])*

Backward compatibility to laravel-csv-seeder

This is an array of column names that will be used as headers.

If $this->header is true then the first row of data will be skipped.
This allows existing headers in a CSV file to be overridden.

This is called "Mapping" because its intended use is to map the fields of
a CSV file without a header line to the columns of a database table.

Note: this setting is currently global and applies to all files or
worksheets that are processed.  To apply differently to different files,
process files with separate Seeder instances.

Example: `['Header Column 1', 'Header Column 2']`

Default: `[]`

### Offset
`$offset` *(integer)*

Number of rows to skip at the start of the data source, excluding the
header row.

Default: `0`

### Output Encodings
`$outputEncoding` *(string)*

Output encoding to database
See [https://www.php.net/manual/en/mbstring.supported-encodings.php](https://www.php.net/manual/en/mbstring.supported-encodings.php)

This value is used as the "to_encoding" parameter to mb_convert_encoding.

Default: `UTF-8`

### Read Chunk Size
`$readChunkSize` *(integer)*

Number of rows to read per chunk.

Default: `5000`

### Skipper
`$skipper` *(string %)*

This is a string used as a prefix to indicate that a column in the data source
should be skipped.  For Excel workbooks, a worksheet prefixed with
this string will also be skipped.  The skipper prefix can be a
multi-character string.

- Example: Data source column `%id_copy` will be skipped with skipper set as `%`
- Example: Data source column `#id_copy` will be skipped with skipper set as `#`
- Example: Data source column `[skip]id_copy` will be skipped with skipper set as `[skip]`
- Example: Worksheet `%worksheet1` will be skipped with skipper set as `%`

Default: `"%"`;

### Destination Table Name
`$tablename` *(string*)

Backward compatibility to laravel-csv-seeder

Table name to insert into in the database.  If this is not set then
the tablename is automatically resolved by the following rules:
- if there is only 1 worksheet in a file and the worksheet is not the name of a table, use the base filename
- otherwise use worksheet name

Use worksheetTableMapping instead to map worksheet names to alternative
table names

Default: `null`

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // specify relative to Laravel project base path
        // specify filename that is automatically dumped from an external process
        $this->file = '/database/seeds/autodump01234456789.xlsx';  // note: could alternatively be a csv
        
        // specify the table this is loaded into
        $this->tablename = 'sales';
        
        // in this example, table truncation also needs to be disabled so previous sales records are not deleted
        $this->truncate = false;
        
        parent::run();
    }
}
```

### Text Output
`$textOutput` *(boolean)*

Set to false to disable output of textual markdown tables.

Default: `TRUE`

### Text Output File Extension
`$textOutputFileExtension` *(string)*

Extension for textual markdown tables.

Default: `md`

### Timestamps
`$timestamps` *(string/boolean TRUE)*

When `true`, set the Laravel timestamp columns 'created_at' and 'updated_at'
with the current date/time.

When `false`, the fields will be set to NULL

Default: `true`

### Truncate Destination Table
`$truncate` *(boolean TRUE)*

Truncate the table before seeding.

Default: `TRUE`

Note: does not currently support array of table names to exclude

See example for [tablename](#destination-table-name) above

### Truncate Destination Table Ignoring Foreign Key Constraints
`$truncateIgnoreForeign` *(boolean TRUE)*

Ignore foreign key constraints when truncating the table before seeding.

When `false`, table will not be truncated if it violates foreign key integrity.

Default: `TRUE`

Note: does not currently support array of table names to exclude


### Unix Timestamps
`$unixTimestamps` *(array [])*

This is an array of column names that contain values that should be
interpreted unix timestamps rather than excel timestamps.
See [Conversions: Date/Time values](#datetime-values)

If column mapping is used (see mapping) the column name should match the
value in the $mapping array instead of the value in the file, if any.

Note: this setting is currently global and applies to all files or
worksheets that are processed.  All columns with the specified name in all files
or worksheets will be interpreted as unix timestamps.  To apply differently to
different files, process files with separate Seeder instances.

Example: `['start_date', 'finish_date']`;

Default: `[]`


### Validate
`$validate` *(array [])*

This is an associative array mapping column names in the data source that
should be validated to a Laravel Validator validation rule.
The available validation rules are described here:
[https://laravel.com/docs/master/validation#available-validation-rules](https://laravel.com/docs/master/validation#available-validation-rules)

Note: this setting is currently global and applies to all files or
worksheets that are processed.  All columns with the specified name in all files
or worksheets will have the validation rule applied.  To apply differently to
different files, process files with separate Seeder instances.

Example:
```
[
  'email' => 'unique:users,email_address',
  'start_date' => 'required|date|after:tomorrow',
  'finish_date' => 'required|date|after:start_date'
]
```

Default: `[]`

### Worksheet Table Mapping
`$worksheetTableMapping` *(array [])*

This is an associative array to map names of worksheets in an Excel file
to table names.

Excel worksheets have a 31 character limit.

This is useful when the table name should be longer than the worksheet
character limit.

Example: `['Sheet1' => 'first_table', 'Sheet2' => 'second_table']`

Default: `[]`

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // specify the table this is loaded into
        $this->worksheetTableMapping = [
            'first_table_name_abbreviated' => 'really_rather_very_super_long_first_table_name', 
            'second_table_name_abbreviated' => 'really_rather_very_super_long_second_table_name'
            ];
        
        parent::run();
    }
}
```

## Details
#### Null values
- String conversions: 'null' is converted to `NULL`, 'true' is converted to `TRUE`, 'false' is converted to `FALSE`
- 'null' strings converted to `NULL` are treated as explicit nulls.  They are not subject to implicit conversions to default values. 
- Empty cells are set to the default value specified in the database table data definition, unless the entire row is empty
- If the entire row consists of empty cells, the row is skipped.  To intentionally insert a null row, put the string value 'null' in each cell

#### Date/Time values
When the destination table column is a date/time type, the cell value is converted to a Date/Time format.
- If the value is numeric, it is assumed to be an excel date value
- If the value is a string, it is parsed using [Carbon::parse](https://carbon.nesbot.com/docs/#api-instantiation) and formatted for the SQL query.
- If the value is a unix timestamp, specify the column name with the [Unix Timestamps](#unix-timestamps) setting to convert it as a unix timestamp instead of an excel timestamp.

## Examples
#### Table with specified timestamps and specified table name
Use a specific timestamp for 'created_at' and 'updated_at' and also
give the seeder a specific table name instead of using the CSV filename;

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/csvs/users.csv';
        $this->tablename = 'email_users';
        $this->timestamps = '1970-01-01 00:00:00';
        
        parent::run();
    }
}
```

#### Worksheet to Table Mapping
Map the worksheet tab names to table names.

Excel worksheet tabs have a 31 character limit.  This is useful when the table name should be longer than the worksheet tab character limit.

See [example](#worksheet-table-mapping) above

#### Mapping
Map the worksheet or CSV headers to table columns, with the following CSV;

##### XLSX
|    |               |               |
|----| ------------- | ------------- |
| 1  | Foo           | Bar           |
| 2  | John          | Doe           |

##### CSV
    1,Foo,Bar
    2,John,Doe

Example:
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/users.xlsx';
        $this->mapping = ['id', 'firstname', 'lastname'];
        $this->header = FALSE;
        
        parent::run();
    }
}
```

Note: this mapping is a legacy laravel-csv-seeder option.   The mapping currently applies to all
worksheets within a workbook, and is currently designed for single sheet workbooks
and CSV files.

There are two workarounds for mapping different column headers for different input files or worksheets:
1. add header columns to your multi-sheet workbooks
2. use CSVs or single-sheet workbooks and create a separate seeder for each that need different column mappings

#### Aliases with defaults
Seed a table with aliases and default values, like this;

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/csvs/users.csv';
        $this->aliases = ['csvColumnName' => 'table_column_name', 'foo' => 'bar'];
        $this->defaults = ['created_by' => 'seeder', 'updated_by' => 'seeder'];
        
        parent::run();
    }
}
```

#### Skipper
Skip a worksheet in a workbook, or a column in an XLSX or CSV with a prefix. For example you use `id` in your worksheet which is only usable in your workbook. The worksheet file might look like the following:

| %id | first_name    | last_name     | %id_copy | birthday   |
|-----| ------------- | ------------- | -------- | ---------- |
| 1   | Foo           | Bar           | 1        | 1970-01-01 |
| 2   | John          | Doe           | 2        | 1980-01-01 |

The first and fourth value of each row will be skipped with seeding. The default prefix is '%' and changeable.  In this example the skip prefix is changed to 'skip:'

| skip:id | first_name    | last_name     | skip:id_copy | birthday   |
|---------| ------------- | ------------- | ------------ | ---------- |
| 1       | Foo           | Bar           | 1            | 1970-01-01 |
| 2       | John          | Doe           | 2            | 1980-01-01 |

```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/users.xlsx';
        $this->skipper = 'skip:';
        
        parent::run();
    }
}
```

To skip a worksheet in a workbook, prefix the worksheet name with '%' or the specified skipper prefix.

#### Validate
Validate each row of an XLSX or CSV like this;
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/users.xlsx';
        $this->validate = [ 'name'              => 'required',
                            'email'             => 'email',
                            'email_verified_at' => 'date_format:Y-m-d H:i:s',
                            'password'          => ['required', Rule::notIn([' '])]];
        
        parent::run();
    }
}
```

#### Hash
Hash values when seeding an XLSX or CSV like this;
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/users.xlsx';
        $this->hashable = ['password'];
        
        parent::run();
    }
}
```

#### Input and Output Encodings
The mb_convert_encodings function is used to convert encodings.
* $this->inputEncodings is an array of possible input encodings.  Default is `[]` which defaults to internal encoding.  See [https://www.php.net/manual/en/mbstring.supported-encodings.php]
* $this->outputEncoding is the output encoding.  Default is 'UTF-8';
```php
use bfinlay\SpreadsheetSeeder\SpreadsheetSeeder;

class UsersTableSeeder extends SpreadsheetSeeder
{    
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->file = '/database/seeds/users.xlsx';
        $this->inputEncodings = ['UTF-8', 'ISO-8859-1'];
        $this->outputEncoding = 'UTF-8';
        
        parent::run();
    }
}
```

#### Retrieving the list of tables seeded
##### Deprecated
Postgres Sequence counters are now automatically updated when using Postgres.  `tablesSeeded` is still available.

##### Previous Documentation

The list of tables that were seeded can be retrieved by reading $this->tablesSeeded, 
which is an array of strings containing the names of the tables that were seeded.

This can be used after seeding to further process tables - for example to reset id sequence numbers in postgres
```php
    public function updatePostgresSeqCounters() {
        $tables = $this->tablesSeeded;
        foreach($tables as $table) {
            if (DB::connection()->getSchemaBuilder()->hasColumn($table, 'id')) {
                $return = DB::select("select setval('{$table}_id_seq', max(id)) from {$table}");
            }
        }
    }
```

## License
Laravel Excel Seeder is open-sourced software licensed under the MIT license.

## Changes
#### 2.3.0
- refactor code base to decouple readers and writers and eliminate mediator
- add ability to set $this->file to an instance of Symfony Finder
- automatically update Postgres sequence numbers when using Postgres
- run tests on 5.8, 6.x, 7.x, 8.x, update composer.json, and document
#### 2.2.0
- added `xl:seed` command to specify individual sheets as suggested in issue #8
#### 2.1.15
- update truncate table to disable foreign key integrity constraints issue #8
#### 2.1.14
- fix for change to mb_convert_encoding in PHP 8 issue #7 (contributed by @mw7147)
#### 2.1.13
- fix bug in text output tables: deleted worksheets were not deleted from text output tables
- refactor text output tables
#### 2.1.12
- enhance progress messages to show progress for each chunk: number of rows processed, memory usage, and processing time
- fix memory leaks in laravel-excel-seeder
- fix memory leaks in laravel framework
- add configuration setting to specify date formats for columns that Carbon cannot automatically parse
- add unit tests for date parsing
#### 2.1.11
- improved date support
#### 2.1.10
- Add `limit` feature
- Organize documentation
- Add `limit` test
#### 2.1.9
- Fix bug: v2.1.8 table name is not determine properly when worksheet name is mapped
- Markdown output: save formulas and comments outside (to the right) of region defined by header columns
- Testing
  - Add test for table name determination when worksheet names are mapped
  - Refactor test namespaces to correspond to test names
  - Move test-specific example data to laravel-excel-seeder-test-data
  - Lock laravel-excel-seeder-test-data to specific version so that test data remains in-sync with package
#### 2.1.8
- Fixed "hashable" setting so that it works per documentation.  Added "hashable" test.
- Added batchInsertSize setting to control batch size of insertions.  Default 5000 rows.
  - This will address `SQLSTATE[HY000]: General error: 7 number of parameters must be between 0 and 65535`
- Added chunked reading to read spreadsheet in chunks to conserve memory.  Default 5000 rows.
  - This will address out of memory errors when reading large worksheets
#### 2.1.7
- Added initial unit test cases for testing this package
- Refined auto-resolving of table name:
  - if there is only 1 worksheet in a file and the worksheet is not the name of a table, use the base filename
  - otherwise use worksheet name
- Implemented fix for issue in DBAL library that occurs when columns have upper case characters
#### 2.1.6
- Fix tablename setting not being used #5 (contributed by @MeowKim)
- Add setting to disable text output (default enabled) (contributed by @MeowKim)
#### 2.1.5
- Change method for text output markdown files.   Create a directory with a separate markdown per sheet instead of one long file.
#### 2.1.4
- Fix bug where worksheet prefixed with skipper string was not skipped if it was the first worksheet in the workbook
#### 2.1.3
- Parameterize text table output to achieve different text table presentations
- Fix markdown issue where some tables with empty columns would not be rendered unless the outside column '|' symbols were present
#### 2.1.2
- Update text table output to output as markdown file
#### 2.1.1
- Fix bug with calling service container that prevented settings from being properly used
#### 2.1.0
- Refactor code for better separation of concerns and decrease coupling between classes
- Add feature to output textual representation of input source spreadsheets for diff
#### 2.0.6
- add input encodings and output encodings parameters
#### 2.0.5
- add tablesSeeded property to track which tables were seeded
#### 2.0.4
- add worksheet to table mapping for mapping worksheet tab names to different table names
- add example Excel spreadsheet '/database/seeds/xlsx/classicmodels.xlsx'
#### 2.0.3
- set default 'skipper' prefix to '%'
- recognize 'skipper' prefix strings greater than 1 character in length 
#### 2.0.2
- skip rows that are entirely empty cells
- skip worksheet tabs that are prefixed with the skipper character.  This allows for additional sheets to be used for documentation, alternative designs, or intermediate calculations.
- issue #2 - workaround to skip reading and calculating cells that are part of a skipped column.   Common use case is using `=index(X:X,match(Y,Z:Z,0))` in a skipped column to verify foreign keys.
