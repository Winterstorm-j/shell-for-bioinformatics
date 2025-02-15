# 5. Inspecting and Manipulating Text Data with Unix Tools - Part 1 
 
!!! abstract "Lesson Objectives"
    * Inspect file/s with utilities such as `head`,`less`. 
    * Extracting and formatting tabular data. 
    * Magical `grep`
    * use `sort`, `uniq`, `join` to manipulate the one or multiple files at once

<center>
![image](./images/inspect_1.png){width="350"}
</center>


Many formats in bioinformatics are simple tabular plain-text files delimited by a character. The most common tabular plain-text file format used in bioinformatics is tab-delimited. Bioinformatics evolved to favor tab-delimited formats because of the convenience of working with these files using Unix tools.

!!! info "Tabular Plain-Text Data Formats" 

    Tabular plain-text data formats are used extensively in computing. The basic format is
    incredibly simple: each row (also known as a record) is kept on its own line, and each
    column (also known as a field) is separated by some delimiter. There are three flavors
    you will encounter: tab-delimited, comma-separated, and variable space-delimited.

## Inspecting data with `head` and `tail`

Although `cat` command is an easy way for us to open and view the content of a file, it is not very practical to do so for a file with thousands of lines as it will exhaust the shell "space". Instead, large files should be inspected first and then manipulated accordingly. The first round of inspection can be done with `head` and `tail` command which prints the first 10 lines and the last 10 lines (`-n 10`) of a a file, respectively. Let's use `head` and `tail` to inspect *Mus_musculus.GRCm38.75_chr1.bed* 

```bash
head Mus_musculus.GRCm38.75_chr1.bed
```

??? success "Output"

    ```bash
    1	3054233	3054733
    1	3054233	3054733
    1	3054233	3054733
    1	3102016	3102125
    1	3102016	3102125
    1	3102016	3102125
    1	3205901	3671498
    1	3205901	3216344
    1	3213609	3216344
    1	3205901	3207317
    ```

```bash
tail Mus_musculus.GRCm38.75_chr1.bed 
```
??? success "Output"

    ```bash
    1	195166217	195166390
    1	195165745	195165851
    1	195165748	195165851
    1	195165745	195165747
    1	195228278	195228398
    1	195228278	195228398
    1	195228278	195228398
    1	195240910	195241007
    1	195240910	195241007
    1	195240910	195241007
    ```
Changing the number of lines printed for either of those commands can be done by passing `-n <number_of_lines>` flag .i.e. Over-ride the `-n 10` default

Try those commands with `-n 4` to print top 4 lines and bottom 4 lines

```bash
head -n 4 Mus_musculus.GRCm38.75_chr1.bed 
```
```bash
tail -n 4 Mus_musculus.GRCm38.75_chr1.bed 
```

???+ question "Exercise 4.1"

    Sometimes it’s useful to see both the beginning and end of a file — for example, if we have a sorted BED file and we want to see the positions of the first feature and last feature. Can you figure out a way to use both `head` and `tail` in a single command to inspect first and last 2 lines of ***Mus_musculus.GRCm38.75_chr1.bed***?

    ??? success "solution"
        ```bash
        (head -n 2; tail -n 2) < Mus_musculus.GRCm38.75_chr1.bed
        ```

???+ question "Exercise 4.2"

    We can also use tail to remove the header of a file. Normally the `-n` argument specifies how many of the last lines of a file to include, but if `-n` is given a number `x` preceded with a `+` sign (e.g., `+x` ), tail will start from the x<sup>th</sup> line. So to chop off a header, we start from the second line with `-n+2`.  
    Use the `seq` command to generate a file containing the numbers 1 to 10, and then use the `tail` command to chop off the first line.

    ??? success "solution"
        ```
        seq 10 > nums.txt
        ```
        ```bash
        cat nums.txt
        ```
        ```bash
        tail -n+2 nums.txt
        ```

## Extract summary information with `wc`

The "wc" in the `wc` command which stands for "word count" - this command can count the numbers of **lines, words**, and **characters** in a file (take a note on the order).

```bash
wc Mus_musculus.GRCm38.75_chr1.bed 

  81226  243678 1698545 Mus_musculus.GRCm38.75_chr1.bed
```
Often, we only need to list the number of lines, which can be done by using the `-l` flag. It can be used as a sanity check - for example, to make sure an output has the same number of lines as the input, OR to check that a certain file format which depends on another format without losing overall data structure wasn't corrupted or over/under manipulated. 

