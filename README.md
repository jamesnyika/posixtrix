# posixtrix
Common queries I run and wish to remember

## 1. Searching for a specific string in a bunch of files 

````bash 

## Search for the string 3.3.8 recursively(r), with line number(n) and match whole word (w)
## also exclude certain directory names and file extensions 
grep -rnw --exclude-dir={assets} --exclude=*.{js,jar,png,jpeg,jpg,psd,svg,pack} '.' -e "3.3.8"

```` 
 
More examples here
````bash
## This will only search through those files which have .c or .h extensions:

grep --include=\*.{c,h} -rnw '/path/to/somewhere/' -e "pattern"

## This will exclude searching all the files ending with .o extension:

grep --exclude=*.o -rnw '/path/to/somewhere/' -e "pattern"
## For directories it's possible to exclude a particular directory(ies) through --exclude-dir parameter. For example, this will exclude the dirs dir1/, dir2/ and all of them matching *.dst/:

grep --exclude-dir={dir1,dir2,*.dst} -rnw '/path/to/somewhere/' -e "pattern"

````


