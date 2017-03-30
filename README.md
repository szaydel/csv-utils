# csv-utils
Split (rows and columns), sort, search, edit, pivot, and obfuscate CSV files.

*New*
- catcsv: concatenate multiple CSV files 
- uniqcsv: remove duplicate rows

There is also a recursion utility which given a CSV file as
a "database" and a starting value will output a hierarchy.
See below for details and limitations.

## Splitcsv
Use the -help argument to show:
```
$ go run splitcsv.go -help
Help Message

Usage: splitcsv [options] input.csv output.csv
  -c string
    	Range spec for columns
  -headers
    	CSV has headers (default true)
  -help
    	Show usage message
  -i string
    	Input CSV filename; default STDIN
  -keep
    	Keep CSV headers on output (default true)
  -o string
    	Output CSV filename; default STDOUT
  -r string
    	Range spec for rows
$ cat test1.csv
A,B,C,D,E,F,G,H,I
1,1,1,1,1,1,1,1,1
2,2,2,2,2,2,2,2,2
3,3,3,3,3,3,3,3,3
4,4,4,4,4,4,4,4,4
5,5,5,5,5,5,5,5,5
6,6,6,6,6,6,6,6,6
7,7,7,7,7,7,7,7,7
8,8,8,8,8,8,8,8,8
9,9,9,9,9,9,9,9,9
$ go run splitcsv.go -c 4-6 -r 4-6 < test1.csv
D,E,F
4,4,4
5,5,5
6,6,6
$
```

## Sortcsv
Use the -help argument to show:

```
$ go run sortcsv.go -help
  -a	Sort ascending (default true) (default true)
  -c int
    	Column to sort (default 0)
  -headers
    	CSV has headers (default true)
  -help
    	Show help message
  -i string
    	CSV file name to sort; default STDIN
  -m string
    	Map to use instead of column values
  -o string
    	CSV output file name; default STDOUT
Map parameter is a filename of a JSON string
The JSON string maps column values to values to be
used for sorting. For example the following will
sort all values of NP first:

{
"NP": 0,
"*": 1
}

The asterisk is used to provide a default.
A default rule is required. If the default
mapping is also an asterisk, then the original
value is used.
```
Example:
```
$ splitcsv -c 4-6 -r 4-6 < test1.csv | sortcsv -a=false -c 2
D,E,F
6,6,6
5,5,5
4,4,4
$
```

## Searchcsv
Use the -help argument to show:

```
$ searchcsv -help
Help Message

Usage: searchcsv [options]
  -c string
    	Range spec for columns
  -headers
    	CSV has headers (default true)
  -help
    	Show help message
  -i string
    	Input CSV filename; default STDIN
  -keep
    	Keep CSV headers on output (default true)
  -o string
    	Output CSV filename; default STDOUT
  -pattern string
    	Search pattern
  -re
    	Search pattern is a regular expression
  -v	Omit rather than include matched rows
```
Examples:
```
$ cat test1.csv
A,B,C
abc,def,Army
one,two,Navy
go,abacus,Marine
Android,Ubuntu,Linux
$ searchcsv -c 1 -pattern "y$" < test1.csv
A,B,C
$ searchcsv -c 3 -pattern "y$" < test1.csv
A,B,C
$ searchcsv -c 3 -pattern "y$" -re=true < test1.csv
A,B,C
abc,def,Army
one,two,Navy
$ searchcsv -c 3 -pattern "[mu][xy]$" -re=true < test1.csv
A,B,C
abc,def,Army
Android,Ubuntu,Linux
$ searchcsv -v -c 3 -pattern "[mu][xy]$" -re=true < test1.csv
A,B,C
one,two,Navy
go,abacus,Marine
```
## Editcsv
Use -help to show:
```
$ editcsv -help
Help Message

Usage: editcsv [options] input.csv output.csv
  -add
    	Add replace string as a new column; default, replace in-place
  -addHeader string
    	Header to use for added column (default "ADDED")
  -c string
    	Range spec for columns
  -headers
    	CSV has headers (default true)
  -help
    	Show help message
  -i string
    	Input CSV filename; default STDIN
  -keep
    	Keep CSV headers on output (default true)
  -o string
    	Output CSV filename; default STDOUT
  -pattern string
    	Search pattern
  -replace string
    	Regexp replace expression
```
Examples:
```
$ cat test1.csv
A,B,C
abc,def,Army
one,two,Navy
go,abacus,Marine
Android,Ubuntu,Linux
$ go run editcsv.go -pattern "^(a)" -replace "x-$1" < test1.csv
A,B,C
x-bc,def,Army
one,two,Navy
go,x-bacus,Marine
Android,Ubuntu,Linux
$ go run editcsv.go -pattern "^.*y$" -replace "--elided--" < test1.csv
A,B,C
abc,def,--elided--
one,two,--elided--
go,abacus,Marine
Android,Ubuntu,Linux

$ editcsv -pattern "^.*y$" -replace "--elided--" -c 3 \
> -add=true -addHeader "final" < test1.csv
A,B,C,final
abc,def,Army,--elided--
one,two,Navy,--elided--
go,abacus,Marine,Marine
Android,Ubuntu,Linux,Linux
```