!!! question "Question"

    Count the number of lines in *Mus_musculus.GRCm38.75_chr1.bed* and *Mus_musculus.GRCm38.75_chr1.gtf* . Anything out of the ordinary ? 

Although `wc -l` is the quickest way to count the number of lines in a file, it is not the most robust as it relies on the very bad assumption that "data is well formatted" 

For an example, if we are to create a file with 3 rows of data and then two empty lines by pressing **Enter** twice, 

```bash
cat > fool_wc.bed
```
>```bash
>  1 100
>  2 200
>  3 300
>```


**Ctrl+D** to end the edits started with `cat >`

```bash
wc -l fool_wc.bed 

  5 fool_wc.bed
```
This is a good place to bring in `grep` again which can be used to count the number of lines while excluding white-spaces (spaces, tabs (`t`) or newlines (`n`))

```bash

grep -c "[^ \n\t]" fool_wc.bed 

  3
```
## Using `cut` with column data and formatting tabular data with `column`

When working with plain-text tabular data formats like tab-delimited and CSV files, we often need to extract specific columns from the original file or stream. For example, suppose we wanted to extract only the start positions (the second column) of the ***Mus_musculus.GRCm38.75_chr1.bed*** file. The simplest way to do this is with `cut`.

```bash
cut -f 2 Mus_musculus.GRCm38.75_chr1.bed | head -n 3

3054233
3054233
3054233
```
>>`-f`  argument is how we specify which columns to keep. It can be used to specify a range as well

```bash
cut -f 2-3 Mus_musculus.GRCm38.75_chr1.bed | head -n 3

 3054233	3054733
 3054233	3054733
 3054233	3054733
```


Using `cut`, we can convert our GTF for ***Mus_musculus.GRCm38.75_chr1.gtf*** to a three-column tab-delimited file of genomic ranges (e.g., chromosome, start, and end position). We’ll chop off the metadata rows using the grep command covered earlier and then use cut to extract the first, fourth, and fifth columns (chromosome, start, end):

```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f 1,4,5 | head -n 3
```
>```bash
>1	3054233	3054733
>1	3054233	3054733
>1	3054233	3054733
>```

Note that although our three-column file of genomic positions looks like a BED-formatted file, it’s not (due to subtle differences in genomic range formats).

`cut` also allows us to specify the column delimiter character. So, if we were to come across a CSV file containing chromosome names, start positions, and end positions, we could select columns from it, too:

As you may have noticed when working with tab-delimited files, it’s not always easy to see which elements belong to a particular column. For example:

```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f 1-8 | head -n 3
```
>```bash    
>  1	pseudogene	gene	3054233	3054733	.	+	.
>  1	unprocessed_pseudogene	transcript	3054233	3054733	.	+	.
>  1	unprocessed_pseudogene	exon	3054233	3054733	.	+	.
>```

While tabs are a great delimiter in plain-text data files, our variable width data causes our columns to not stack up well. There’s a fix for this in Unix: program column `-t` (the `-t` option tells column to treat data as a table). The column -t command produces neat columns that are much easier to read:

```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f 1-8 | column -t | head -n 3
```
>```bash
>   1  pseudogene                          gene         3054233    3054733    .  +  .
>   1  unprocessed_pseudogene              transcript   3054233    3054733    .  +  .
>   1  unprocessed_pseudogene              exon         3054233    3054733    .  +  .
>```

Note that you should only use `column -t` to visualize data in the terminal, not to reformat data to write to a file. Tab-delimited data is preferable to data delimited by a variable number of spaces, since it’s easier for programs to parse. Like `cut` , `column`’s default delimiter is the tab character (`\t` ). We can specify a different delimiter with the `-s` option. So, if we wanted to visualize the columns of the ***Mus_musculus.GRCm38.75_chr1_bed.csv*** file more easily, we could use:

```bash
column -s "," -t Mus_musculus.GRCm38.75_chr1_bed.csv | head -n 3

    1	3054233	3054733
    1	3054233	3054733
    1	3054233	3054733
```

