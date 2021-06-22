# OpenRefine
Common examples of [OpenRefine](https://openrefine.org/download.html) expressions I use to work with archival description and digital collections metadata.

 * [Find typos](#find-typos)
  * [Find duplicates](#find-duplicates)
  * [Create new column with pre-populated value](#create-new-column-with-pre-populated-value)
  * [Add leading zeros](#add-leading-zeros)
  * [Merge columns (with the same separator)](#merge-columns--with-the-same-separator-)
  * [Merge columns (with different separators)](#merge-columns--with-different-separators-)
  * [Split column into multiple columns](#split-column-into-multiple-columns)
  * [Create a custom facet](#create-a-custom-facet)
  * [Add slug to all files from one series](#add-slug-to-all-files-from-one-series)
  * [Clean up dates](#clean-up-dates)
    + [Split dates into two columns (e.g. start and end)](#split-dates-into-two-columns--eg-start-and-end-)
    + [When you have start and endDate columns and you want to populate the endDate column with the same value as start (if there was nothing to split!)](#when-you-have-start-and-enddate-columns-and-you-want-to-populate-the-enddate-column-with-the-same-value-as-start--if-there-was-nothing-to-split--)
    + [Remove various characters, punctuation, and circa notations](#remove-various-characters--punctuation--and-circa-notations)
    + [Converting free-text dates to ISO-8601 machine readable](#converting-free-text-dates-to-iso-8601-machine-readable)
  * [Other common transformations](#other-common-transformations)
  * [Sources](#sources)

<hr>

## Find typos
Why this is useful?
* To avoid adding additional terms to a controlled taxonomy / vocabulary list
* Sometimes controlled vocabulary columns are turned into facets or drop-down lists, and you wouldn’t want these to include typos

Column > `Facet > Text Facet`
* Sort by **count** to quickly identify typos

## Find duplicates
Why this is useful?
* You may have columns that require unique values, e.g. identifier in a file list

Column > `Customized facets > Duplicates facet`

* True = duplicates

If you want to facet the identified duplicates, then:

Column > `Facet > Text Facet`
* Can sort by name or count

## Create new column with pre-populated value
Why this is useful?
* Quickly populate a column with the same value, e.g. Level of Description

Column > `Edit column > Add column based on this column...`
* Give column a name
* Replace `value` with the term you want to fill down in quotation marks
  * “File”

Also useful for creating a URL column based on an ID column. Example:
`'https://archive.org/details/'+value`

## Add leading zeros
Why this is useful?
* Quickly add leading zeros to boxes or file numbers so that they are consistent lengths

Column > `Edit cells > Transform`

Expression:

`"000"[0,3-length(value)] + value`

* 3 is the length, so if you want more or less leading zeros, adjust accordingly

## Merge columns (with the same separator)
Why this is useful?
* Sometimes you may want to merge data from multiple columns into one, e.g. when creating a ‘Citation’ field for a digital collection.

Go to one of the columns you would like to join, then

`Edit column > Join columns`
* You can add separator between the contents of each column.
* You can overwrite combining information into the original column or create a new column for the combining contents.

## Merge columns (with different separators)
Why this is useful?
* Merge columns to create accession/box(file)
* Similar to CONCATENATE function in Excel

Go to one of the columns you would like to merge (i.e box no.), then

`Edit cells > Transform`

Expression:

`‘B1991-0013’ + ‘/’ + value + ‘(‘ + cells[‘columnName’].value + ‘)’`

^ In this example, I am appending an accession number to a values in a box and file number column so that it reads `B1991-0013/box(file)`

## Split column into multiple columns
Why this is useful?
* Sometimes you want to split a column into more useful pieces of data, i.e Surname and First Name

Column > `Edit column > Split into several columns...`
* Can split by separator or by field lengths

## Create a custom facet
* To facet multiple columns at once (e.g. subject terms spread out over multiple columns)

Select first subject column then 
`Facet > Custom Text Facet`

Expression:

`[cells.Subject1.value, cells.Subject2.value, cells.Subject3.value]`

where `Subject1` = column name

If column name has spaces, put it into brackets and remove . before cells, like so:

`[cells["Subject 1"].value, cells["Subject 2"].value]`


## Add slug to all files from one series
This assumes you have a column identifying what series each row is in
Select series column > `Facet > Text facet` 

If there is already a slug column:
* Slug column > `Edit cells > Transform`
* Expression: change `“value”` to `“series-slug”`

If there is no slug column:
* Series column > Edit column > Add column based on this column
* New column name: `qubitParentSlug`
* Expression: change `“value”` to `“your-series-slug”`


## Clean up dates
![“ISO 8601”, XKCD,](https://imgs.xkcd.com/comics/iso_8601.png )

*Image:* ["ISO 8601", XKCD](https://xkcd.com/1179/)

Why this is useful?
* Sometimes you need to turn free text field with approximate dates, into machine-readable dates to support date range searching and sorting
* See Cassie Schmitt, “Date Formats”, https://icantiemyownshoes.wordpress.com/2014/04/24/clean-up-dates-and-openrefine/ 

### Split dates into two columns (e.g. start and end)
1. Look and see what the separators are, most likely - and ,
* `Facet > Text facet`

2. Select column to split
* `Edit column > Split into several columns..`
* By separator `[,\-]`
  * Make sure **regular expression** is **checked** ✓
  * **Remove this column** is **unchecked** ☐

### When you have start and endDate columns and you want to populate the endDate column with the same value as start (if there was nothing to split!)
1. Select all blank cells in endDate column
* `Facet > Customized facets > Facet by blank > true`
2. Fill blank cells with the values from the startDate column
* `Edit cells > Transform`

Expression: `cells[‘startDate’].value`, where ‘startDate’ is the column header

### Remove various characters, punctuation, and circa notations
Isolate rows that may have these things
* Column > `Text filter > [`

then...
* `Edit cells > Transform`
* `value.replace('[ca. ','').replace(']','').replace('[','')`

...and whatever else might be in the date column

Depending on what you have, you will need to do a variety of things to the data such as replacing the month with it’s number. 

* Again, see Cassie Schmitt, “Date Formats”, https://icantiemyownshoes.wordpress.com/2014/04/24/clean-up-dates-and-openrefine/

### Converting free-text dates to ISO-8601 machine readable
* Will depend on how the dates are written, but here are basic steps:
  * Use `Facets` or `Text Filter` to isolate rows with dates that are more than just year
  * Split into 2 or 3 columns - day, month, year or month, year
  * Replace month with numeric month (e.g. Jan to 01)
```
value.replace(‘Jan. ‘, ’01’).replace(‘Feb. ‘, ’02’).replace(‘Mar. ‘, ’03’).replace(‘Apr. ‘, ’04’).replace(‘May ‘, ’05’).replace(‘Jun. ’, ’06’).replace(‘Jul. ‘, ’07’).replace(‘Aug. ‘, ’08’).replace(‘Sep. ‘, ’09’).replace(‘Sept. ‘, ’09’).replace(‘Oct. ‘, ’10’).replace(‘Nov. ‘, ’11’).replace(‘Dec. ‘, ’12’)
```
  * In the `eventStartDate` column, transform the freetext dates (4 Jan. 1972) with data from the **day**, **month** and **year** columns 
    * `Edit cells > Transform`
    *  `cells[‘year’].value + ‘-’ + cells[‘month’].value + ‘-’ + cells[‘day’].value`


## Other common transformations

*See “Common Transformations”* - https://guides.library.illinois.edu/openrefine/commontransform 

Column > `Edit cells > Common transforms`

* Delete blanks
* Remove whitespace
* Unescape HTML entities
* To titlecase
* To uppercase
* To lowercase
* To number
* To date
* To text

You can also extract your steps if you think you’ll be repeating them again on another dataset

<hr>

## Sources

[University of Illinois Library OpenRefine LibGuide](https://guides.library.illinois.edu/openrefine)

[Library Carpentry: OpenRefine](https://librarycarpentry.org/lc-open-refine/)

[Chaos → Order blog](https://icantiemyownshoes.wordpress.com/)

[Katrina Cohen-Palacios, “Wikidata and Archivists”](http://hdl.handle.net/10315/36898)

[Open Refine Documentation Manual](https://docs.openrefine.org/)