## Pivotcsv
Use -help to show:
```
$ pivotcsv -help
  -c int
    	Column to pivot (REQUIRED)
  -headers
    	CSV must have headers; cannot be false (default true)
  -help
    	Show help message
  -i string
    	CSV file name to pivot; default STDIN
  -nf string
    	Format to use for numbers (default "%v")
  -nv string
    	String to signal novalue; default is empty string
  -o string
    	CSV output file name; default STDOUT
  -on
    	Only consider numeric data and sum them (default true)
  -os
    	Consider data as strings and concatenate
  -s int
    	Column to sum/concat (REQUIRED)
  -sd string
    	Concatenation delimiter; default is comma (default ",")
```
Examples:
```
$ cat test1.csv
A,B,C,D,E,F
a1,b1,c1,d1,X,1
a2,b2,c2,d2,Y,3
a1,b1,c1,d1,X,3
a2,b2,c2,d2,Y,3
$ go run pivotcsv.go -i test1.csv -c 5 -s 6
A,B,C,D,X,Y
a1,b1,c1,d1,4,
a2,b2,c2,d2,,6

$ go run pivotcsv.go -i test1.csv -c 1 -s 2 -os
a1,a2,C,D,E,F
b1,,X,1
b1,,X,3
,"b2,b2",Y,3
$ cat test1.csv
A,B,C,D,E,F
a1,b1,c1,d1,X,1
a2,b2,c2,d2,Y,3
a1,b1,c1,d1,X,3
a2,b2,c2,d2,Y,3

$ cat test2.csv
A,B,C,D,E,F
a1,b1,c1,d1,X,1
a2,b2,c2,d2,X,3
a1,b1,c1,d1,X,3
a2,b2,c2,d2,X,3
a1,b1,c1,d1,Y,2
a2,b2,c2,d2,Y,4
a1,b1,c1,d1,Y,4
a2,b2,c2,d2,Y,4
a1,b1,c1,d1,Z,3
a2,b2,c2,d2,Z,5
a1,b1,c1,d1,Z,5
a2,b2,c2,d2,Z,5
$ go run pivotcsv.go -c 5 -s 6 < test2.csv
A,B,C,D,X,Y,Z
a1,b1,c1,d1,4,6,8
a2,b2,c2,d2,6,8,10
$

```

## Reordercsv
Use -help to show:
```
$ reordercsv -help
Help Message

  -c string
    	Order of columns from input
  -headers
    	CSV has headers (default true)
  -help
    	Show usage message
  -i string
    	Input CSV filename; default STDIN
  -keep
    	Keep CSV headers on output (default true)
  -o string
    	Output CSV filename; default STDOUT
$
```
Example:
```
$ cat test1.csv
A,B,C,D,E,F,G,H,I
1,1,1,1,1,1,1,1,1
2,2,2,2,2,2,2,2,2
3,3,3,3,3,3,3,3,3
4,4,4,4,4,4,4,4,4
5,5,5,5,5,5,5,5,5
6,6,6,6,6,6,6,6,6
7,7,7,7,7,7,7,7,7
8,8,8,8,8,8,8,8,8
9,9,9,9,9,9,9,9,9
$ reordercsv -i test1.csv -c 3,2,1,1
C,B,A,A
1,1,1,1
2,2,2,2
3,3,3,3
4,4,4,4
5,5,5,5
6,6,6,6
7,7,7,7
8,8,8,8
9,9,9,9
$
```

## Obfuscatecsv
*Notes*
1. If mulitple columns have the same data, then they must be obfuscated at the same to preserve identity of same value in two columns
2. The "prefix" is required and recommended to be something that is related to the data. For example, if names are being obfuscated, then use "name" as the prefix.
3. The sequences are simply the row and column of the first occurence of the value. That gives you a way to work backward if you need to.
4. The default delimiter between the row and column sequence number is a dash. If no delimiter is desired just use "" as shown below.