??? warning "Counting the number of columns of a *.gtf* with `grep` and `wc`"

    **GTF** Format has 9 columns 

    
    |1       |2      |3       |4     |5   |6     |7      |8     |9         |
    |:-------|:------|:-------|:-----|:---|:-----|:------|:-----|:---------|
    |seqname |source |feature |start |end |score |strand |frame |attribute |
    
    In theory, we should be able to use the following command to isolate the column headers and count the number
    
    ```bash
    grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | head -1 | wc -w
    ```
    However, this will return **16** and the problem is with the **attribute** column which is a *"A semicolon-separated list of tag-value pairs, providing additional information about each feature."*

    ```
    gene_id "ENSMUSG00000090025"; gene_name "Gm16088"; gene_source "havana"; gene_biotype "pseudogene";
    ```
    Therefore, we need to *translate** these values to a single "new line" with `tr` command and then count the number of lines than than the number of words
    
    ```bash
    grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | head -1 | tr '\t' '\n' | wc -l
    ```


## Sorting Plain-Text Data with `sort`

Very often we need to work with sorted plain-text data in bioinformatics. The two most common reasons to sort data are as follows:

- Certain operations are much more efficient when performed on sorted data.
- Sorting data is a prerequisite to finding all unique lines.

`sort`  is designed to work with plain-text data with columns. Create a test .bed file with few rows and use `sort` command without any arguments.

```bash
cat > test_sort.bed
```

??? abstract  "input"

    ```bash
    chr1	26	39
    chr1	32	47
    chr3	11	28
    chr1	40	49
    chr3	16	27
    chr1	9	28
    chr2	35	54
    chr1	10	19
    ```
```bash
sort test_sort.bed 
```
??? success "Output"

    ```bash
    chr1	10	19
    chr1	26	39
    chr1	32	47
    chr1	40	49
    chr1	9	28
    chr2	35	54
    chr3	11	28
    chr3	16	27
    ```
`sort` without any arguments simply sorts a file alphanumerically by line. Because chromosome is the first column, sorting by line effectively groups chromosomes together, as these are "ties" in the sorted order.

However, using `sort`’s default of sorting alphanumerically by line and doesn’t handle tabular data properly. There are two new features we need:

- The ability to sort by particular columns

- The ability to tell sort that certain columns are numeric values (and not alpha‐numeric text)

`sort` has a simple syntax to do this. Let’s look at how we’d sort example.bed by chromosome (first column), and start position (second column):

```bash
sort -k1,1 -k2,2n test_sort.bed
``` 
??? success "Output"

    ```bash
    chr1	9	28
    chr1	10	19
    chr1	26	39
    chr1	32	47
    chr1	40	49
    chr2	35	54
    chr3	11	28
    chr3	16	27
    ```

!!! quote ""

    Here, we specify the columns (and their order) we want to sort by as `-k` arguments. In technical terms, `-k` specifies the sorting keys and their order. Each `-k` argument takes a range of columns as `start,end`, so to sort by a single column we use `start,start`. In the preceding example, we first sorted by the first column (chromosome), as the first `-k` argument was `-k1,1` . Sorting by the first column alone leads to many ties in rows
    with the same chromosomes (e.g., “chr1” and “chr3”). Adding a second `-k` argument with a different column tells sort how to break these ties. In our example, `-k2,2n` tells sort to sort by the second column (start position), treating this column as numerical data (because there’s an `n` in `-k2,2n`).

    The end result is that rows are grouped by chromosome and sorted by start position.


??? question "Exercise 4.3"

    ***Mus_musculus.GRCm38.75_chr1_random.gtf*** file is ***Mus_musculus.GRCm38.75_chr1.gtf*** with permuted rows (and without a metadata
    header). Can you group rows by chromosome, and sort by position? If yes, append the output to a separate file.

    ??? success "solution"
        ```bash
        sort -k1,1 -k4,4n Mus_musculus.GRCm38.75_chr1_random.gtf > Mus_musculus.GRCm38.75_chr1_sorted.gtf
        ```

## Finding Unique Values with `uniq`

`uniq` takes lines from a file or standard input stream and outputs all lines with consecutive duplicates removed. While this is a relatively simple functionality, you will use `uniq` very frequently in command-line data processing.

```bash
cat letters.txt 
```

??? success "Output"

    ```bash
    A 
    A
    B
    C
    B
    C
    C
    C
    ```
```bash
uniq letters.txt 
```
>```bash
>A
>B
>C
>B
>C
>```

As you can see, `uniq` does not return the unique values in letters.txt — it only removes consecutive duplicate lines (keeping one). If instead we did want to find all unique lines in a file, we would first sort all lines using `sort` so that all identical lines are grouped next to each other, and then run `uniq`.

```bash
sort letters.txt | uniq

A
B
C
```
`uniq` with `-c` shows the counts of occurrences next to the unique lines.