Use -help to show:
```
$ obfuscatecsv -help
Help Message

Usage: obfuscatecsv [options]
  -c string
    	Range spec for columns to obfuscate
  -d string
    	Delimiter for sequences (default "-")
  -headers
    	CSV has headers (default true)
  -help
    	Show help message
  -i string
    	Input CSV filename; default STDIN
  -keep
    	Keep CSV headers on output (default true)
  -o string
    	Output CSV filename; default STDOUT
  -prefix string
    	Prefix for obfuscator value
$
```
Example:
```
$ cat test1.csv
A,B,C
abc,def,Army
def,abc,Navy
ijk,abc,Navy
zyz,def,Army
abc,abc,AF
$ obfuscatecsv -i test1.csv -prefix XT -c 1,2
A,B,C
XT2-0,XT2-1,Army
XT2-1,XT2-0,Navy
XT4-0,XT2-0,Navy
XT5-0,XT2-1,Army
XT2-0,XT2-0,AF
$ obfuscatecsv -i test1.csv -prefix XT -c 1,2 -d "" | obfuscatecsv -prefix DOD -c 3
A,B,C
XT20,XT21,DOD2-2
XT21,XT20,DOD3-2
XT40,XT20,DOD3-2
XT50,XT21,DOD2-2
XT20,XT20,DOD6-2
$
```

## Recursecsv
*Notes*
1. It will always output the normal hierarchical columns in this order: level, root, parent, child, path, and cycle (a Yes/No)
2. Note defaults shown in the help message below
3. At present it can only take two columns of data, the parent and child columns. If these have other associated values, they will have to be added back in to this output.
4. The input must have column headers, since they are re-used in the output CSV.

*To Do*
- [done] Add support for the equivalent to Oracle's connect_by_root
- I plan to address the limitation in note 3 above by allowing the attributes associated to the parent and child to be specified. However, this is not a perfect fix, since it is possible for a child to have different values depending on the parent. For example, think of a "bolt" child. The number of bolts used in a parent assembly will vary depending on where it is used. Thus quantity is context dependent and would be lost in the planned fix. At present, I am not sure how to address this problem satisfactorily given the method used in this simple program.
- [done] Make headers optional and just defaults for output headers; probably "parent" and "child"
- [done] Add support for a list of start values from a text file. This way, once the data is loaded into memory, all hierarchies can be done in one session.

Use -help to show:
```
$ recursecsv -help
Help Message

  -child int
    	Child column; default 2 (default 2)
  -delimiter string
    	String for path delimiter (default ">")
  -help
    	Show usage message
  -i string
    	Input CSV filename; default STDIN
  -o string
    	Output CSV filename; default STDOUT
  -parent int
    	Parent column; default 1 (default 1)
  -start string
    	Start value of hierarchy
```

Example:
```
$ cat test1.csv
parent,child
A,X
A,B
B,C
D,E
C,D
X,Y
Y,B
E,C
$ recursecsv -i test1.csv -start A
Level,parent,child,Path,Cycle
1,A,B,>A>B,No
2,B,C,>A>B>C,No
3,C,D,>A>B>C>D,No
4,D,E,>A>B>C>D>E,No
5,E,C,>A>B>C>D>E>C,Yes
1,A,X,>A>X,No
2,X,Y,>A>X>Y,No
3,Y,B,>A>X>Y>B,No
4,B,C,>A>X>Y>B>C,No
5,C,D,>A>X>Y>B>C>D,No
6,D,E,>A>X>Y>B>C>D>E,No
7,E,C,>A>X>Y>B>C>D>E>C,Yes
$
```

## Catcsv
Use -help to show:
```
$ catcsv -help
Help Message

Usage: catcsv [options] input1.csv input2.csv ...
  -f    Force concatenation of different width CSV files
  -headers
        CSV has headers (default true)
  -help
        Show usage message
  -keep
        Keep CSV headers on output (default true)
  -o string
        Output CSV filename; default STDOUT
```

## Uniqcsv 
Use -help to show:
```
$ uniqcsv -help
Help Message

Usage: uniqcsv [options]
NOTE: must be sorted; only compares row against prior row.  -headers
        CSV has headers (default true)
  -help
        Show help message
  -i string
        Input CSV filename; default STDIN
  -keep
        Keep CSV headers on output (default true)
  -o string
        Output CSV filename; default STDOUT
```

For example:
```
$ cat test1.csv
A,B,C
1,2,3
1,2,3
4,5,6
d,e,f
d,e,f
d,e,f
$ uniqcsv < test1.csv
A,B,C
1,2,3
4,5,6
d,e,f
$
```