```bash
uniq -c letters.txt 
```
>```bash
>      2 A
>      1 B
>      1 C
>      1 B
>      3 C
>```

```bash
sort letters.txt | uniq -c
```
>```bash
>      2 A
>      2 B
>      4 C
>```

Combined with other Unix tools like `cut`, `grep` and `sort`, `uniq` can be used to summarize columns of tabular data:

```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c
```
??? success "Output"

    ```bash
    25901 CDS
    36128 exon
    2027 gene
    2290 start_codon
    2299 stop_codon
    4993 transcript
    7588 UTR
    ```
Count in order from most frequent to last

```bash
grep -v "^#" Mus_musculus.GRCm38.75_chr1.gtf | cut -f3 | sort | uniq -c | sort -rn
```
>```bash
>  36128 exon
>  25901 CDS
>   7588 UTR
>   4993 transcript
>   2299 stop_codon
>   2290 start_codon
>   2027 gene
>```

!!! abstract ""
 
    * `n` and `r` represents *numerical sort* and *reverse* order (Or descending as the default as ascending) 


## Joining two files with `join`

!!! pied-piper "Prepare"

    Create a file named *example_lengths.txt* with following content

    ```bash
    chr1	58352
    chr2	39521
    chr3	24859
    ```
The Unix tool join is used to join different files together by a common column. For example, we may want to add chromosome lengths recorded in example_lengths.txt to example.bed BED file, we saw earlier. The files look like this:

```
cat example.bed
echo "=========="
cat example_lengths.txt
```

>```bash
>chr1	26	39
>chr1	32	47
>chr3	11	28
>chr1	40	49
>chr3	16	27
>chr1	9	28
>chr2	35	54
>chr1	10	19
>==========
>chr1	58352
>chr2	39521
>chr3	24859
>```
 
To do this, we need to join both of these tabular files by their common column, the one containing the chromosome names. But first, we first need to sort both files by the column to be joined on (join would not work otherwise):

```bash
sort -k1,1 example.bed > example_sorted.bed
```
```bash
sort -c -k1,1 example_lengths.txt
```
!!! info ""
    `-c`, `--check`, `--check=diagnose-first` = check for sorted input; do not sort

The basic syntax is `join -1 <file_1_field> -2 <file_2_field> <file_1> <file_2>`. So, with *example.bed* and *example_lengths.txt* this would be:

```bash
join -1 1 -2 1 example_sorted.bed example_lengths.txt > example_with_lengths.txt
```

```bash
cat example_with_lengths.txt
```
There are many types of joins. For now, it’s important that we make sure join is working as we expect. Our expectation is that this join should not lead to fewer rows than in our example.bed file. We can verify this with `wc -l`:

```bash
wc -l example_sorted.bed example_with_lengths.txt 
```
>```bash
>  8 example_sorted.bed
>  8 example_with_lengths.txt
> 16 total
>```

However, look what happens if our second file, *example_lengths.txt* doesn’t have the lengths for **chr3**:

```bash
head -n 2 example_lengths.txt > example_lengths_alt.txt #truncate file
```
```bash
join -1 1 -2 1 example_sorted.bed example_lengths_alt.txt
```
>```bash
>chr1 10 19 58352
>chr1 26 39 58352
>chr1 32 47 58352
>chr1 40 49 58352
>chr1 9 28 58352
>chr2 35 54 39521
>```

```bash
join -1 1 -2 1 example_sorted.bed example_lengths_alt.txt | wc -l
```
>```bash
>6
>```

Because **chr3** is absent from *example_lengths_alt.txt*, our join omits rows from example_sorted.bed that do not have an entry in the first column of example_lengths_alt.txt. If we don’t want this behavior, we can use option -a to include unpairable lines—ones that do not have an entry in either file:

```bash
join -1 1 -2 1 -a 1 example_sorted.bed example_lengths_alt.txt 
```
!!! info ""
    `-a` FILENUM  - also print unpairable lines from file FILENUM, where FILENUM is 1 or 2, corresponding to FILE1 or FILE2

>```bash
>chr1 10 19 58352
>chr1 26 39 58352
>chr1 32 47 58352
>chr1 40 49 58352
>chr1 9 28 58352
>chr2 35 54 39521
>chr3 11 28
>chr3 16 27
>```

<p align="center"><b><a class="btn" href="https://genomicsaotearoa.github.io/shell-for-bioinformatics/" style="background: var(--bs-dark);font-weight:bold">Back to homepage</a></b></p>
