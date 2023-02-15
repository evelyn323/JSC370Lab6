Lab 06 - Regular Expressions and Web Scraping
================

``` r
knitr::opts_chunk$set(include  = TRUE)
```

# Learning goals

-   Use a real world API to make queries and process the data.
-   Use regular expressions to parse the information.
-   Practice your GitHub skills.

# Lab description

In this lab, we will be working with the [NCBI
API](https://www.ncbi.nlm.nih.gov/home/develop/api/) to make queries and
extract information using XML and regular expressions. For this lab, we
will be using the `httr`, `xml2`, and `stringr` R packages.

This markdown document should be rendered using `github_document`
document ONLY and pushed to your *JSC370-labs* repository in
`lab06/README.md`.

## Question 1: How many sars-cov-2 papers?

Build an automatic counter of sars-cov-2 papers using PubMed. You will
need to apply XPath as we did during the lecture to extract the number
of results returned by PubMed in the following web address:

    https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2

Complete the lines of code:

``` r
pubmedurl <- "https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2"
# Downloading the website
website <- xml2::read_html(pubmedurl)

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]/span")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "192,677"

``` r
stringr::str_extract(counts, "[\\d,]+")
```

    ## [1] "192,677"

-   How many sars-cov-2 papers are there?

Answer: There are 192,677 sars-cov-2 papers

Don’t forget to commit your work!

## Question 2: Academic publications on COVID19 and Hawaii

Use the function `httr::GET()` to make the following query:

1.  Baseline URL:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

2.  Query parameters:

    -   db: pubmed
    -   term: covid19 hawaii
    -   retmax: 1000

The parameters passed to the query are documented
[here](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

``` r
ncbiurl <- "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi"

library(httr)
query_ids <- GET(
  url   = ncbiurl,
  query = list(
    db = "pubmed",
    term = "covid19 hawaii",
    retmax = 1000
  )
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

The query will return an XML object, we can turn it into a character
list to analyze the text directly with `as.character()`. Another way of
processing the data could be using lists with the function
`xml2::as_list()`. We will skip the latter for now.

Take a look at the data, and continue with the next question (don’t
forget to commit and push your results to your GitHub repo!).

``` r
ids
```

    ## {xml_document}
    ## <eSearchResult>
    ## [1] <Count>338</Count>
    ## [2] <RetMax>338</RetMax>
    ## [3] <RetStart>0</RetStart>
    ## [4] <IdList>\n  <Id>36757946</Id>\n  <Id>36743999</Id>\n  <Id>36721521</Id>\n ...
    ## [5] <TranslationSet>\n  <Translation>\n    <From>covid19</From>\n    <To>"cov ...
    ## [6] <QueryTranslation>("covid 19"[MeSH Terms] OR "covid 19"[All Fields] OR "c ...

## Question 3: Get details about the articles

The Ids are wrapped around text in the following way:
`<Id>... id number ...</Id>`. we can use a regular expression that
extract that information. Fill out the following lines of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[0-9]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
```

``` r
print(ids[1])
```

    ## [1] "36757946"

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

1.  Baseline url:
    <https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

2.  Query parameters:

    -   db: pubmed
    -   id: A character with all the ids separated by comma, e.g.,
        “1232131,546464,13131”
    -   retmax: 1000
    -   rettype: abstract

**Pro-tip**: If you want `GET()` to take some element literal, wrap it
around `I()` (as you would do in a formula in R). For example, the text
`"123,456"` is replaced with `"123%2C456"`. If you don’t want that
behavior, you would need to do the following `I("123,456")`.

``` r
paste(ids, collapse=",")
```

    ## [1] "36757946,36743999,36721521,36715070,36685780,36662525,36652247,36619744,36602072,36591237,36582496,36572233,36560974,36555339,36554713,36536782,36485021,36473671,36454775,36454600,36443734,36425114,36399438,36399335,36395004,36382733,36381259,36377347,36371370,36367993,36349436,36347663,36317584,36306800,36299999,36284135,36279491,36265162,36247032,36227851,36217024,36216013,36212220,36196777,36186620,36178690,36173983,36152041,36146513,36128780,36126272,36113145,36108254,36091468,36081413,36065487,36051401,36043851,36018589,35997977,35969382,35954726,35953887,35951232,35947596,35942481,35923385,35922730,35921109,35916603,35899278,35873301,35873076,35833192,35821668,35818366,35801366,35775002,35746577,35742804,35716737,35710669,35700997,35678989,35673364,35661139,35636901,35632529,35627656,35608504,35605700,35605371,35600422,35594688,35584085,35570918,35528753,35528751,35503921,35502304,35495071,35455823,35415617,35405638,35399704,35382257,35382073,35379351,35370509,35358349,35320122,35311911,35296505,35273936,35263081,35252592,35182433,35179422,35156056,35146208,35142835,35141345,35123100,35120186,35111349,35100194,35089919,35076582,35059075,35047039,35028589,35027873,35023402,34967845,34946336,34940870,34936687,34909439,34901299,34897806,34896104,34890263,34877361,34855156,34825050,34815815,34786570,34778744,34765988,34765987,34757265,34744296,34739906,34736373,34736125,34734403,34732841,34704071,34704070,34704069,34704068,34704067,34704066,34704065,34704064,34704063,34704061,34687321,34662333,34661133,34661132,34661131,34661130,34661129,34661128,34661127,34661126,34661125,34661124,34661123,34661122,34661121,34621978,34562997,34559481,34545941,34536350,34532685,34529634,34491990,34481278,34473201,34448649,34417121,34406840,34391908,34367726,34355196,34352507,34334985,34331541,34314211,34308400,34308322,34291832,34287651,34287159,34283939,34254888,34228774,34226774,34210370,34195618,34189029,34183789,34183411,34183191,34180390,34140009,34125658,34117878,34108898,34102878,34091576,34062806,33990619,33982008,33980567,33973241,33971389,33966879,33938253,33929934,33926498,33900192,33897904,33894385,33889849,33889848,33859192,33856881,33851191,33826985,33821261,33789080,33781762,33781585,33775167,33772441,33770003,33769536,33746047,33728687,33718878,33717793,33706209,33661861,33661727,33657176,33655229,33607081,33606666,33606656,33587873,33495523,33482708,33471778,33464637,33442699,33422679,33422626,33417334,33407957,35412644,33331197,33316097,33308888,33301024,33276110,33270782,33251328,33244071,33236896,33229999,33216726,33193454,33186704,33176077,33139866,33098971,33096099,33087192,33083826,33043445,33027604,32984015,32969950,32921878,32914097,32914093,32912595,32907823,32907673,32891785,32888905,32881116,32837709,32837535,32763956,32763350,32745072,32742897,32692706,32690354,32680824,32666058,32649272,32596689,32592394,32584245,32501143,32486844,32462545,32432219,32432218,32432217,32427288,32420720,32386898,32371624,32371551,32361738,32326959,32323016,32314954,32300051,32259247,32151778"

``` r
ncbiurl2 <- "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi"

publications <- GET(
  url   = ncbiurl2,
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse=",")),
    retmax = 1000,
    rettype = "abstract"
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```

With this in hand, we can now analyze the data. This is also a good time
for committing and pushing your work!

## Question 4: Distribution of universities, schools, and departments

Using the function `stringr::str_extract_all()` applied on
`publications_txt`, capture all the terms of the form:

1.  University of …
2.  … Institute of …

Write a regular expression that captures all such instances

``` r
library(stringr)
institution <- str_extract_all(
  publications_txt,
  "University\\s+of\\s+(?:the\\s)?(?:South\\s)?[:alpha:]+|[:alpha:]+\\s+Institute\\s+of\\s+[:alpha:]+"
  ) 
institution <- unlist(institution)
as.data.frame(table(institution))
```

    ##                                 institution Freq
    ## 1                Army Institute of Research    1
    ## 2          Australian Institute of Tropical   15
    ## 3         Beijing Institute of Pharmacology    2
    ## 4                Berlin Institute of Health    4
    ## 5               Breadfruit Institute of the    1
    ## 6                Broad Institute of Harvard    2
    ## 7                 Cancer Institute of Emory    2
    ## 8                   Cancer Institute of New    1
    ## 9                 Davis Institute of Health    8
    ## 10            Genome Institute of Singapore    1
    ## 11     Graduate Institute of Rehabilitation    3
    ## 12          Health Institute of Montpellier    1
    ## 13           Heidelberg Institute of Global    1
    ## 14               Higher Institute of Health    1
    ## 15                    i Institute of Marine    2
    ## 16             Indian Institute of Tropical    5
    ## 17             Leeds Institute of Rheumatic    2
    ## 18    Massachusetts Institute of Technology    1
    ## 19             Mayo Institute of Technology    1
    ## 20           Medanta Institute of Education    1
    ## 21  Mediterranean Institute of Oceanography    2
    ## 22                  MGM Institute of Health    1
    ## 23        Monterrey Institute of Technology    1
    ## 24            National Institute of Allergy    4
    ## 25         National Institute of Biomedical    1
    ## 26      National Institute of Biostructures    1
    ## 27      National Institute of Environmental    3
    ## 28            National Institute of General    1
    ## 29         National Institute of Infectious    1
    ## 30       National Institute of Neurological    1
    ## 31             National Institute of Public    1
    ## 32         National Institute of Technology    1
    ## 33         Nordic Institute of Chiropractic    1
    ## 34       Prophylactic Institute of Southern    2
    ## 35                Research Institute of New    4
    ## 36       Research Institute of Tuberculosis    2
    ## 37        Swiss Institute of Bioinformatics    1
    ## 38              the Institute of Biomedical    1
    ## 39                the Institute of Medicine    1
    ## 40                    University of Alabama    3
    ## 41                    University of Alberta    2
    ## 42                    University of Applied    4
    ## 43                    University of Arizona    7
    ## 44                   University of Arkansas   26
    ## 45                      University of Basel    8
    ## 46                      University of Benin    1
    ## 47                   University of Botswana    1
    ## 48                   University of Bradford    1
    ## 49                    University of Bristol    5
    ## 50                    University of British    4
    ## 51                    University of Calgary   23
    ## 52                 University of California   97
    ## 53                  University of Cambridge    1
    ## 54                   University of Campinas    3
    ## 55                    University of Chicago   16
    ## 56                    University of Chinese    1
    ## 57                 University of Cincinnati   16
    ## 58                   University of Colorado    5
    ## 59                University of Connecticut    3
    ## 60                 University of Copenhagen    8
    ## 61                    University of Córdoba    1
    ## 62                     University of Dayton    1
    ## 63                  University of Education    1
    ## 64                     University of Exeter    1
    ## 65                   University of Florence    1
    ## 66                    University of Florida    9
    ## 67                    University of Georgia    2
    ## 68                      University of Ghana    1
    ## 69                    University of Granada    2
    ## 70                     University of Guilan    1
    ## 71                      University of Haifa    1
    ## 72                      University of Hawai  367
    ## 73                    University of Hawaiʻi    1
    ## 74                     University of Hawaii  406
    ## 75                     University of Hawaiï    2
    ## 76                     University of Hawaìi    1
    ## 77                     University of Health   12
    ## 78                       University of Hong    4
    ## 79                   University of Honolulu    6
    ## 80                     University of Ibadan    1
    ## 81                   University of Illinois    7
    ## 82                University of Information    2
    ## 83                       University of Iowa    4
    ## 84                       University of Juiz    4
    ## 85                     University of Kansas    4
    ## 86                   University of Kentucky    2
    ## 87                      University of Korea    1
    ## 88                    University of KwaZulu    3
    ## 89                   University of Lausanne    1
    ## 90                      University of Leeds    2
    ## 91                     University of Leuven    1
    ## 92                 University of Louisville    1
    ## 93                      University of Maine    2
    ## 94                     University of Malaya    3
    ## 95                   University of Maryland   23
    ## 96              University of Massachusetts   24
    ## 97                    University of Medical    3
    ## 98                   University of Medicine    6
    ## 99                  University of Melbourne    2
    ## 100                     University of Miami    3
    ## 101                  University of Michigan   10
    ## 102                     University of Minas    1
    ## 103                 University of Minnesota    6
    ## 104                  University of Missouri    2
    ## 105                   University of Montana    3
    ## 106                     University of Mount    1
    ## 107                    University of Murcia    1
    ## 108                  University of Nebraska    5
    ## 109                    University of Nevada    3
    ## 110                       University of New   20
    ## 111                     University of North    9
    ## 112                      University of Ohio    1
    ## 113                   University of Ontario    1
    ## 114                      University of Oslo    6
    ## 115                    University of Oxford   11
    ## 116                   University of Palermo    1
    ## 117                     University of Paris    1
    ## 118              University of Pennsylvania  119
    ## 119                University of Pittsburgh   15
    ## 120                     University of Porto    3
    ## 121                    University of Puerto    3
    ## 122                University of Queensland    3
    ## 123                     University of Rhode    3
    ## 124                  University of Richmond    1
    ## 125                       University of Rio    1
    ## 126                 University of Rochester    4
    ## 127                    University of Rwanda    1
    ## 128                       University of Sao    2
    ## 129                       University of São    4
    ## 130                   University of Science   34
    ## 131                   University of Sergipe    1
    ## 132                 University of Singapore    1
    ## 133            University of South Carolina    3
    ## 134             University of South Florida    1
    ## 135                  University of Southern   27
    ## 136                    University of Sydney    2
    ## 137                   University of Syndney    1
    ## 138                University of Technology    5
    ## 139                     University of Texas   12
    ## 140                University of the Health  242
    ## 141           University of the Philippines    2
    ## 142              University of Thessalonica    1
    ## 143                   University of Toronto   15
    ## 144                    University of Toulon    1
    ## 145                  University of Tübingen    7
    ## 146                      University of Utah    8
    ## 147                       University of Uyo    1
    ## 148                University of Washington   12
    ## 149                      University of West    1
    ## 150                   University of Western    1
    ## 151                 University of Wisconsin   13
    ## 152                   University of Wyoming    1
    ## 153                      University of York    1

Repeat the exercise and this time focus on schools and departments in
the form of

1.  School of …
2.  Department of …

And tabulate the results

``` r
schools_and_deps <- str_extract_all(
  publications_txt,
  "School\\s+of\\s+[:alpha:]+|Department\\s+of\\s+[:alpha:]+"
  )
as.data.frame(table(schools_and_deps))
```

    ##                      schools_and_deps Freq
    ## 1            Department of\nEducation    2
    ## 2               Department of\nHealth    1
    ## 3                Department of Ageing    1
    ## 4          Department of Agricultural    1
    ## 5           Department of Agriculture    8
    ## 6               Department of Anatomy   30
    ## 7            Department of Anesthesia    3
    ## 8         Department of Anesthesilogy    1
    ## 9        Department of Anesthesiology   15
    ## 10         Department of Anthropology    3
    ## 11              Department of Applied    4
    ## 12          Department of Atmospheric    3
    ## 13           Department of Behavioral    3
    ## 14         Department of Biochemistry    2
    ## 15            Department of Bioethics    1
    ## 16           Department of Biological   12
    ## 17              Department of Biology   32
    ## 18           Department of Biomedical    4
    ## 19           Department of Biophysics    1
    ## 20          Department of Biosciences    4
    ## 21        Department of Biostatistics   22
    ## 22               Department of Botany    1
    ## 23             Department of Business   34
    ## 24           Department of Cardiology    1
    ## 25       Department of Cardiovascular    1
    ## 26                 Department of Cell    4
    ## 27            Department of Chemistry    3
    ## 28                Department of Civil   13
    ## 29             Department of Clinical   11
    ## 30             Department of Commerce    5
    ## 31        Department of Communication    3
    ## 32        Department of Communicology    2
    ## 33            Department of Community    5
    ## 34        Department of Computational    1
    ## 35             Department of Computer    1
    ## 36             Department of Critical   15
    ## 37  Department of Cyberinfrastructure    1
    ## 38              Department of Defense   20
    ## 39            Department of Dentistry    1
    ## 40          Department of Dermatology   27
    ## 41              Department of Ecology    4
    ## 42             Department of Economic    5
    ## 43            Department of Economics   18
    ## 44            Department of Education    9
    ## 45            Department of Emergency   18
    ## 46              Department of English    2
    ## 47        Department of Environmental   10
    ## 48         Department of Epidemiology   26
    ## 49         Department of Experimental    1
    ## 50               Department of Family   17
    ## 51                 Department of Fish    1
    ## 52            Department of Fisheries    1
    ## 53             Department of Forestry    6
    ## 54              Department of General    7
    ## 55              Department of Genetic    1
    ## 56            Department of Geography   10
    ## 57          Department of Geosciences    4
    ## 58            Department of Geriatric    1
    ## 59               Department of Health  248
    ## 60           Department of Hematology    3
    ## 61                 Department of Home    1
    ## 62             Department of Homeland    1
    ## 63                Department of Human    1
    ## 64           Department of Immunology    1
    ## 65           Department of Infectious   29
    ## 66          Department of Information    2
    ## 67            Department of Intensive    3
    ## 68             Department of Internal   85
    ## 69        Department of International    3
    ## 70          Department of Kinesiology    2
    ## 71                Department of Labor    3
    ## 72           Department of Laboratory    3
    ## 73          Department of Mathematics   19
    ## 74           Department of Mechanical    8
    ## 75              Department of Medical   17
    ## 76             Department of Medicine  207
    ## 77           Department of Metabolism    1
    ## 78         Department of Microbiology   15
    ## 79            Department of Molecular   10
    ## 80               Department of Native    5
    ## 81              Department of Natural   13
    ## 82           Department of Nephrology    5
    ## 83         Department of Neurological   12
    ## 84            Department of Neurology    4
    ## 85         Department of Neurosurgery    1
    ## 86              Department of Nursing    1
    ## 87            Department of Nutrition    7
    ## 88                   Department of OB    8
    ## 89           Department of Obstetrics   20
    ## 90         Department of Occupational    2
    ## 91        Department of Ophthalmology    1
    ## 92          Department of Orthopaedic    4
    ## 93           Department of Orthopedic    5
    ## 94       Department of Otolaryngology    4
    ## 95           Department of Paediatric    1
    ## 96        Department of Parliamentary    1
    ## 97            Department of Pathology   18
    ## 98            Department of Pediatric    4
    ## 99           Department of Pediatrics   45
    ## 100      Department of Pharmaceutical    1
    ## 101        Department of Pharmacology    2
    ## 102            Department of Pharmacy    2
    ## 103            Department of Physical    5
    ## 104          Department of Physiology   16
    ## 105       Department of Physiotherapy    1
    ## 106               Department of Plant    1
    ## 107          Department of Population   17
    ## 108          Department of Prevention    1
    ## 109          Department of Preventive  179
    ## 110          Department of Psychiatry   36
    ## 111          Department of Psychology   10
    ## 112              Department of Public   18
    ## 113           Department of Pulmonary    2
    ## 114        Department of Quantitative   34
    ## 115        Department of Radiotherapy    1
    ## 116      Department of Rehabilitation    7
    ## 117            Department of Research   31
    ## 118        Department of Rheumatology    7
    ## 119             Department of Science    1
    ## 120             Department of Smoking    8
    ## 121              Department of Social    8
    ## 122           Department of Sociology    9
    ## 123              Department of Sports    1
    ## 124               Department of State    3
    ## 125          Department of Statistics    6
    ## 126             Department of Surgery   41
    ## 127            Department of Taxation    2
    ## 128                 Department of the    1
    ## 129             Department of Traffic    1
    ## 130       Department of Translational    1
    ## 131      Department of Transportation    1
    ## 132            Department of Tropical   67
    ## 133                Department of Twin    4
    ## 134             Department of Urology    1
    ## 135            Department of Veterans   18
    ## 136          Department of Veterinary    7
    ## 137               Department of Virus    1
    ## 138            Department of Wildlife    5
    ## 139             Department of Zoology    1
    ## 140               School of Aerospace    7
    ## 141             School of Agriculture    1
    ## 142                 School of Applied    1
    ## 143                 School of Aquatic    1
    ## 144              School of Biological    4
    ## 145              School of Biomedical    3
    ## 146                   School of Brown    2
    ## 147                School of Business    5
    ## 148                School of Chemical    1
    ## 149                School of Clinical    1
    ## 150           School of Communication    2
    ## 151               School of Economics    1
    ## 152               School of Education    3
    ## 153              School of Electronic    1
    ## 154                  School of Energy    2
    ## 155             School of Engineering    1
    ## 156           School of Environmental    2
    ## 157            School of Epidemiology    6
    ## 158                  School of Forest    1
    ## 159                School of Forestry    4
    ## 160              School of Government    1
    ## 161                School of Hawaiian    2
    ## 162                  School of Health   14
    ## 163                 School of Hygiene    2
    ## 164              School of Immunology    1
    ## 165                     School of Law    1
    ## 166                    School of Life    3
    ## 167                School of Medicine  688
    ## 168                 School of Natural    4
    ## 169                 School of Nursing   72
    ## 170                   School of Ocean    1
    ## 171                 School of Pacific    1
    ## 172                School of Pharmacy    2
    ## 173                School of Physical    6
    ## 174           School of Physiotherapy    1
    ## 175              School of Population    2
    ## 176                  School of Public  104
    ## 177                School of Sciences    4
    ## 178                  School of Social   69
    ## 179                     School of the    1
    ## 180          School of Transportation    1
    ## 181              School of Veterinary    1

## Question 5: Form a database

We want to build a dataset which includes the title and the abstract of
the paper. The title of all records is enclosed by the HTML tag
`ArticleTitle`, and the abstract by `Abstract`.

Before applying the functions to extract text directly, it will help to
process the XML a bit. We will use the `xml2::xml_children()` function
to keep one element per id. This way, if a paper is missing the
abstract, or something else, we will be able to properly match PUBMED
IDS with their corresponding records.

``` r
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
```

Now, extract the abstract and article title for each one of the elements
of `pub_char_list`. You can either use `sapply()` as we just did, or
simply take advantage of vectorization of `stringr::str_extract`

``` r
abstracts <- str_extract(pub_char_list, "<Abstract>(\\n|.)+</Abstract>")
# abstracts <- str_remove_all(abstracts, "<Abstract>|</Abstract>") #remove only abstract tags
abstracts <- str_remove_all(abstracts, "<.*?>") #remove all html tags
abstracts <- str_replace_all(abstracts, "\\s+", " ")
length(pub_char_list)
```

    ## [1] 338

``` r
table(is.na(abstracts))
```

    ## 
    ## FALSE  TRUE 
    ##   285    53

-   How many of these don’t have an abstract?

Answer: 53 publications

Now, the title

``` r
titles <- str_extract(pub_char_list, "<Title>(\\n|.)+</Title>")
titles <- str_remove_all(titles, "<Title>|</Title>")
table(is.na(titles))
```

    ## 
    ## FALSE 
    ##   338

-   How many of these don’t have a title ?

Zero publications

Finally, put everything together into a single `data.frame` and use
`knitr::kable` to print the results

``` r
database <- data.frame(
  title = titles,
  abstract = abstracts
)
knitr::kable(database)
```

| title                                                                                                                                     | abstract                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|:------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PloS one                                                                                                                                  | Accurate COVID-19 prognosis is a critical aspect of acute and long-term clinical management. We identified discrete clusters of early stage-symptoms which may delineate groups with distinct disease severity phenotypes, including risk of developing long-term symptoms and associated inflammatory profiles. 1,273 SARS-CoV-2 positive U.S. Military Health System beneficiaries with quantitative symptom scores (FLU-PRO Plus) were included in this analysis. We employed machine-learning approaches to identify symptom clusters and compared risk of hospitalization, long-term symptoms, as well as peak CRP and IL-6 concentrations. We identified three distinct clusters of participants based on their FLU-PRO Plus symptoms: cluster 1 (“Nasal cluster”) is highly correlated with reporting runny/stuffy nose and sneezing, cluster 2 (“Sensory cluster”) is highly correlated with loss of smell or taste, and cluster 3 (“Respiratory/Systemic cluster”) is highly correlated with the respiratory (cough, trouble breathing, among others) and systemic (body aches, chills, among others) domain symptoms. Participants in the Respiratory/Systemic cluster were twice as likely as those in the Nasal cluster to have been hospitalized, and 1.5 times as likely to report that they had not returned-to-activities, which remained significant after controlling for confounding covariates (P \< 0.01). Respiratory/Systemic and Sensory clusters were more likely to have symptoms at six-months post-symptom-onset (P = 0.03). We observed higher peak CRP and IL-6 in the Respiratory/Systemic cluster (P \< 0.01). We identified early symptom profiles potentially associated with hospitalization, return-to-activities, long-term symptoms, and inflammatory profiles. These findings may assist in patient prognosis, including prediction of long COVID risk. Copyright: This is an open access article, free of all copyright, and may be freely reproduced, distributed, transmitted, modified, built upon, or otherwise used by anyone for any lawful purpose. The work is made available under the Creative Commons CC0 public domain dedication.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| The chronicle of mentoring & coaching                                                                                                     | ‘Critical’ career milestones for faculty (e.g., tenure, securing grant funding) relate to career advancement, job satisfaction, service/leadership, scholarship/research, clinical or teaching activities, professionalism, compensation, and work-life balance. However, barriers and challenges to these milestones encountered by junior faculty have been inadequately studied, particularly those affecting underrepresented minorities in science (URM-S). Additionally, little is known about how barriers and challenges to career milestones have changed during the COVID-19 pandemic for URM-S and non-URM faculty mentees in science. In this study, we conducted semi-structured interviews with 31 faculty mentees from four academic institutions (located in New Mexico, Arizona, Idaho, and Hawaii), including 22 URM-S (women or racial/ethnic). Respondents were given examples of ‘critical’ career milestones and were asked to identify and discuss barriers and challenges that they have encountered or expect to encounter while working toward achieving these milestones. We performed thematic descriptive analysis using NVivo software in an iterative, team-based process. Our preliminary analysis identified five key themes that illustrate barriers and challenges encountered: Job and career development, Discrimination and a lack of workplace diversity; Lack of interpersonal relationships and inadequate social support at the workplace; Personal and family matters; and Unique COVID-19-related issues. COVID-19 barriers and challenges were related to online curriculum creation and administration, interpersonal relationship development, inadequate training/service/conference opportunities, and disruptions in childcare and schooling. Although COVID-19 helped create new barriers and challenges for junior faculty mentees, traditional barriers and challenges for ‘critical’ career milestones continue to be reported among our respondents. URM-S respondents also identified discrimination and diversity-related barriers and challenges. Subsequent interviews will focus on 12-month and 24-month follow-ups and provide additional insight into the unique challenges and barriers to ‘critical’ career milestones that URM and non-URM faculty in science have encountered during the unique historical context of the COVID-19 pandemic.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Global health, epidemiology and genomics                                                                                                  | Whilst the coronavirus disease 2019 (COVID-19) vaccination rollout is well underway, there is a concern in Africa where less than 2% of global vaccinations have occurred. In the absence of herd immunity, health promotion remains essential. YouTube has been widely utilised as a source of medical information in previous outbreaks and pandemics. There are limited data on COVID-19 information on YouTube videos, especially in languages widely spoken in Africa. This study investigated the quality and reliability of such videos. Medical information related to COVID-19 was analysed in 11 languages (English, isiZulu, isiXhosa, Afrikaans, Nigerian Pidgin, Hausa, Twi, Arabic, Amharic, French, and Swahili). Cohen’s Kappa was used to measure inter-rater reliability. A total of 562 videos were analysed. Viewer interaction metrics and video characteristics, source, and content type were collected. Quality was evaluated using the Medical Information Content Index (MICI) scale and reliability was evaluated by the modified DISCERN tool. Kappa coefficient of agreement for all languages was p \< 0.01. Informative videos (471/562, 83.8%) accounted for the majority, whilst misleading videos (12/562, 2.13%) were minimal. Independent users (246/562, 43.8%) were the predominant source type. Transmission of information (477/562 videos, 84.9%) was most prevalent, whilst content covering screening or testing was reported in less than a third of all videos. The mean total MICI score was 5.75/5 (SD 4.25) and the mean total DISCERN score was 3.01/5 (SD 1.11). YouTube is an invaluable, easily accessible resource for information dissemination during health emergencies. Misleading videos are often a concern; however, our study found a negligible proportion. Whilst most videos were fairly reliable, the quality of videos was poor, especially noting a dearth of information covering screening or testing. Governments, academic institutions, and healthcare workers must harness the capability of digital platforms, such as YouTube to contain the spread of misinformation. Copyright © 2023 Kapil Narain et al.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Journal of the Pediatric Infectious Diseases Society                                                                                      | In a 1:1 matched test-negative design among 5-11-year-olds in the Kaiser Permanente Southern California health system (n=3984), BNT162b2 effectiveness against omicron-related emergency department or urgent care encounters was 60% \[95%CI: 47-69\] \<3 months post-dose-two and 28% \[8-43\] after ≥3 months. A booster improved protection to 77% \[53-88\]. © The Author(s) 2023. Published by Oxford University Press on behalf of The Journal of the Pediatric Infectious Diseases Society.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Hawai’i journal of health & social welfare                                                                                                | This study explored how undergraduate students at the University of Hawai’i at Manoa sought and consumed information about the virus that causes COVID-19. This study also examined student perceptions of the severity of and their susceptibility to the virus and their main concerns about it. Four hundred fifty-six students completed online surveys between October and early December of 2020 and 2021. Students reported low to moderate levels of information seeking across four domains: (1) knowledge about COVID-19 and its symptoms; (2) preventing the spread of the virus; (3) the current state of the pandemic in Hawai’i; and (4) the likely future of the pandemic in Hawai’i. Overall, websites, television, and Instagram were the top 3 channels used by students to seek information for these domains. Students reported primarily paying attention to information from government and news organizations as sources. However, students’ preferred channels and sources varied with the type of information they sought. Students also reported believing that COVID-19 is severe and that they are susceptible to being infected with it. The more time students reported seeking information, the greater their perceptions of COVID-19’s severity across all domains. Students’ primary concerns about COVID-19 centered on state regulations/policies, vaccines, tourism/travel, the economy, and pandemic/post-pandemic life. These findings can help public health practitioners in Hawai’i determine how best to reach an undergraduate student population with information related to COVID-19. ©Copyright 2023 by University Health Partners of Hawai‘i (UHP Hawai‘i).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| JAMA network open                                                                                                                         | Immunocompromised individuals are at increased risk for severe outcomes due to SARS-CoV-2 infection. Given the varying and complex nature of COVID-19 vaccination recommendations, it is important to understand COVID-19 vaccine uptake in this vulnerable population. To assess mRNA COVID-19 vaccine uptake and factors associated with uptake among immunocompromised individuals from December 14, 2020, through August 6, 2022. This cohort study was conducted with patients of Kaiser Permanente Southern California (KPSC), an integrated health care system in the US. The study included patients aged 18 years or older who were immunocompromised (individuals with an immunocompromising condition or patients who received immunosuppressive medications in the year prior to December 14, 2020) and still met criteria for being immunocompromised 1 year later. Age, sex, self-identified race and ethnicity, prior positive COVID-19 test result, immunocompromising condition, immunomodulating medication, comorbidities, health care utilization, and neighborhood median income. Outcomes were the number of doses of mRNA COVID-19 vaccine received and the factors associated with receipt of at least 4 doses, estimated by hazard ratios (HRs) and 95% Wald CIs via Cox proportional hazards regression. Statistical analyses were conducted between August 9 and 23, 2022. Overall, 42 697 immunocompromised individuals met the study eligibility criteria. Among these, 18 789 (44.0%) were aged 65 years or older; 20 061 (47.0%) were women and 22 635 (53.0%) were men. With regard to race and ethnicity, 4295 participants (10.1%) identified as Asian or Pacific Islander, 5174 (12.1%) as Black, 14 289 (33.5%) as Hispanic, and 17 902 (41.9%) as White. As of the end of the study period and after accounting for participant censoring due to death or disenrollment from the KPSC health plan, 78.0% of immunocompromised individuals had received a third dose of mRNA COVID-19 vaccine. Only 41.0% had received a fourth dose, which corresponds to a primary series and a monovalent booster dose for immunocompromised individuals. Uptake of a fifth dose was only 0.9% following the US Centers for Disease Control and Prevention (CDC) recommendation to receive a second monovalent booster (ie, fifth dose). Adults aged 65 years or older (HR, 3.95 \[95% CI, 3.70-4.22\]) were more likely to receive at least 4 doses compared with those aged 18 to 44 years or 45 to 64 years (2.52 \[2.36-2.69\]). Hispanic and non-Hispanic Black adults (HR, 0.77 \[95% CI, 0.74-0.80\] and 0.82 \[0.78-0.87\], respectively, compared with non-Hispanic White adults), individuals with prior documented SARS-CoV-2 infection (0.71 \[0.62-0.81\] compared with those without), and individuals receiving high-dose corticosteroids (0.88 \[0.81-0.95\] compared with those who were not) were less likely to receive at least 4 doses. These findings suggest that adherence to CDC mRNA monovalent COVID-19 booster dose recommendations among immunocompromised individuals was low. Given the increased risk for severe COVID-19 in this vulnerable population and the well-established additional protection afforded by booster doses, targeted and tailored efforts to ensure that immunocompromised individuals remain up to date with COVID-19 booster dose recommendations are warranted.                                                                                                                        |
| JAMA network open                                                                                                                         | Understanding the factors associated with post-COVID conditions is important for prevention. To identify characteristics associated with persistent post-COVID-19 symptoms and to describe post-COVID-19 medical encounters. This cohort study used data from the Epidemiology, Immunology, and Clinical Characteristics of Emerging Infectious Diseases With Pandemic Potential (EPICC) study implemented in the US military health system (MHS); MHS beneficiaries aged 18 years or older who tested positive for SARS-CoV-2 from February 28, 2020, through December 31, 2021, were analyzed, with 1-year follow-up. SARS-CoV-2 infection. The outcomes analyzed included survey-reported symptoms through 6 months after SARS-CoV-2 infection and International Statistical Classification of Diseases and Related Health Problems, Tenth Revision diagnosis categories reported in medical records 6 months following SARS-CoV-2 infection vs 3 months before infection. More than half of the 1832 participants in these analyses were aged 18 to 44 years (1226 \[66.9%\]; mean \[SD\] age, 40.5 \[13.7\] years), were male (1118 \[61.0%\]), were unvaccinated at the time of their infection (1413 \[77.1%\]), and had no comorbidities (1290 \[70.4%\]). A total of 728 participants (39.7%) had illness that lasted 28 days or longer (28-89 days: 364 \[19.9%\]; ≥90 days: 364 \[19.9%\]). Participants who were unvaccinated prior to infection (risk ratio \[RR\], 1.39; 95% CI, 1.04-1.85), reported moderate (RR, 1.80; 95% CI, 1.47-2.22) or severe (RR, 2.25; 95% CI, 1.80-2.81) initial illnesses, had more hospitalized days (RR per each day of hospitalization, 1.02; 95% CI, 1.00-1.03), and had a Charlson Comorbidity Index score of 5 or greater (RR, 1.55; 95% CI, 1.01-2.37) were more likely to report 28 or more days of symptoms. Among unvaccinated participants, postinfection vaccination was associated with a 41% lower risk of reporting symptoms at 6 months (RR, 0.59; 95% CI, 0.40-0.89). Participants had higher risk of pulmonary (RR, 2.00; 95% CI, 1.40-2.84), diabetes (RR, 1.46; 95% CI, 1.00-2.13), neurological (RR, 1.29; 95% CI, 1.02-1.64), and mental health-related medical encounters (RR, 1.28; 95% CI, 1.01-1.62) at 6 months after symptom onset than at baseline (before SARS-CoV-2 infection). In this cohort study, more severe acute illness, a higher Charlson Comorbidity Index score, and being unvaccinated were associated with a higher risk of reporting COVID-19 symptoms lasting 28 days or more. Participants with COVID-19 were more likely to seek medical care for diabetes, pulmonary, neurological, and mental health-related illness for at least 6 months after onset compared with their pre-COVID baseline health care use patterns. These findings may inform the risk-benefit ratio of COVID-19 vaccination policy.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Frontiers in cellular and infection microbiology                                                                                          | Native Hawaiians and Pacific Islanders (NHPIs) suffer from higher prevalence of and mortality to type 2 diabetes mellitus (T2DM) than any other major race/ethnic group in Hawaii. Health inequities in this indigenous population was further exacerbated by the SARS-CoV-2 pandemic. T2DM progression and medical complications exacerbated by COVID-19 are partially regulated by the gut microbiome. However, there is limited understanding of the role of gut bacteria in the context of inflammation-related diseases of health disparities including T2DM and obesity. To address these gaps, we used a community-based research approach from a cohort enriched with NHPI residents on the island of Oahu, Hawaii (N=138). Gut microbiome profiling was achieved via 16s rDNA metagenomic sequencing analysis from stool DNA. Gut bacterial capacity for butyrate-kinase (BUK)-mediated fiber metabolism was assessed using quantitative PCR to measure the abundance of BUK DNA and RNA relative to total bacterial load per stool sample. In our cohort, age positively correlated with hemoglobin A1c (%; R=0.39; P\<0.001) and body mass index (BMI; R=0.28; P\<0.001). The relative abundance of major gut bacterial phyla significantly varied across age groups, including Bacteroidetes (P\<0.001), Actinobacteria (P=0.007), and Proteobacteria (P=0.008). A1c was negatively correlated with the relative levels of BUK DNA copy number (R=-0.17; P=0.071) and gene expression (R=-0.33; P=0.003). Interestingly, we identified specific genera of gut bacteria potentially mediating the effects of diet on metabolic health in this cohort. Additionally, α-diversity among gut bacterial genera significantly varied across T2DM and BMI categories. Together, these results provide insight into age-related differences in gut bacteria that may influence T2DM and obesity in NHPIs. Furthermore, we observed overlapping patterns between gut bacteria and T2DM risk factors, indicating more nuanced, interdependent interactions among these factors as partial determinants of health outcomes. This study adds to the paucity of NHPI-specific data to further elucidate the biological characteristics associated with pre-existing health inequities in this racial/ethnic group that is significantly underrepresented in biomedical research. Copyright © 2022 Wells, Kunihiro, Phankitnirundorn, Peres, McCracken, Umeda, Lee, Kim, Juarez and Maunakea.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Clinical toxicology (Philadelphia, Pa.)                                                                                                   | This is the 39th Annual Report of America’s Poison Centers’ National Poison Data System (NPDS). As of 1 January, 2021, all 55 of the nation’s poison centers (PCs) uploaded case data automatically to NPDS. The upload interval was 4.87 \[4.38, 8.62\] (median \[25%, 75%\]) minutes, effectuating a near real-time national exposure and information database and surveillance system. We analyzed the case data tabulating specific indices from NPDS. The methodology was similar to that of previous years. Where changes were introduced, the differences are identified. Cases with medical outcomes of death were evaluated by a team of medical and clinical toxicologist reviewers using an ordinal scale of 1-6 to assess the Relative Contribution to Fatality (RCF) of the exposure. In 2021, 2,851,166 closed encounters were logged by NPDS: 2,080,917 human exposures, 62,189 animal exposures, 703,086 information requests, 4,920 human confirmed nonexposures, and 54 animal confirmed nonexposures. Total encounters showed a 14.0% decrease from 2020, and human exposure cases decreased by 2.22%, while health care facility (HCF) human exposure cases increased by 7.20%. All information requests decreased by 37.0%, medication identification (Drug ID) requests decreased by 20.8%, and medical information requests showed a 61.1% decrease, although these remain about 13-fold higher than before the COVID-19 pandemic. Drug Information requests showed a 146% increase, reflecting COVID-19 vaccine calls to PCs. Human exposures with less serious outcomes have decreased 1.80% per year since 2008, while those with more serious outcomes (moderate, major or death) have increased 4.56% per year since 2000.Consistent with the previous year, the top 5 substance classes most frequently involved in all human exposures were analgesics (11.2%), household cleaning substances (7.49%), cosmetics/personal care products (5.88%), antidepressants (5.61%), and sedatives/hypnotics/antipsychotics (4.73%). As a class, antidepressant exposures increased most rapidly, by 1,663 cases/year (5.30%/year) over the past 10 years for cases with more serious outcomes.The top 5 most common exposures in children age 5 years or less were cosmetics/personal care products (10.8%), household cleaning substances (10.7%), analgesics (8.16%), dietary supplements/herbals/homeopathic (7.00%), and foreign bodies/toys/miscellaneous (6.51%). Drug identification requests comprised 3.64% of all information contacts. NPDS documented 4,497 human exposures resulting in death; 3,809 (84.7%) of these were judged as related (RCF of 1-Undoubtedly responsible, 2-Probably responsible, or 3-Contributory). These data support the continued value of PC expertise and the need for specialized medical toxicology information to manage more serious exposures. Unintentional and intentional exposures continue to be a significant cause of morbidity and mortality in the US. The near real-time status of NPDS represents a national public health resource to collect and monitor US exposure cases and information contacts. The continuing mission of NPDS is to provide a nationwide infrastructure for surveillance for all types of exposures (e.g., foreign body, infectious, venomous, chemical agent, or commercial product), and the identification and tracking of significant public health events. NPDS is a model system for the near real-time surveillance of national and global public health. |
| Frontiers in immunology                                                                                                                   | Low-density granulocytes (LDGs) are a distinct subset of neutrophils whose increased abundance is associated with the severity of COVID-19. However, the long-term effects of severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) infection on LDG levels and phenotypic alteration remain unexplored. Using participants naïve to SARS-CoV-2 (NP), infected with SARS-CoV-2 with no residual symptoms (NRS), and infected with SARS-CoV-2 with chronic pulmonary symptoms (PPASC), we compared LDG levels and their phenotype by measuring the expression of markers for activation, maturation, and neutrophil extracellular trap (NET) formation using flow cytometry. The number of LDGs was elevated in PPASC compared to NP. Individuals infected with SARS-CoV-2 (NRS and PPASC) demonstrated increased CD10+ and CD16hi subset counts of LDGs compared to NP group. Further characterization of LDGs demonstrated that LDGs from COVID-19 convalescents (PPASC and NRS) displayed increased markers of NET forming ability and aggregation with platelets compared to LDGs from NP, but no differences were observed between PPASC and NRS. Our data from a small cohort study demonstrates that mature neutrophils with a heightened activation phenotype remain in circulation long after initial SARS-CoV-2 infection. Persistent elevation of markers for neutrophil activation and NET formation on LDGs, as well as an enhanced proclivity for platelet-neutrophil aggregation (PNA) formation in COVID-19 convalescent individuals may be associated with PPASC prognosis and development. Copyright © 2022 Dean, Devendra, Jiyarom, Subia, Tallquist, Nerurkar, Chang, Chow, Shikuma and Park.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Federal practitioner : for the health care professionals of the VA, DoD, and PHS                                                          | During the initial phase of the COVID-19 pandemic, facilities transformed some medical care to virtual appointments. There was a subsequent decline in chronic disease screening and management, as well as cancer screening rates. COVID-19 vaccine events offered an opportunity to provide face-to-face preventive care to veterans, and mobile vaccine events enabled us to reach rural veterans. In this quality improvement project, we partnered with state and community organizations to reach veterans at large vaccine events, as well as in rural sites and homeless housing. The program resulted in the successful provision of preventive care to 115 veterans at these events, with high follow-up for recommended medical care. In all, 404 clinical reminders were completed and 10 new veterans were enrolled for health care. Important clinical findings included an invasive colorectal cancer, positive HIV point-of-care test, diabetic retinal disease, uncontrolled hypertension, and depression. Vaccine events offer a venue for chronic disease screening, referral, and cancer screening. Copyright © 2022 Frontline Medical Communications Inc., Parsippany, NJ, USA.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| The Journal of arthroplasty                                                                                                               | The COVID-19 pandemic caused a surge of same-day discharge (SDD) for total joint arthroplasty. However, SDD may not be beneficial for all patients. Therefore, continued investigation into the safety of SDD is necessary as well as risk stratification for improved patient outcomes. This retrospective cohort study examined 31,851 elective SDD hip and knee arthroplasties from 2016 to 2020 in a large national database. Logistic regression models were used to identify patient variables and preoperative comorbidities that contribute to postoperative complication or readmission with SDD. Adjusted odds ratios (AOR) and 95% confidence intervals (CI) were calculated. SDD increased from 1.4% in 2016 to 14.6% in 2020. SDD is associated with lower odds of readmission (AOR: 0.994, CI: 0.992-0.996) and postoperative complications (AOR: 0.998, CI: 0.997-1.000). Patients who have preoperative dyspnea (AOR: 1.03, CI: 1.02-1.04, P \< .001), chronic obstructive pulmonary disease (AOR: 1.02, CI: 1.01-1.03, P = .002), and hypoalbuminemia (AOR: 1.02, CI: 1.00-1.03, P \< .001), had higher odds of postoperative complications. Patients who had preoperative dyspnea (AOR: 1.02, CI: 1.01-1.03), hypertension (AOR: 1.01, CI: 1.01-1.03, P = .003), chronic corticosteroid use (AOR: 1.02, CI: 1.01-1.03, P \< .001), bleeding disorder (AOR: 1.02; CI: 1.01-1.03, P \< .001), and hypoalbuminemia (AOR: 1.01, CI: 1.00-1.02, P = .038), had higher odds of readmission. SDD is safe with certain comorbidities. Preoperative screening for cardiopulmonary comorbidities (eg, dyspnea, hypertension, and chronic obstructive pulmonary disease), chronic corticosteroid use, bleeding disorder, and hypoalbuminemia may improve SDD outcomes. Copyright © 2022 Elsevier Inc. All rights reserved.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Proceedings of the … Annual Hawaii International Conference on System Sciences. Annual Hawaii International Conference on System Sciences | Our collaboration seeks to demonstrate shared interrogation by exploring the ethics of machine learning benchmarks from a socio-technical management perspective with insight from public health and ethnic studies. Benchmarks, such as ImageNet, are annotated open data sets for training algorithms. The COVID-19 pandemic reinforced the practical need for ethical information infrastructures to analyze digital and social media, especially related to medicine and race. Social media analysis that obscures Black teen mental health and ignores anti-Asian hate fails as information infrastructure. Despite inadequately handling non-dominant voices, machine learning benchmarks are the basis for analysis in operational systems. Turning to the management literature, we interrogate cross-cutting problems of benchmarks through the lens of coupling, or mutual interdependence between people, technologies, and environments. Uncoupling inequality from machine learning benchmarks may require conceptualizing the social dependencies that build structural barriers to inclusion.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| International journal of molecular sciences                                                                                               | Severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) is a highly contagious and pathogenic coronavirus that emerged in late 2019 and caused a pandemic of respiratory illness termed as coronavirus disease 2019 (COVID-19). Cancer patients are more susceptible to SARS-CoV-2 infection. The treatment of cancer patients infected with SARS-CoV-2 is more complicated, and the patients are at risk of poor prognosis compared to other populations. Patients infected with SARS-CoV-2 are prone to rapid development of acute respiratory distress syndrome (ARDS) of which pulmonary fibrosis (PF) is considered a sequelae. Both ARDS and PF are factors that contribute to poor prognosis in COVID-19 patients. However, the molecular mechanisms among COVID-19, ARDS and PF in COVID-19 patients with cancer are not well-understood. In this study, the common differentially expressed genes (DEGs) between COVID-19 patients with and without cancer were identified. Based on the common DEGs, a series of analyses were performed, including Gene Ontology (GO) and pathway analysis, protein-protein interaction (PPI) network construction and hub gene extraction, transcription factor (TF)-DEG regulatory network construction, TF-DEG-miRNA coregulatory network construction and drug molecule identification. The candidate drug molecules (e.g., Tamibarotene CTD 00002527) obtained by this study might be helpful for effective therapeutic targets in COVID-19 patients with cancer. In addition, the common DEGs among ARDS, PF and COVID-19 patients with and without cancer are TNFSF10 and IFITM2. These two genes may serve as potential therapeutic targets in the treatment of COVID-19 patients with cancer. Changes in the expression levels of TNFSF10 and IFITM2 in CD14+/CD16+ monocytes may affect the immune response of COVID-19 patients. Specifically, changes in the expression level of TNFSF10 in monocytes can be considered as an immune signature in COVID-19 patients with hematologic cancer. Targeting N6-methyladenosine (m6A) pathways (e.g., METTL3/SERPINA1 axis) to restrict SARS-CoV-2 reproduction has therapeutic potential for COVID-19 patients.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| International journal of environmental research and public health                                                                         | Limited information exists about social network variation and health information sharing during COVID-19, especially for Native Hawaiians (NH), Other Pacific Islanders (OPI), and Filipinos, who experienced COVID-19 inequities. Hawai’i residents aged 18-35 completed an online survey regarding social media sources of COVID-19 information and social network health information measured by how many people participants: (1) talked to and (2) listened to about health. Regression models were fit with age, gender, race/ethnicity, chronic disease status, pandemic perceptions, and health literacy as predictors of information sources (logistic) and social network size (Poisson). Respondents were 68% female; 41% NH, OPI, or Filipino; and 73% conducted a recent COVID-19 digital search for themselves or others. Respondents listened to others or discussed their own health with \~2-3 people. Respondents who talked with more people about their health were more likely to have larger networks for listening to others. In regression models, those who perceived greater risk of acquiring COVID-19 discussed their health with more people; in discussing others’ health, women and those with chronic diseases listened to a greater number. Understanding young adults’ social networks and information sources is important for health literacy and designing effective health communications, especially to reach populations experiencing health inequities.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Lancet regional health. Americas                                                                                                          | The impact of the COVID-19 vaccination campaign in the US has been hampered by a substantial geographical heterogeneity of the vaccination coverage. Several studies have proposed vaccination hesitancy as a key driver of the vaccination uptake disparities. However, the impact of other important structural determinants such as local disparities in healthcare capacity is virtually unknown. In this cross-sectional study, we conducted causal inference and geospatial analyses to assess the impact of healthcare capacity on the vaccination coverage disparity in the US. We evaluated the causal relationship between the healthcare system capacity of 2417 US counties and their COVID-19 vaccination rate. We also conducted geospatial analyses using spatial scan statistics to identify areas with low vaccination rates. We found a causal effect of the constraints in the healthcare capacity of a county and its low-vaccination uptake. Counties with higher constraints in their healthcare capacity were more probable to have COVID-19 vaccination rates ≤50, with 35% higher constraints in low-vaccinated areas (vaccination rates ≤ 50) compared to high-vaccinated areas (vaccination rates \> 50). We also found that COVID-19 vaccination in the US exhibits a distinct spatial structure with defined “vaccination coldspots”. We found that the healthcare capacity of a county is an important determinant of low vaccine uptake. Our study highlights that even in high-income nations, internal disparities in healthcare capacity play an important role in the health outcomes of the nation. Therefore, strengthening the funding and infrastructure of the healthcare system, particularly in rural underserved areas, should be intensified to help vulnerable communities. None. © 2022 The Authors.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| JMIR cancer                                                                                                                               | Telehealth visits increase patients’ access to care and are often rated as “just as good” as face-to-face visits by oncology patients. Telehealth visits have become increasingly more common in the care of patients with cancer since the advent of the COVID-19 pandemic. Asians and Pacific Islanders are two of the fastest growing racial groups in the United States, but there are few studies assessing patient satisfaction with telemedicine among these two racial groups. Our objective was to compare satisfaction with communication during telehealth visits versus face-to-face visits among oncology patients, with a specific focus on Asian patients and Native Hawaiian and other Pacific Islander (NHOPI) patients. We surveyed a racially diverse group of patients who were treated at community cancer centers in Hawaii and had recently experienced a face-to-face visit or telehealth visit. Questions for assessing satisfaction with patient-physician communication were adapted from a previously published study of cancer survivors. Variables that impact communication, including age, sex, household income, education level, and cancer type and stage, were captured. Multivariable logistic models for patient satisfaction were created, with adjustments for sociodemographic factors. Participants who attended a face-to-face visit reported higher levels of satisfaction in all communication measures than those reported by participants who underwent a telehealth encounter. The univariate analysis revealed lower levels of satisfaction during telehealth visits among Asian participants and NHOPI participants compared to those among White participants for all measures of communication (eg, when asked to what degree “\[y\]our physician listened carefully to you”). Asian patients and NHOPI patients were significantly less likely than White patients to strongly agree with the statement (P\<.004 and P\<.007, respectively). Racial differences in satisfaction with communication persisted in the multivariate analysis even after adjusting for sociodemographic factors. There were no significant racial differences in communication during face-to-face visits. Asian patients and NHOPI patients were significantly less content with patient-physician communication during telehealth visits when compared to White patients. This difference among racial groups was not seen in face-to-face visits. The observation that telehealth increases racial disparities in health care satisfaction should prompt further exploration. ©Jared D Acoba, Chelsea Yin, Michael Meno, Justin Abe, Ian Pagano, Sharon Tamashiro, Kristy Fujinaga, Christa Braun-Inglis, Jami Fukui. Originally published in JMIR Cancer (<https://cancer.jmir.org>), 09.12.2022.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Virus research                                                                                                                            | Rapid transmission and reproduction of RNA viruses prepare conducive conditions to have a high rate of mutations in their genetic sequence. The viral mutations make adapt the severe acute respiratory syndrome coronavirus 2 in the host environment and help the evolution of the virus then also caused a high mortality rate by the virus that threatens worldwide health. Mutations and adaptation help the virus to escape confrontations that are done against it. In the present study, we analyzed 6,510,947 sequences of non-structural protein 1 as one of the conserved regions of the virus to find out frequent mutations and substitute amino acids in comparison with the wild type. NSP1 mutations rate divided into continents were different. Based on this continental categorization, E87D in global vision and also in Europe notably increased. The E87D mutation has signed up to January 2022 as the first frequent mutation observed. The remarkable mutations, H110Y and R24C have the second and third frequencies, respectively. According to the important role of non-structural protein 1 on the host mRNA translation, developing drug design against the protein could be so hopeful to find more effective ways the control and then treatment of the global pandemic coronavirus disease 2019. Published by Elsevier B.V.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| PloS one                                                                                                                                  | Using genomics, bioinformatics and statistics, herein we demonstrate the effect of statewide and nationwide quarantine on the introduction of SARS-CoV-2 variants of concern (VOC) in Hawai’i. To define the origins of introduced VOC, we analyzed 260 VOC sequences from Hawai’i, and 301,646 VOC sequences worldwide, deposited in the GenBank and global initiative on sharing all influenza data (GISAID), and constructed phylogenetic trees. The trees define the most recent common ancestor as the origin. Further, the multiple sequence alignment used to generate the phylogenetic trees identified the consensus single nucleotide polymorphisms in the VOC genomes. These consensus sequences allow for VOC comparison and identification of mutations of interest in relation to viral immune evasion and host immune activation. Of note is the P71L substitution within the E protein, the protein sensed by TLR2 to produce cytokines, found in the B.1.351 VOC may diminish the efficacy of some vaccines. Based on the phylogenetic trees, the B.1.1.7, B.1.351, B.1.427, and B.1.429 VOC have been introduced in Hawai’i multiple times since December 2020 from several definable geographic regions. From the first worldwide report of VOC in GenBank and GISAID, to the first arrival of VOC in Hawai’i, averages 320 days with quarantine, and 132 days without quarantine. As such, the effect of quarantine is shown to significantly affect the time to arrival of VOC in Hawai’i. Further, the collective 2020 quarantine of 43-states in the United States demonstrates a profound impact in delaying the arrival of VOC in states that did not practice quarantine, such as Utah. Our data demonstrates that at least 76% of all definable SARS-CoV-2 VOC have entered Hawai’i from California, with the B.1.351 variant in Hawai’i originating exclusively from the United Kingdom. These data provide a foundation for policy-makers and public-health officials to apply precision public health genomics to real-world policies such as mandatory screening and quarantine. Copyright: © 2022 Maison et al. This is an open access article distributed under the terms of the Creative Commons Attribution License, which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Health security                                                                                                                           | Local health jurisdictions have struggled to protect their communities during the pandemic, and successes are few. The Kauai District Health Office (KDHO) of the Hawaii Department of Health serves a rural island community of 73,000 residents. As a state agency, KDHO works closely with the county mayor and administration. Kauai has experienced comparatively low COVID-19 case and case-fatality rates while maintaining strong community and leadership cohesion. Kauai’s response was highly rated by residents in a recent Community Assessment for Public Health Emergency Response survey. In this article, we describe examples of local response efforts in the areas of (1) policy and regulations, (2) health-directed isolation and quarantine, (3) case investigation and contact tracing, (4) testing availability, (5) vaccine rollout and availability, and (6) public information, as well as the factors that have contributed to Kauai’s successes. KDHO regularly prioritizes agencywide initiatives that cross program silos; staff have experience using the incident command system in real-world situations; the community health worker team is multicultural, multilingual, and well established; and staff are integral members of the community they serve. Preexisting partnerships were strong, including those with county agencies, healthcare partners, and nongovernmental organizations, which facilitated early and effective collaboration. Response successes include implementation of unified command, coordinated public messaging, early protective measures, effective disease control and outbreak response, attention to secondary impacts of the pandemic, free community testing, mass vaccination, and mobile vaccinations and testing. The value of local health departments engaging regularly and authentically with partners and communities cannot be overstated. It has saved lives on Kauai. Local health jurisdictions should focus on all-hazards and all-staff endeavors to enhance their disaster response effectiveness.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| BMC public health                                                                                                                         | Racial disparities in psychological distress associated with COVID-19 remain unclear in the U.S. This study aims to investigate the associations between social determinants of health and COVID-19-related psychological distress across different racial/ethnic groups in the US (i.e., non-Hispanic Whites, Hispanic, non-Hispanic Asians, and non-Hispanic African Americans). This study used cross-sectional data from the 2020 California Health Interview Survey Adult Data Files (N = 21,280). Adjusting for covariates-including age, gender, COVID-19 pandemic challenges, and risk of severe illness from COVID-19-four sets of weighted binary logistic regressions were conducted. The rates of moderate/severe psychological distress significantly varied across four racial/ethnic groups (p \< 0.001), with the highest rate found in the Hispanic group. Across the five domains of social determinants of health, we found that unemployment, food insecurity, housing instability, high educational attainment, usual source of health care, delayed medical care, and low neighborhood social cohesion and safety were associated with high levels of psychological distress in at least one racial/ethnic group (p \< 0.05). Our study suggests that Hispanic adults face more adverse social determinants of health and are disproportionately impacted by the pandemic. Public health practice and policy should highlight social determinants of heath that are associated with different racial/ethnic groups and develop tailored programs to reduce psychological distress. © 2022. The Author(s).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Frontiers in medicine                                                                                                                     | Evidence highlighted the likelihood of unmet mental health needs (UMHNs) among LGBTQ+ than non-LGBTQ+ populations during COVID-19. However, there lacks evidence to accurately answer to what extent the gap was in UMHN between LGBTQ+ and non-LGBTQ+ populations. We aim to evaluate the difference in UMHN between LGBTQ+ and non-LGBTQ+ during COVID-19. Cross-sectional data from Household Pulse Survey between 21 July 2021 and 9 May 2022 were analyzed. LGBTQ+ was defined based on self-reported sex at birth, gender, and sexual orientation identity. UMHN was assessed by a self-reported question. Multivariable logistic regressions generated adjusted odds ratios (AODs) of UMHN, both on overall and subgroups, controlling for a variety of socio-demographic and economic-affordability confounders. 81267 LGBTQ+ and 722638 non-LGBTQ+ were studied. The difference in UMHN between LGBTQ+ and non-LGBTQ+ (as reference) varied from 4.9% (95% CI 1.2-8.7%) in Hawaii to 16.0% (95% CI 12.2-19.7%) in Utah. In multivariable models, compared with non-LGBTQ+ populations, LGBTQ+ had a higher likelihood to report UMHN (AOR = 2.27, 95% CI 2.18-2.39), with the highest likelihood identified in transgender (AOR = 3.63, 95% CI 2.97-4.39); compared with LGBTQ+ aged 65+, LGBTQ+ aged 18-25 had a higher likelihood to report UMHN (AOR = 1.34, 95% CI 1.03-1.75); compared with White LGBTQ+ populations, Black and Hispanic LGBTQ+ had a lower likelihood to report UMHN (AOR = 0.72, 95% CI 0.63-0.82; AOR = 0.85, 95% CI 0.75-0.97, respectively). During the COVID-19, LGBTQ+ had a substantial additional risk of UMHN than non-LGBTQ+. Disparities among age groups, subtypes of LGBTQ+, and geographic variance were also identified. Copyright © 2022 Chen, Wang, She, Qin and Ming.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| PloS one                                                                                                                                  | Using bargaining agreement data from the Federal Mediation Conciliation Services, we found that the median national resident COVID-19 mortality percentage (as of April 24, 2022) of unionized nursing homes and that of nonunionized ones were not statically different (10.2% vs. 10.7%; P = 0.32). The median nursing home resident COVID-19 mortality percentage varied from 0% in Hawaii to above 16% in Rhode Island (16.6%). Unionized nursing homes had a statistically significant lower median mortality percentage than nonunionized nursing homes (P \< 0.1) in Missouri, and had a higher median mortality percentage than nonunionized nursing homes (P \< 0.05) in Alabama and Tennessee. Higher average resident age, lower percentage of Medicare residents, small size, for-profit ownership, and chain organization affiliation were associated with higher resident COVID-19 mortality percentage. Overall, no evidence was found that nursing home resident COVID-19 mortality percentage differed between unionized nursing homes and nonunionized nursing homes in the U.S. Copyright: © 2022 Olson et al. This is an open access article distributed under the terms of the Creative Commons Attribution License, which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| JAMA                                                                                                                                      | NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Preventing chronic disease                                                                                                                | The true extent of racial and ethnic disparities in COVID-19 hospitalizations may be hidden by misclassification of race and ethnicity. This study aimed to quantify this inaccuracy in a hospital’s electronic medical record (EMR) against the gold standard of self-identification and then project data onto state-level COVID-19 hospitalizations by self-identified race and ethnicity. To identify misclassification of race and ethnicity in the EMRs of a hospital in Honolulu, Hawaii, research and quality improvement staff members surveyed all available patients (N = 847) in 5 cohorts in 2007, 2008, 2010, 2013, and 2020 at randomly selected hospital and ambulatory units. The survey asked patients to self-identify up to 12 races and ethnicities. We compared these data with data from EMRs. We then estimated the number of COVID-19 hospitalizations by projecting racial misclassifications onto publicly available data. We determined significant differences via simulation-constructed medians and 95% CIs. EMR-based and self-identified race and ethnicity were the same in 86.5% of the sample. Native Hawaiians (79.2%) were significantly less likely than non-Native Hawaiians (89.4%) to be correctly classified on initial analysis; this difference was driven by Native Hawaiians being more likely than non-Native Hawaiians to be multiracial (93.4% vs 30.3%). When restricted to multiracial patients only, we found no significant difference in accuracy (P = .32). The number of COVID-19-related hospitalizations was 8.7% higher among Native Hawaiians and 3.9% higher among Pacific Islanders when we projected self-identified race and ethnicity rather than using EMR data. Using self-identified rather than hospital EMR data on race and ethnicity may uncover further disparities in COVID-19 hospitalizations.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Eastern Mediterranean health journal = La revue de sante de la Mediterranee orientale = al-Majallah al-sihhiyah li-sharq al-mutawassit    | Since winter 2020, excess deaths due to COVID-19 have been higher in Eastern Europe than most of Western Europe, partly because regulatory enforcement was poor. This paper analysed data from 50 countries in the WHO European Region, in addition to data from USA and Canada. Excess mMortality and vaccination data were retrieved from “Our World In Data” and regulation implementation was assessed using standard methods. Multiple linear regression was used to assess the association between mortality and each covariate. Excess mortality increased by 4.1 per 100 000 (P = 0.038) for every percentage decrease in vaccination rate and with 6/100 000 (p=0.011) for every decreased unit in the regulatory implementation score a country achieved in the Rule of Law Index. Degree of regulation enforcement, likely including public health measure enforcement, may be an important factor in controlling COVID-19’s deleterious health impacts. Copyright © Authors 2022; licensee World Health Organization. EMHJ is an open access journal. This paper is available under the Creative Commons Attribution Non-Commercial ShareAlike 3.0 IGO licence (CC BY-NC-SA 3.0 IGO; <https://creativecommons.org/licenses/by-nc-sa/3.0/igo>).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Hawai’i journal of health & social welfare                                                                                                | The COVID-19 pandemic increased stress and worry among faculty and staff members at universities across the US. To assess the well-being of university faculty and staff, a survey was administered at a medical school in the state of Hawai’i during early fall 2020. The purpose of the exploratory study was to assess and gauge faculty and staff members’ well-being regarding the school’s response to COVID-19. Participants in this study represented a convenience sample of compensated teaching, research, and administrative faculty and staff members. A total of 80 faculty and 73 staff members participated. Overall, faculty and staff reported relatively low levels of worries and stress. Staff members reported greater levels of worry and stress than faculty members in 8 of the 11 questions. Statistical differences were detected in 3 questions, with staff reporting higher levels of worry and stress in their health and well-being of themselves (P \< .001), paying bills (P \< .001), and losing their jobs (P \< .001). Both faculty and staff reported good overall satisfaction on the timeliness and clarity of messages that they received, support from leadership and the school, and support to adjust to changes in response to COVID-19. For both faculty and staff, the greatest worry or concern for the open-ended question on worry and stress was related to financial and economic issues. Data from this survey and can contribute to an understanding of medical school employee well-being during a major operational disruption and may help develop policies and programs to assist employees in different employment categories during future disruptions. ©Copyright 2022 by University Health Partners of Hawai‘i (UHP Hawai‘i).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Child: care, health and development                                                                                                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

        <ISOAbbreviation>Child Care Health Dev</ISOAbbreviation>
      </Journal>
      <ArticleTitle>A review of state regulations for child care: Preventing, recognizing and reporting child maltreatment.</ArticleTitle>
      <ELocationID EIdType="doi" ValidYN="Y">10.1111/cch.13080</ELocationID>
      <Abstract>
        <AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">Prior to the COVID-19 pandemic, nearly 60% of children under 5 years of age were cared for in out-of-home child care arrangements in the United States. Thus, child care provides an opportunity to identify and address potential child maltreatment. However, during the pandemic, rates of reporting child maltreatment decreased-likely because children spent less time in the presence of mandated reporters. As children return to child care, states must have regulations in place to help child care providers prevent, recognize and report child maltreatment. However, little is known about the extent to which state regulations address child maltreatment. Therefore, the purpose of this cross-sectional study was to assess state regulations related to child maltreatment and compare them to national standards.</AbstractText>
        <AbstractText Label="METHOD" NlmCategory="METHODS">We reviewed state regulations for all 50 states and the District of Columbia for child care centres ('centres') and family child care homes ('homes') through 31 July 2021 and compared these regulations to eight national health and safety standards on child maltreatment. We coded regulations as either not meeting, partially meeting or fully meeting each standard.</AbstractText>
        <AbstractText Label="RESULTS" NlmCategory="RESULTS">Three states (Colorado, Utah and Washington) had regulations for centres, and one state (Washington) had regulations for homes that at least partially met all eight national standards. Nearly all states had regulations consistent with the standards requiring that caregivers and teachers are mandated reporters of child maltreatment and requiring that they be trained in preventing, recognizing and reporting child maltreatment. One state (Hawaii) did not have regulations consistent with any of the national standards for either centres or homes.</AbstractText>
        <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">Generally, states lacked regulations related to the prevention, recognition and reporting of child maltreatment for both centres and homes. Encouraging states to adopt regulations that meet national standards and further exploring their impact on child welfare are important next steps.</AbstractText>
        <CopyrightInformation>© 2022 John Wiley &amp; Sons Ltd.</CopyrightInformation>
      </Abstract>
      <AuthorList CompleteYN="Y">
        <Author ValidYN="Y">
          <LastName>Grossman</LastName>
          <ForeName>Elyse R</ForeName>
          <Initials>ER</Initials>
          <Identifier Source="ORCID">0000-0003-1444-5804</Identifier>
          <AffiliationInfo>
            <Affiliation>Department of Health, Behavior and Society, John Hopkins Bloomberg School of Public Health, Baltimore, Maryland, USA.</Affiliation>
          </AffiliationInfo>
        </Author>
        <Author ValidYN="Y">
          <LastName>McClendon</LastName>
          <ForeName>Jasmine E</ForeName>
          <Initials>JE</Initials>
          <AffiliationInfo>
            <Affiliation>University of California Davis Medical Center, Sacramento, California, USA.</Affiliation>
          </AffiliationInfo>
        </Author>
        <Author ValidYN="Y">
          <LastName>Gielen</LastName>
          <ForeName>Andrea C</ForeName>
          <Initials>AC</Initials>
          <AffiliationInfo>
            <Affiliation>Department of Health, Behavior and Society, John Hopkins Bloomberg School of Public Health, Baltimore, Maryland, USA.</Affiliation>
          </AffiliationInfo>
        </Author>
        <Author ValidYN="Y">
          <LastName>McDonald</LastName>
          <ForeName>Eileen M</ForeName>
          <Initials>EM</Initials>
          <AffiliationInfo>
            <Affiliation>Department of Health, Behavior and Society, John Hopkins Bloomberg School of Public Health, Baltimore, Maryland, USA.</Affiliation>
          </AffiliationInfo>
        </Author>
        <Author ValidYN="Y">
          <LastName>Benjamin-Neelon</LastName>
          <ForeName>Sara E</ForeName>
          <Initials>SE</Initials>
          <Identifier Source="ORCID">0000-0003-4643-2397</Identifier>
          <AffiliationInfo>
            <Affiliation>Department of Health, Behavior and Society, John Hopkins Bloomberg School of Public Health, Baltimore, Maryland, USA.</Affiliation>
          </AffiliationInfo>
        </Author>
      </AuthorList>
      <Language>eng</Language>
      <GrantList CompleteYN="Y">
        <Grant>
          <GrantID>1R49CE002466</GrantID>
          <Acronym>CC</Acronym>
          <Agency>CDC HHS</Agency>
          <Country>United States</Country>
        </Grant>
        <Grant>
          <GrantID>1R49CE002466</GrantID>
          <Acronym>CC</Acronym>
          <Agency>CDC HHS</Agency>
          <Country>United States</Country>
        </Grant>
      </GrantList>
      <PublicationTypeList>
        <PublicationType UI="D016428">Journal Article</PublicationType>
      </PublicationTypeList>
      <ArticleDate DateType="Electronic">
        <Year>2022</Year>
        <Month>11</Month>
        <Day>14</Day>
      </ArticleDate>
    </Article>
    <MedlineJournalInfo>
      <Country>England</Country>
      <MedlineTA>Child Care Health Dev</MedlineTA>
      <NlmUniqueID>7602632</NlmUniqueID>
      <ISSNLinking>0305-1862</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">child care</Keyword>
      <Keyword MajorTopicYN="N">child maltreatment</Keyword>
      <Keyword MajorTopicYN="N">mandatory reporting</Keyword>
      <Keyword MajorTopicYN="N">state regulations</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="revised"> <Year>2022</Year> <Month>9</Month>
<Day>23</Day> </PubMedPubDate> <PubMedPubDate PubStatus="received">
<Year>2021</Year> <Month>12</Month> <Day>10</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="accepted"> <Year>2022</Year> <Month>11</Month>
<Day>5</Day> </PubMedPubDate> <PubMedPubDate PubStatus="pubmed">
<Year>2022</Year> <Month>11</Month> <Day>16</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="medline">
<Year>2022</Year> <Month>11</Month> <Day>16</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="entrez">
<Year>2022</Year> <Month>11</Month> <Day>15</Day> <Hour>3</Hour>
<Minute>2</Minute> </PubMedPubDate> </History>
<PublicationStatus>aheadofprint</PublicationStatus> <ArticleIdList>
<ArticleId IdType="pubmed">36377347</ArticleId>
<ArticleId IdType="doi">10.1111/cch.13080</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|Prior to the COVID-19 pandemic, nearly 60%
of children under 5 years of age were cared for in out-of-home child
care arrangements in the United States. Thus, child care provides an
opportunity to identify and address potential child maltreatment.
However, during the pandemic, rates of reporting child maltreatment
decreased-likely because children spent less time in the presence of
mandated reporters. As children return to child care, states must have
regulations in place to help child care providers prevent, recognize and
report child maltreatment. However, little is known about the extent to
which state regulations address child maltreatment. Therefore, the
purpose of this cross-sectional study was to assess state regulations
related to child maltreatment and compare them to national standards. We
reviewed state regulations for all 50 states and the District of
Columbia for child care centres (‘centres’) and family child care homes
(‘homes’) through 31 July 2021 and compared these regulations to eight
national health and safety standards on child maltreatment. We coded
regulations as either not meeting, partially meeting or fully meeting
each standard. Three states (Colorado, Utah and Washington) had
regulations for centres, and one state (Washington) had regulations for
homes that at least partially met all eight national standards. Nearly
all states had regulations consistent with the standards requiring that
caregivers and teachers are mandated reporters of child maltreatment and
requiring that they be trained in preventing, recognizing and reporting
child maltreatment. One state (Hawaii) did not have regulations
consistent with any of the national standards for either centres or
homes. Generally, states lacked regulations related to the prevention,
recognition and reporting of child maltreatment for both centres and
homes. Encouraging states to adopt regulations that meet national
standards and further exploring their impact on child welfare are
important next steps. © 2022 John Wiley & Sons Ltd. \| \|Vaccine
\|Monitoring COVID-19 vaccine hesitancy helps design and implement
strategies to increase vaccine uptake. Utilizing the large scale
cross-sectional Household Pulse Survey data collected between July 21
and October 11 in 2021, this study aims to construct measures of
COVID-19 vaccine hesitancy and identify demographic disparities among
U.S. adults (18y+). Factor analysis identified three factors of vaccine
hesitancy: safety concerns (prevalence: 70.1 %). trust issues (53.5 %),
and not seen as necessary (33.8 %). Among those who did not show
willingness to receive COVID-19 vaccine, females were more likely to
have safety concerns (73.7 %) compared to males (66.7 %), but less
likely to have trust issues (female: 49.7 %; male: 57.1 %) or not seen
as necessary (female: 23.8 %; male 43.4 %). Higher education was
associated with higher prevalence of not seen as necessary. Younger
adults and Whites had higher prevalence of having trust issues and not
seen as necessary compared to their counter parts. Copyright © 2022
Elsevier Ltd. All rights reserved. \| \|Journal of perinatal medicine
<ISOAbbreviation>J Perinat Med</ISOAbbreviation> </Journal>
<ArticleTitle>Maternal telehealth: innovations and Hawai’i
perspectives.</ArticleTitle> <Pagination> <StartPage>69</StartPage>
<EndPage>82</EndPage> <MedlinePgn>69-82</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1515/jpm-2022-0394</ELocationID>
<Abstract> <AbstractText>Access to maternal-fetal medicine (MFM)
subspecialty services is a critical part of a healthcare system that
optimizes pregnancy outcomes for women with complex medical and
obstetrical disorders. Healthcare services in the State of Hawai’i
consist of a complicated patchwork of independently run community health
clinics and hospital systems which are difficult for many pregnant
patients to navigate. Maternal telehealth services have been identified
as a solution to increase access to subspecialty prenatal services for
women in rural communities or neighboring islands, especially during the
COVID-19 pandemic. Telehealth innovations have been rapidly developing
in the areas of remote ultrasound, hypertension management, diabetes
management, and fetal monitoring. This report describes how telehealth
innovations are being introduced by MFM specialists to optimize care for
a unique population of high-risk patients in a remote area of the world
such as Hawai’i, as well as review currently available telemedicine
technologies and future innovations.</AbstractText>
<CopyrightInformation>© 2022 Walter de Gruyter GmbH,
Berlin/Boston.</CopyrightInformation> </Abstract>
<AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Sullivan</LastName> <ForeName>Cathlyn</ForeName>
<Initials>C</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, Division of Maternal Fetal Medicine,
University of Hawai’i, John A. Burns School of Medicine, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Cazin</LastName> <ForeName>Marguerite</ForeName>
<Initials>M</Initials> <AffiliationInfo> <Affiliation>Occidental
College, Los Angeles, CA, USA.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Higa</LastName>
<ForeName>Christina</ForeName> <Initials>C</Initials> <AffiliationInfo>
<Affiliation>Social Science Research Institute, University of Hawai’i,
College of Social Sciences, Honolulu, HI, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Zalud</LastName> <ForeName>Ivica</ForeName>
<Initials>I</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, Division of Maternal Fetal Medicine,
University of Hawai’i, John A. Burns School of Medicine, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Lee</LastName> <ForeName>Men-Jean</ForeName>
<Initials>MJ</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, Division of Maternal Fetal Medicine,
University of Hawai’i, John A. Burns School of Medicine, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> </AuthorList>
<Language>eng</Language> <PublicationTypeList>
<PublicationType UI="D016428">Journal Article</PublicationType>
<PublicationType UI="D016454">Review</PublicationType>
</PublicationTypeList> <ArticleDate DateType="Electronic">
<Year>2022</Year> <Month>11</Month> <Day>14</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>Germany</Country>
      <MedlineTA>J Perinat Med</MedlineTA>
      <NlmUniqueID>0361031</NlmUniqueID>
      <ISSNLinking>0300-5577</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D011247" MajorTopicYN="N">Pregnancy</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006254" MajorTopicYN="N" Type="Geographic">Hawaii</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D058873" MajorTopicYN="N">Pandemics</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="Y">COVID-19</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D017216" MajorTopicYN="Y">Telemedicine</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D012424" MajorTopicYN="N">Rural Population</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">Hawaiʻi</Keyword>
      <Keyword MajorTopicYN="N">maternal health</Keyword>
      <Keyword MajorTopicYN="N">obstetrics</Keyword>
      <Keyword MajorTopicYN="N">remote fetal monitoring</Keyword>
      <Keyword MajorTopicYN="N">rural health</Keyword>
      <Keyword MajorTopicYN="N">telehealth</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="received"> <Year>2022</Year> <Month>8</Month>
<Day>15</Day> </PubMedPubDate> <PubMedPubDate PubStatus="accepted">
<Year>2022</Year> <Month>10</Month> <Day>27</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="pubmed"> <Year>2022</Year> <Month>11</Month>
<Day>12</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="medline"> <Year>2023</Year> <Month>1</Month>
<Day>17</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="entrez"> <Year>2022</Year> <Month>11</Month>
<Day>11</Day> <Hour>15</Hour> <Minute>52</Minute> </PubMedPubDate>
</History> <PublicationStatus>epublish</PublicationStatus>
<ArticleIdList> <ArticleId IdType="pubmed">36367993</ArticleId>
<ArticleId IdType="doi">10.1515/jpm-2022-0394</ArticleId>
<ArticleId IdType="pii">jpm-2022-0394</ArticleId> </ArticleIdList>
<ReferenceList> References \|Access to maternal-fetal medicine (MFM)
subspecialty services is a critical part of a healthcare system that
optimizes pregnancy outcomes for women with complex medical and
obstetrical disorders. Healthcare services in the State of Hawai’i
consist of a complicated patchwork of independently run community health
clinics and hospital systems which are difficult for many pregnant
patients to navigate. Maternal telehealth services have been identified
as a solution to increase access to subspecialty prenatal services for
women in rural communities or neighboring islands, especially during the
COVID-19 pandemic. Telehealth innovations have been rapidly developing
in the areas of remote ultrasound, hypertension management, diabetes
management, and fetal monitoring. This report describes how telehealth
innovations are being introduced by MFM specialists to optimize care for
a unique population of high-risk patients in a remote area of the world
such as Hawai’i, as well as review currently available telemedicine
technologies and future innovations. © 2022 Walter de Gruyter GmbH,
Berlin/Boston. \| \|Birth defects research \|Remdesivir is an antiviral
drug approved for the treatment of COVID-19, whose developmental
toxicity remains unclear. More information about the safety of
remdesivir is urgently needed for people of childbearing potential, who
are affected by the ongoing pandemic. Morphogenetic embryoid bodies
(MEBs) are three-dimensional (3D) aggregates of pluripotent stem cells
that recapitulate embryonic body patterning in vitro, and have been used
as effective embryo models to detect the developmental toxicity of
chemical exposures specifically and sensitively. MEBs were generated
from mouse P19C5 and human H9 pluripotent stem cells, and used to
examine the effects of remdesivir. The morphological effects were
assessed by analyzing the morphometric parameters of MEBs after exposure
to varying concentrations of remdesivir. The molecular impact of
remdesivir was evaluated by measuring the transcript levels of
developmental regulator genes. The mouse MEB morphogenesis was impaired
by remdesivir at 1-8 μM. Remdesivir affected MEBs in a manner dependent
on metabolic conversion, and its potency was higher than GS-441524 and
GS-621763, presumptive anti-COVID-19 drugs that act similarly to
remdesivir. The expressions of developmental regulator genes,
particularly those involved in axial and somite patterning, were
dysregulated by remdesivir. The early stage of MEB development was more
vulnerable to remdesivir exposure than the later stage. The
morphogenesis and gene expression profiles of human MEBs were also
impaired by remdesivir at 1-8 μM. Remdesivir impaired mouse and human
MEBs at concentrations that are comparable to the therapeutic plasma
levels in humans, urging further investigation into the potential impact
of remdesivir on developing embryos. © 2022 The Authors. Birth Defects
Research published by Wiley Periodicals LLC. \| \|American journal of
preventive medicine \|Shelter-in-place orders altered facilitators and
barriers to tobacco use (e.g., outlet closures, restricted social
gatherings). This study examined whether the duration of time in shelter
in place and compliance with different shelter-in-place orders
influenced adolescent cigarette and E-cigarette use and how the use may
differ by demographic characteristics. Shelter-in-place policy data
obtained from government websites were merged with cross-sectional 2020
survey data on adolescents in California. Treatment variables included
the proportion of time in shelter in place and self-reported compliance
with shelter-in-place orders (for essential businesses and retail spaces
and social and outdoor contexts). Multilevel logit models for
dichotomous past 6-month cigarette and E-cigarette use and multilevel
negative binomial regression models for past 6-month frequency of use
were used. Moderation analyses were conducted on demographic measures.
The sample included 1,196 adolescents (mean age=15.8 years, age
range=13-19 years, 49.2% female, 50.0% White). Analyses were conducted
in 2022. No associations were found between the proportion of time in
shelter in place and outcomes. Shelter-in-place compliance with
essential business and retail space orders was associated with lower
odds of using cigarettes and E-cigarettes in the past 6 months.
Compliance with social and outdoor context-related orders were
associated with lower odds of using E-cigarettes and fewer days using
cigarettes and E-cigarettes. Being aged ≥18 years moderated the
associations between essential business/retail space and social/outdoor
context-related shelter-in-place compliance orders and past 6-month
frequency of cigarette smoking. Findings support tailored interventions
for less compliant and older adolescents for future pandemic mitigation
measures. Copyright © 2022 American Journal of Preventive Medicine.
Published by Elsevier Inc. All rights reserved. \| \|Anatomia,
histologia, embryologia <ISOAbbreviation>Anat Histol
Embryol</ISOAbbreviation> </Journal> <ArticleTitle>Extended reality
veterinary medicine case studies for diagnostic veterinary imaging
instruction: Assessing student perceptions and examination
performance.</ArticleTitle> <Pagination> <StartPage>101</StartPage>
<EndPage>114</EndPage> <MedlinePgn>101-114</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1111/ahe.12879</ELocationID>
<Abstract> <AbstractText>Educational technologies in veterinary medicine
aim to train veterinarians faster and improve clinical outcomes.
COVID-19 pandemic, shifted face-to-face teaching to online, thus, the
need to provide effective education remotely was exacerbated. Among
recent technology advances for veterinary medical education, extended
reality (XR) is a promising teaching tool. This study aimed to develop a
case resolution approach for radiographic anatomy studies using XR
technology and assess students’ achievement of differential diagnostic
skills. Learning objectives based on Bloom’s taxonomy keywords were used
to develop four clinical cases (3 dogs/1 cat) of spinal injuries
utilizing CT scans and XR models and presented to 22 third-year
veterinary medicine students. Quantitative assessment (ASMT) of 7
questions probing ‘memorization’, ‘understanding and application’,
‘analysis’ and ‘evaluation’ was given before and after contact with XR
technology as well as qualitative feedback via a survey. Mean ASMT
scores increased during case resolution (pre 51.6% (±37%)/post 60.1%
(± 34%); p \< 0.01), but without significant difference between cases
(Kruskal-Wallis H = 2.18, NS). Learning objectives were examined for six
questions (Q1-Q6) across cases (C1-4): Memorization improved
sequentially (Q1, 2 8/8), while Understanding and Application (Q3,4)
showed the greatest improvement (26.7%-76.9%). Evaluation and Analysis
(Q5,6) was somewhat mixed, improving (5/8), no change (3/8) and
declining (1/8).Positive student perceptions suggest that case studies’
online delivery was well received stimulating learning in diagnostic
imaging and anatomy while developing visual-spatial skills that aid
understanding cross-sectional images. Therefore, XR technology could be
a useful approach to complement radiological instruction in veterinary
medicine.</AbstractText> <CopyrightInformation>© 2022 Wiley-VCH
GmbH.</CopyrightInformation> </Abstract> <AuthorList CompleteYN="Y">
<Author ValidYN="Y"> <LastName>Guaraná</LastName> <ForeName>Julia
B</ForeName> <Initials>JB</Initials>
<Identifier Source="ORCID">0000-0002-6964-0589</Identifier>
<AffiliationInfo> <Affiliation>Department of Veterinary Medicine,
Faculty of Animal Science and Food Engineering, University of São Paulo
(USP), São Paulo, Brazil.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Aytaç</LastName>
<ForeName>Güneş</ForeName> <Initials>G</Initials>
<Identifier Source="ORCID">0000-0003-4902-2844</Identifier>
<AffiliationInfo> <Affiliation>Department of Anatomy, Biochemistry &
Physiology, John A. Burns School of Medicine, University of Hawaii (UH),
Honolulu, Hawaii, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Müller</LastName> <ForeName>Alois
F</ForeName> <Initials>AF</Initials>
<Identifier Source="ORCID">0000-0002-4053-9161</Identifier>
<AffiliationInfo> <Affiliation>Department of Veterinary Medicine,
Faculty of Animal Science and Food Engineering, University of São Paulo
(USP), São Paulo, Brazil.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Thompson</LastName>
<ForeName>Jesse</ForeName> <Initials>J</Initials>
<Identifier Source="ORCID">0000-0003-0639-033X</Identifier>
<AffiliationInfo> <Affiliation>Department of Anatomy, Biochemistry &
Physiology, John A. Burns School of Medicine, University of Hawaii (UH),
Honolulu, Hawaii, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Freitas</LastName> <ForeName>Silvio
H</ForeName> <Initials>SH</Initials>
<Identifier Source="ORCID">0000-0001-7595-0367</Identifier>
<AffiliationInfo> <Affiliation>Department of Veterinary Medicine,
Faculty of Animal Science and Food Engineering, University of São Paulo
(USP), São Paulo, Brazil.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Lee</LastName>
<ForeName>U-Young</ForeName> <Initials>UY</Initials>
<Identifier Source="ORCID">0000-0002-8990-1213</Identifier>
<AffiliationInfo> <Affiliation>Department of Anatomy, College of
Medicine, The Catholic University of Korea (CUK), Seoul, South
Korea.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Lozanoff</LastName> <ForeName>Scott</ForeName>
<Initials>S</Initials>
<Identifier Source="ORCID">0000-0001-9089-3153</Identifier>
<AffiliationInfo> <Affiliation>Department of Anatomy, Biochemistry &
Physiology, John A. Burns School of Medicine, University of Hawaii (UH),
Honolulu, Hawaii, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Ferrante</LastName>
<ForeName>Bruno</ForeName> <Initials>B</Initials>
<Identifier Source="ORCID">0000-0001-9783-7744</Identifier>
<AffiliationInfo> <Affiliation>Department of Veterinary Medicine,
Faculty of Animal Science and Food Engineering, University of São Paulo
(USP), São Paulo, Brazil.</Affiliation> </AffiliationInfo>
<AffiliationInfo> <Affiliation>Veterinary Clinical and Surgery
Department of Veterinary School, Federal University of Minas Gerais
(UFMG), Belo Horizonte, Minas Gerais, Brazil.</Affiliation>
</AffiliationInfo> </Author> </AuthorList> <Language>eng</Language>
<PublicationTypeList> <PublicationType UI="D016428">Journal
Article</PublicationType> </PublicationTypeList>
<ArticleDate DateType="Electronic"> <Year>2022</Year> <Month>11</Month>
<Day>01</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>Germany</Country>
      <MedlineTA>Anat Histol Embryol</MedlineTA>
      <NlmUniqueID>7704218</NlmUniqueID>
      <ISSNLinking>0340-2096</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000818" MajorTopicYN="N">Animals</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004285" MajorTopicYN="N">Dogs</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="N">COVID-19</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004283" MajorTopicYN="Y">Dog Diseases</DescriptorName>
        <QualifierName UI="Q000000981" MajorTopicYN="N">diagnostic imaging</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D007858" MajorTopicYN="N">Learning</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D058873" MajorTopicYN="N">Pandemics</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D014057" MajorTopicYN="N">Tomography, X-Ray Computed</DescriptorName>
        <QualifierName UI="Q000662" MajorTopicYN="N">veterinary</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002371" MajorTopicYN="Y">Cat Diseases</DescriptorName>
        <QualifierName UI="Q000000981" MajorTopicYN="N">diagnostic imaging</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004520" MajorTopicYN="Y">Education, Veterinary</DescriptorName>
        <QualifierName UI="Q000458" MajorTopicYN="N">organization &amp; administration</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D013337" MajorTopicYN="Y">Students, Medical</DescriptorName>
        <QualifierName UI="Q000523" MajorTopicYN="N">psychology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D020375" MajorTopicYN="Y">Education, Distance</DescriptorName>
        <QualifierName UI="Q000458" MajorTopicYN="N">organization &amp; administration</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004521" MajorTopicYN="N">Educational Measurement</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">distance learning</Keyword>
      <Keyword MajorTopicYN="N">extended reality</Keyword>
      <Keyword MajorTopicYN="N">radiological anatomy</Keyword>
      <Keyword MajorTopicYN="N">visual-spatial learning</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="revised"> <Year>2022</Year> <Month>8</Month>
<Day>7</Day> </PubMedPubDate> <PubMedPubDate PubStatus="received">
<Year>2022</Year> <Month>6</Month> <Day>30</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="accepted"> <Year>2022</Year> <Month>8</Month>
<Day>31</Day> </PubMedPubDate> <PubMedPubDate PubStatus="pubmed">
<Year>2022</Year> <Month>11</Month> <Day>2</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="medline">
<Year>2023</Year> <Month>1</Month> <Day>19</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="entrez">
<Year>2022</Year> <Month>11</Month> <Day>1</Day> <Hour>5</Hour>
<Minute>53</Minute> </PubMedPubDate> </History>
<PublicationStatus>ppublish</PublicationStatus> <ArticleIdList>
<ArticleId IdType="pubmed">36317584</ArticleId>
<ArticleId IdType="doi">10.1111/ahe.12879</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|Educational technologies in veterinary
medicine aim to train veterinarians faster and improve clinical
outcomes. COVID-19 pandemic, shifted face-to-face teaching to online,
thus, the need to provide effective education remotely was exacerbated.
Among recent technology advances for veterinary medical education,
extended reality (XR) is a promising teaching tool. This study aimed to
develop a case resolution approach for radiographic anatomy studies
using XR technology and assess students’ achievement of differential
diagnostic skills. Learning objectives based on Bloom’s taxonomy
keywords were used to develop four clinical cases (3 dogs/1 cat) of
spinal injuries utilizing CT scans and XR models and presented to 22
third-year veterinary medicine students. Quantitative assessment (ASMT)
of 7 questions probing ‘memorization’, ‘understanding and application’,
‘analysis’ and ‘evaluation’ was given before and after contact with XR
technology as well as qualitative feedback via a survey. Mean ASMT
scores increased during case resolution (pre 51.6% (±37%)/post 60.1% (±
34%); p \< 0.01), but without significant difference between cases
(Kruskal-Wallis H = 2.18, NS). Learning objectives were examined for six
questions (Q1-Q6) across cases (C1-4): Memorization improved
sequentially (Q1, 2 8/8), while Understanding and Application (Q3,4)
showed the greatest improvement (26.7%-76.9%). Evaluation and Analysis
(Q5,6) was somewhat mixed, improving (5/8), no change (3/8) and
declining (1/8).Positive student perceptions suggest that case studies’
online delivery was well received stimulating learning in diagnostic
imaging and anatomy while developing visual-spatial skills that aid
understanding cross-sectional images. Therefore, XR technology could be
a useful approach to complement radiological instruction in veterinary
medicine. © 2022 Wiley-VCH GmbH. \| \|The Lancet. Infectious diseases
\|NA \| \|iScience \|The MetaSUB Consortium, founded in 2015, is a
global consortium with an interdisciplinary team of clinicians,
scientists, bioinformaticians, engineers, and designers, with members
from more than 100 countries across the globe. This network has
continually collected samples from urban and rural sites including
subways and transit systems, sewage systems, hospitals, and other
environmental sampling. These collections have been ongoing since 2015
and have continued when possible, even throughout the COVID-19 pandemic.
The consortium has optimized their workflow for the collection,
isolation, and sequencing of DNA and RNA collected from these various
sites and processing them for metagenomics analysis, including the
identification of SARS-CoV-2 and its variants. Here, the Consortium
describes its foundations, and its ongoing work to expand on this
network and to focus its scope on the mapping, annotation, and
prediction of emerging pathogens, mapping microbial evolution and
antibiotic resistance, and the discovery of novel organisms and
biosynthetic gene clusters. © 2022 The Authors. \| \|Nature
communications \|Genes with moderate to low expression heritability may
explain a large proportion of complex trait etiology, but such genes
cannot be sufficiently captured in conventional transcriptome-wide
association studies (TWASs), partly due to the relatively small
available reference datasets for developing expression genetic
prediction models to capture the moderate to low genetically regulated
components of gene expression. Here, we introduce a method, the
Summary-level Unified Method for Modeling Integrated Transcriptome
(SUMMIT), to improve the expression prediction model accuracy and the
power of TWAS by using a large expression quantitative trait loci (eQTL)
summary-level dataset. We apply SUMMIT to the eQTL summary-level data
provided by the eQTLGen consortium. Through simulation studies and
analyses of genome-wide association study summary statistics for 24
complex traits, we show that SUMMIT improves the accuracy of expression
prediction in blood, successfully builds expression prediction models
for genes with low expression heritability, and achieves higher
statistical power than several benchmark methods. Finally, we conduct a
case study of COVID-19 severity with SUMMIT and identify 11 likely
causal genes associated with COVID-19 severity. © 2022. The Author(s).
\| \|Clinical nurse specialist CNS \|To gain insights in how women use
technology to address health information needs during the prenatal and
postpartum time frame. An exploratory qualitative study recruited
pregnant and recent postpartum women to share their perspectives on
information they needed and how they obtained it. Women who were
pregnant or \<90 days postpartum (n = 26) were recruited via social
media and invited to share their experiences. Design thinking
methodology was used to develop questions to understand information
needs in the perinatal period as well as in context of the COVID-19
pandemic. Verbatim transcripts were coded by the research team according
to Braun and Clarke’s reflexive thematic analysis. Five themes explain
the experience of seeking information to support the perinatal period.
Women explained the need for the following: (1) information and
relationships are inseparable, (2) current practices leave needs unmet,
(3) the pandemic exposes vulnerability in prenatal care, (4) left to
figure it out alone, and (5) bridging the gap through technology.
Aggregated findings suggest how usual care can be modified to improve
support for women through personalized care, improved information
support, and use of technology. The study findings inform innovative
strategies using current technologies to improve health promotion in a
dynamic health environment. Copyright © 2022 Wolters Kluwer Health,
Inc. All rights reserved. \| \|JMIR public health and surveillance \|The
Covid-19 pandemic had many unprecedented secondary outcomes resulting in
various mental health issues leading to substance use coping behaviors.
The extent of substance use changes in a US sample by nativity have not
been previously described. Our aim was to design an online survey to
assess the social distancing and isolation issues exacerbated by the
Covid-19 pandemic to describe substance use as coping behaviors by
comparing substance use changes prior to and during the pandemic. A
comprehensive 116-item survey to understand the impact of Covid-19 and
social distancing on physical and psychosocial mental health and chronic
diseases was designed. Approximately 10,000 online surveys were
distributed by Qualtrics LLC between 13 May 2021 and 09 January 2022
across the US (i.e., continental US, Hawaii, Alaska, and territories) to
adults 18 and older. We oversampled low income and rural adults among
non-Hispanic White, non-Hispanic Black, Hispanic/Latino adults, and
foreign-born participants. Of the 5,938 surveys returned, 5,413 surveys
were used after Qualtrics proprietary expert review fraud detection, and
detailed assessments of completion rate and timing to complete the
survey. Participant demographics, substance use coping behaviors, and
substance use prior to and during the pandemic are described by the
overall US resident sample, followed by US born and foreign-born
self-report. Substance use included tobacco use, e-cigarettes/vaping,
alcohol use, marijuana use, and other illicit substance use. Marginal
homogeneity based on Stuart-Maxwell test was used to assess changes in
self-reported substance use prior to and during the pandemic. The sample
was mostly White (2182/5413, 40.3%), women (3369/5406, 62.3%) that
identified as straight/heterosexual (4805/5406, 89.3%), reported making
≥\$75,000 (1405/5355, 26.2%) and had vocational/technical training
(1746/5404, 32.3%). Significant changes were found in alcohol use
self-reports the overall sample. Significant changes were also observed
among US born and foreign-born samples. Similarities were observed
between US born and foreign-born participants on increased alcohol use
from: (1) no alcohol use prior to the pandemic to using alcohol once to
several times a month; and (2) once to several times per week to
everyday to several times per day. While significant changes were
observed from no prior alcohol use to some level of increased use, the
inverse was observed as well and more pronounced among foreign-born
participants. That is, there was a 5.1% overall change in some level of
alcohol use prior to the pandemic to no alcohol use during the pandemic
among foreign born, compared to the 4.3% change among US born. To better
prepare for the inadvertent effects of public health policies meant to
protect individuals, we must understand the mental health burdens that
can precipitate into substance use coping mechanisms that not only have
a deleterious effect on physical and mental health but exacerbate
morbidity and mortality to disease like Covid-19. \| \|Earth systems and
environment \|The unprecedented outbreak of Coronavirus Disease 2019
(COVID-19) has impacted the whole world in every aspect including
health, social life, economic activity, education, and the environment.
The pandemic has led to an improvement in air quality all around the
world, including in Malaysia. Lockdowns have resulted in industry
shutting down and road travel decreasing which can reduce the emission
of Greenhouse Gases (GHG) and air pollution. This research assesses the
impact of the COVID-19 lockdown on emissions using the Air Pollution
Index (API), aerosols, and GHG which is Nitrogen Dioxide (NO2) in
Malaysia. The data used is from Sentinel-5p and Sentinel-2A which
monitor the air quality based on Ozone (O3) and NO2 concentration. Using
an interpolated API Index Map comparing 2019, before the implementation
of a Movement Control Order (MCO), and 2020, after the MCO period we
examine the impact on pollution during and after the COVID-19 lockdown.
Data used Sentinel-5p, Sentinel-2A, and Air Pollution Index of Malaysia
(APIMS) to monitor the air quality that contains NO2 concentration. The
result has shown the recovery in air quality during the MCO
implementation which indirectly shows anthropogenic activities towards
the environmental condition. The study will help to enhance and support
the policy and scope for air pollution management strategies as well as
raise public awareness of the main causes that contribute to air
pollution. © King Abdulaziz University and Springer Nature Switzerland
AG 2022, Springer Nature or its licensor holds exclusive rights to this
article under a publishing agreement with the author(s) or other
rightsholder(s); author self-archiving of the accepted manuscript
version of this article is solely governed by the terms of such
publishing agreement and applicable law. \| \|Preventing chronic disease
\|Forgone health care, defined as not using health care despite
perceiving a need for it, is associated with poor health outcomes,
especially among people with chronic conditions. The objective of our
study was to examine how the pandemic affected forgone health care
during 3 stages of the pandemic. We used the Medicare Current
Beneficiary Survey COVID-19 Rapid Response Questionnaire administered in
summer 2020, fall 2020, and winter 2021 to examine sociodemographic
characteristics, chronic diseases, COVID-19 vaccination status, and
telehealth availability in relation to beneficiary reports of forgone
health care. Of the 3 periods studied, the overall rate of forgone
health care was highest in summer 2020 (20.8%), followed by fall 2020
(7.8%) and winter 2021 (6.5%). COVID-19 vaccination status, age, sex,
race and ethnicity, US region, availability of primary care telehealth
appointments, and chronic conditions (heart disease, arthritis,
depression, osteoporosis or a broken hip, and diabetes or high blood
glucose) were significantly related to forgone care. High rates of
forgone care among Medicare participants varied over time and were
significantly related to beneficiary characteristics. Our findings
highlight the need for health care reform and changes in policy to
address the issue of access to care for people with chronic conditions
during a pandemic or other public health emergency. \| \|Communications
biology \|SARS-CoV-2 worldwide spread and evolution has resulted in
variants containing mutations resulting in immune evasive epitopes that
decrease vaccine efficacy. We acquired SARS-CoV-2 positive clinical
samples and compared the worldwide emerged spike mutations from Variants
of Concern/Interest, and developed an algorithm for monitoring the
evolution of SARS-CoV-2 in the context of vaccines and monoclonal
antibodies. The algorithm partitions logarithmic-transformed prevalence
data monthly and Pearson’s correlation determines exponential emergence
of amino acid substitutions (AAS) and lineages. The SARS-CoV-2 genome
evaluation indicated 49 mutations, with 44 resulting in AAS. Nine of the
ten most worldwide prevalent (\>70%) spike protein changes have
Pearson’s coefficient r \> 0.9. The tenth, D614G, has a prevalence \>99%
and r-value of 0.67. The resulting algorithm is based on the patterns
these ten substitutions elucidated. The strong positive correlation of
the emerged spike protein changes and algorithmic predictive value can
be harnessed in designing vaccines with relevant immunogenic epitopes.
Monitoring, next-generation vaccine design, and mAb clinical efficacy
must keep up with SARS-CoV-2 evolution, as the virus is predicted to
remain endemic. © 2022. The Author(s). \| \|The Lancet. Respiratory
medicine \|The SARS-CoV-2 omicron (B.1.1.529 BA.1) lineage was first
detected in November, 2021, and is associated with reduced vaccine
effectiveness. By March, 2022, BA.1 had been replaced by sub-lineage
BA.2 in the USA. As new variants evolve, vaccine performance must be
continually assessed. We aimed to evaluate the effectiveness and
durability of BNT162b2 (Pfizer-BioNTech) against hospital and emergency
department admissions for BA.1 and BA.2. In this test-negative,
case-control study, we sourced data from the electronic health records
of adult (aged ≥18 years) members of Kaiser Permanente Southern
California (KPSC), which is a health-care system in the USA, who were
admitted to one of 15 KPSC hospitals or emergency departments (without
subsequent hospitalisation) between Dec 27, 2021, and June 4, 2022, with
an acute respiratory infection and were tested for SARS-CoV-2 by RT-PCR.
Omicron sub-lineage was determined by use of sequencing, spike gene
target failure, and the predominance of variants in certain time
periods. Our main outcome was the effectiveness of two or three doses of
BNT162b2 in preventing emergency department or hospital admission.
Variant-specific vaccine effectiveness was evaluated by comparing the
odds ratios from logistic regression models of vaccination between
test-positive cases and test-negative controls, adjusting for the month
of admission, age, sex, race and ethnicity, body-mass index, Charlson
Comorbidity Index, previous influenza or pneumococcal vaccines, and
previous SARS-CoV-2 infection. We also assessed effectiveness by the
time since vaccination. This study is registered at ClinicalTrials.gov,
NCT04848584, and is ongoing. Of 65 813 total admissions during the study
period, we included 16 994 in our analyses, of which 7435 were due to
BA.1, 1056 were due to BA.2, and 8503 were not due to SARS-CoV-2. In
adjusted analyses, two-dose vaccine effectiveness was 40% (95% CI 27 to
50) for hospitalisation and 29% (18 to 38) for emergency department
admission against BA.1 and 56% (31 to 72) for hospitalisation and 16%
(-5 to 33) for emergency department admission against BA.2. Three-dose
vaccine effectiveness was 79% (74 to 83) for hospitalisation and 72% (67
to 77) for emergency department admission against BA.1 and 71% (55 to
81) for hospitalisation and 21% (1 to 37) for emergency department
admission against BA.2. Less than 3 months after the third dose, vaccine
effectiveness was 80% (74 to 84) for hospitalisation and 74% (69 to 78)
for emergency department admission against BA.1. Vaccine effectiveness 3
months or more after the third dose was 76% (69 to 82) against
BA.1-related hospitalisation and 65% (56 to 73) against BA.1-related
emergency department admission. Against BA.2, vaccine effectiveness was
74% (47 to 87) for hospitalisation and 59% (40 to 72) for emergency
department admission at less than 3 months after the third dose and 70%
(53 to 81) for hospitalisation and 5% (-21 to 25) for emergency
department admission at 3 months or more after the third dose. Two doses
of BNT162b2 provided only partial protection against BA.1-related and
BA.2-related hospital and emergency department admission, which
underscores the need for booster doses against omicron. Although three
doses offered high levels of protection (≥70%) against hospitalisation,
variant-adapted vaccines are probably needed to improve protection
against less severe endpoints, like emergency department admission,
especially for BA.2. Pfizer. Copyright © 2023 Elsevier Ltd. All rights
reserved. \| \|Hawai’i journal of health & social welfare \|Increasing
numbers of medical students participate in international electives.
However, this recent trend has yet to be examined in non-Western
high-income countries such as Japan. The aim of this study is to assess
recent trends in Japan, and to suggest ways in which those trends might
be influenced. A retrospective cross-sectional analysis of responses to
an 8-item questionnaire sent in August 2019 to 82 medical schools in
Japan is reported. The responses were received in September 2019.
Narrative responses were obtained regarding rationales for exchange
programs, participant feedback, and challenges encountered. Responses
were translated into English and categorized into themes. Of 82 Japanese
medical schools, 56 (68%) responded to the questionnaire. Both the
number of incoming and outgoing exchange students had increased steadily
over the preceding 3-year period. The leading destinations for Japanese
students were the United States (30%), other Asian (36%), and European
countries (24%). Narrative responses reveal different rationales from
those reported by medical schools in Western high-income countries. Only
a few Japanese students chose low or middle-income countries as their
destinations, as opposed to the trend seen in Western high-income
countries. The reported challenges encountered by the exchange programs
may provide insights for improvement. Exchanges have been greatly
affected by the coronavirus disease 2019 pandemic. The results can serve
as pre-pandemic baseline data and should promote further international
collaboration for medical education under current circumstances.
©Copyright 2022 by University Health Partners of Hawai‘i (UHP Hawai‘i).
\| \|Epilepsia <ISOAbbreviation>Epilepsia</ISOAbbreviation> </Journal>
<ArticleTitle>Fenfluramine provides clinically meaningful reduction in
frequency of drop seizures in patients with Lennox-Gastaut syndrome:
Interim analysis of an open-label extension study.</ArticleTitle>
<Pagination> <StartPage>139</StartPage> <EndPage>151</EndPage>
<MedlinePgn>139-151</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1111/epi.17431</ELocationID>
<Abstract> <AbstractText Label="OBJECTIVE" NlmCategory="OBJECTIVE">This
study was undertaken to evaluate the long-term safety and effectiveness
of fenfluramine in patients with Lennox-Gastaut syndrome
(LGS).</AbstractText>
<AbstractText Label="METHODS" NlmCategory="METHODS">Eligible patients
with LGS who completed a 14-week phase 3 randomized clinical trial
enrolled in an open-label extension (OLE; NCT03355209). All patients
were initially started on .2 mg/kg/day fenfluramine and after 1 month
were titrated by effectiveness and tolerability, which were assessed at
3-month intervals. The protocol-specified treatment duration was
12 months, but COVID-19-related delays resulted in 142 patients
completing their final visit after 12 months.</AbstractText>
<AbstractText Label="RESULTS" NlmCategory="RESULTS">As of October 19,
2020, 247 patients were enrolled in the OLE. Mean age was
14.3 ± 7.6 years (79 \[32%\] adults) and median fenfluramine treatment
duration was 364 days; 88.3% of patients received 2-4 concomitant
antiseizure medications. Median percentage change in monthly drop
seizure frequency was -28.6% over the entire OLE (n = 241) and -50.5% at
Month 15 (n = 142, p \< .0001); 75 of 241 patients (31.1%) experienced
≥50% reduction in drop seizure frequency. Median percentage change in
nondrop seizure frequency was -45.9% (n = 192, p = .0038). Generalized
tonic-clonic seizures (GTCS) and tonic seizures were most responsive to
treatment, with median reductions over the entire OLE of 48.8%
(p \< .0001, n = 106) and 35.8% (p \< .0001, n = 186), respectively. A
total of 37.6% (95% confidence interval \[CI\] = 31.4%-44.1%, n = 237)
of investigators and 35.2% of caregivers (95% CI = 29.1%-41.8%, n = 230)
rated patients as Much Improved/Very Much Improved on the Clinical
Global Impression of Improvement scale. The most frequent
treatment-emergent adverse events were decreased appetite (16.2%) and
fatigue (13.4%). No cases of valvular heart disease (VHD) or pulmonary
arterial hypertension (PAH) were observed.</AbstractText>
<AbstractText Label="SIGNIFICANCE" NlmCategory="CONCLUSIONS">Patients
with LGS experienced sustained reductions in drop seizure frequency on
fenfluramine treatment, with a particularly robust reduction in
frequency of GTCS, the key risk factor for sudden unexpected death in
epilepsy. Fenfluramine was generally well tolerated; VHD or PAH was not
observed long-term. Fenfluramine may provide an important long-term
treatment option for LGS.</AbstractText> <CopyrightInformation>© 2022
UCB and The Authors. Epilepsia published by Wiley Periodicals LLC on
behalf of International League Against Epilepsy.</CopyrightInformation>
</Abstract> <AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Knupp</LastName> <ForeName>Kelly G</ForeName>
<Initials>KG</Initials>
<Identifier Source="ORCID">0000-0002-1967-0827</Identifier>
<AffiliationInfo> <Affiliation>University of Colorado, Children’s
Hospital Colorado, Aurora, Colorado, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Scheffer</LastName> <ForeName>Ingrid E</ForeName>
<Initials>IE</Initials>
<Identifier Source="ORCID">0000-0002-2311-2174</Identifier>
<AffiliationInfo> <Affiliation>University of Melbourne, Austin Hospital
and Royal Children’s Hospital, Melbourne, Victoria,
Australia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Ceulemans</LastName>
<ForeName>Berten</ForeName> <Initials>B</Initials> <AffiliationInfo>
<Affiliation>Department of Pediatric Neurology, Antwerp University
Hospital, Antwerp, Belgium.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Sullivan</LastName>
<ForeName>Joseph</ForeName> <Initials>J</Initials>
<Identifier Source="ORCID">0000-0003-2081-8988</Identifier>
<AffiliationInfo> <Affiliation>University of California, San Francisco
Weill Institute for Neurosciences, Benioff Children’s Hospital, San
Francisco, California, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Nickels</LastName> <ForeName>Katherine
C</ForeName> <Initials>KC</Initials>
<Identifier Source="ORCID">0000-0002-6579-3035</Identifier>
<AffiliationInfo> <Affiliation>Department of Neurology, Mayo Clinic,
Rochester, Minnesota, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Lagae</LastName>
<ForeName>Lieven</ForeName> <Initials>L</Initials>
<Identifier Source="ORCID">0000-0002-7118-0139</Identifier>
<AffiliationInfo> <Affiliation>Member of the European Reference Network
EpiCARE, Department of Pediatric Neurology, University of Leuven,
Leuven, Belgium.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Guerrini</LastName>
<ForeName>Renzo</ForeName> <Initials>R</Initials>
<Identifier Source="ORCID">0000-0002-7272-7079</Identifier>
<AffiliationInfo> <Affiliation>Pediatric Neurology and Neurogenetics
Unit, Anna Meyer Children’s Hospital, University of Florence, Florence,
Italy.</Affiliation> </AffiliationInfo> <AffiliationInfo>
<Affiliation>Stella Maris Foundation, Scientific Institute for Research
and Health Care, Pisa, Italy.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Zuberi</LastName> <ForeName>Sameer
M</ForeName> <Initials>SM</Initials> <AffiliationInfo>
<Affiliation>Paediatric Neurosciences Research Group, Royal Hospital for
Children, Glasgow, UK.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Nabbout</LastName>
<ForeName>Rima</ForeName> <Initials>R</Initials>
<Identifier Source="ORCID">0000-0001-5877-4074</Identifier>
<AffiliationInfo> <Affiliation>Reference Center for Rare Epilepsies,
Necker-Sick Children University Hospital, Public Hospital Network of
Paris, member of EpiCARE, Imagine Institute, Paris Cité University,
Paris, France.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Riney</LastName>
<ForeName>Kate</ForeName> <Initials>K</Initials>
<Identifier Source="ORCID">0000-0002-1122-3555</Identifier>
<AffiliationInfo> <Affiliation>Neuroscience Unit, Queensland Children’s
Hospital, South Brisbane, Queensland, Australia.</Affiliation>
</AffiliationInfo> <AffiliationInfo> <Affiliation>School of Clinical
Medicine, University of Queensland, St Lucia, Queensland,
Australia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Agarwal</LastName>
<ForeName>Anupam</ForeName> <Initials>A</Initials> <AffiliationInfo>
<Affiliation>Zogenix (now a part of UCB), Emeryville, California,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Lock</LastName> <ForeName>Michael</ForeName>
<Initials>M</Initials> <AffiliationInfo> <Affiliation>Independent
Consultant, Zogenix (now a part of UCB), Haiku, Hawaii,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Dai</LastName> <ForeName>David</ForeName>
<Initials>D</Initials> <AffiliationInfo> <Affiliation>Syneos Health,
Morrisville, North Carolina, USA.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Farfel</LastName>
<ForeName>Gail M</ForeName> <Initials>GM</Initials> <AffiliationInfo>
<Affiliation>Zogenix (now a part of UCB), Emeryville, California,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Galer</LastName> <ForeName>Bradley S</ForeName>
<Initials>BS</Initials>
<Identifier Source="ORCID">0000-0002-4735-3327</Identifier>
<AffiliationInfo> <Affiliation>Zogenix (now a part of UCB), Emeryville,
California, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Gammaitoni</LastName> <ForeName>Arnold
R</ForeName> <Initials>AR</Initials>
<Identifier Source="ORCID">0000-0002-4775-2862</Identifier>
<AffiliationInfo> <Affiliation>Zogenix (now a part of UCB), Emeryville,
California, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Polega</LastName>
<ForeName>Shikha</ForeName> <Initials>S</Initials> <AffiliationInfo>
<Affiliation>Zogenix (now a part of UCB), Emeryville, California,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Davis</LastName> <ForeName>Ronald</ForeName>
<Initials>R</Initials> <AffiliationInfo> <Affiliation>Neurology and
Epilepsy Research Center, Orlando, Florida, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Gil-Nagel</LastName> <ForeName>Antonio</ForeName>
<Initials>A</Initials> <AffiliationInfo> <Affiliation>Ruber
International Hospital, Madrid, Spain.</Affiliation> </AffiliationInfo>
</Author> </AuthorList> <Language>eng</Language> <PublicationTypeList>
<PublicationType UI="D016449">Randomized Controlled
Trial</PublicationType> <PublicationType UI="D017428">Clinical Trial,
Phase III</PublicationType> <PublicationType UI="D016428">Journal
Article</PublicationType> </PublicationTypeList>
<ArticleDate DateType="Electronic"> <Year>2022</Year> <Month>11</Month>
<Day>09</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>United States</Country>
      <MedlineTA>Epilepsia</MedlineTA>
      <NlmUniqueID>2983306R</NlmUniqueID>
      <ISSNLinking>0013-9580</ISSNLinking>
    </MedlineJournalInfo>
    <ChemicalList>
      <Chemical>
        <RegistryNumber>0</RegistryNumber>
        <NameOfSubstance UI="D000927">Anticonvulsants</NameOfSubstance>
      </Chemical>
      <Chemical>
        <RegistryNumber>2DS058H2CF</RegistryNumber>
        <NameOfSubstance UI="D005277">Fenfluramine</NameOfSubstance>
      </Chemical>
    </ChemicalList>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000328" MajorTopicYN="N">Adult</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002648" MajorTopicYN="N">Child</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000293" MajorTopicYN="N">Adolescent</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D055815" MajorTopicYN="N">Young Adult</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D065768" MajorTopicYN="Y">Lennox Gastaut Syndrome</DescriptorName>
        <QualifierName UI="Q000188" MajorTopicYN="N">drug therapy</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000927" MajorTopicYN="N">Anticonvulsants</DescriptorName>
        <QualifierName UI="Q000627" MajorTopicYN="N">therapeutic use</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005277" MajorTopicYN="N">Fenfluramine</DescriptorName>
        <QualifierName UI="Q000627" MajorTopicYN="N">therapeutic use</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D016896" MajorTopicYN="N">Treatment Outcome</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="Y">COVID-19</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D012640" MajorTopicYN="N">Seizures</DescriptorName>
        <QualifierName UI="Q000188" MajorTopicYN="N">drug therapy</QualifierName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">Lennox-Gastaut syndrome</Keyword>
      <Keyword MajorTopicYN="N">developmental and epileptic encephalopathies</Keyword>
      <Keyword MajorTopicYN="N">fenfluramine</Keyword>
      <Keyword MajorTopicYN="N">long-term open-label extension</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="revised"> <Year>2022</Year> <Month>9</Month>
<Day>30</Day> </PubMedPubDate> <PubMedPubDate PubStatus="received">
<Year>2022</Year> <Month>6</Month> <Day>15</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="accepted"> <Year>2022</Year> <Month>10</Month>
<Day>3</Day> </PubMedPubDate> <PubMedPubDate PubStatus="pubmed">
<Year>2022</Year> <Month>10</Month> <Day>6</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="medline">
<Year>2023</Year> <Month>1</Month> <Day>21</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="entrez">
<Year>2022</Year> <Month>10</Month> <Day>5</Day> <Hour>5</Hour>
<Minute>43</Minute> </PubMedPubDate> </History>
<PublicationStatus>ppublish</PublicationStatus> <ArticleIdList>
<ArticleId IdType="pubmed">36196777</ArticleId>
<ArticleId IdType="doi">10.1111/epi.17431</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|This study was undertaken to evaluate the
long-term safety and effectiveness of fenfluramine in patients with
Lennox-Gastaut syndrome (LGS). Eligible patients with LGS who completed
a 14-week phase 3 randomized clinical trial enrolled in an open-label
extension (OLE; NCT03355209). All patients were initially started on .2
mg/kg/day fenfluramine and after 1 month were titrated by effectiveness
and tolerability, which were assessed at 3-month intervals. The
protocol-specified treatment duration was 12 months, but
COVID-19-related delays resulted in 142 patients completing their final
visit after 12 months. As of October 19, 2020, 247 patients were
enrolled in the OLE. Mean age was 14.3 ± 7.6 years (79 \[32%\] adults)
and median fenfluramine treatment duration was 364 days; 88.3% of
patients received 2-4 concomitant antiseizure medications. Median
percentage change in monthly drop seizure frequency was -28.6% over the
entire OLE (n = 241) and -50.5% at Month 15 (n = 142, p \< .0001); 75 of
241 patients (31.1%) experienced ≥50% reduction in drop seizure
frequency. Median percentage change in nondrop seizure frequency was
-45.9% (n = 192, p = .0038). Generalized tonic-clonic seizures (GTCS)
and tonic seizures were most responsive to treatment, with median
reductions over the entire OLE of 48.8% (p \< .0001, n = 106) and 35.8%
(p \< .0001, n = 186), respectively. A total of 37.6% (95% confidence
interval \[CI\] = 31.4%-44.1%, n = 237) of investigators and 35.2% of
caregivers (95% CI = 29.1%-41.8%, n = 230) rated patients as Much
Improved/Very Much Improved on the Clinical Global Impression of
Improvement scale. The most frequent treatment-emergent adverse events
were decreased appetite (16.2%) and fatigue (13.4%). No cases of
valvular heart disease (VHD) or pulmonary arterial hypertension (PAH)
were observed. Patients with LGS experienced sustained reductions in
drop seizure frequency on fenfluramine treatment, with a particularly
robust reduction in frequency of GTCS, the key risk factor for sudden
unexpected death in epilepsy. Fenfluramine was generally well tolerated;
VHD or PAH was not observed long-term. Fenfluramine may provide an
important long-term treatment option for LGS. © 2022 UCB and The
Authors. Epilepsia published by Wiley Periodicals LLC on behalf of
International League Against Epilepsy. \| \|Health equity \|COVID-19
disproportionately affects racial/ethnic minorities and vaccine can help
mitigate infection and transition, decrease rate of hospitalization,
lower mortality rate, and control the pandemic. This study aims to
examine disparities in COVID-19 vaccination rate by age among Whites,
Hispanics, Blacks, and Asian Americans, and the modification effects by
gender and education. We used seven waves of biweekly surveys from the
Household Pulse Survey collected between July 21, 2021, and October 11,
2021. Asians reported the highest, Blacks reported the lowest
vaccination rate, and gender differences were minimal. Increasing age
was associated with higher vaccination rate except for the oldest age
group. The decline was from 84.4% (70-79 years) to 41.1% (80-88 years:
41.1%) among Hispanics and 92.8% to 69.6% among Asians. Educational
effect was the most salient among younger adults with the largest gaps
observed in Blacks. Among 18-29-year Black participants, the vaccination
rates were 31.1% (confidence interval \[95% CI\]: 25.7-37.1) for high
school or lower, 58.9% (95% CI: 54.2-63.5) for some college or associate
degree, and 74.2% (95% CI: 69.4-78.5) for bachelor or higher degrees,
leaving a 43.1% gap between the lowest and the highest education levels.
The gaps in this age group were 33.7% among Whites, 32.1% among
Hispanics, and 20.5% among Asian Americans. Our study advances the
existing literature on COVID-19 vaccination by providing empirical
evidence on the dynamic race/ethnic-age-education differences across
racial/ethnic groups. The findings from our study provide scientific
foundation for the development of more strategies to improve vaccination
rate for the minority populations. © Wei Zhang et al., 2022; Published
by Mary Ann Liebert, Inc. \| \|JAMA network open \|NA \| \|PloS one
\|Results from observational studies and randomized clinical trials
(RCTs) have led to the consensus that hydroxychloroquine (HCQ) and
chloroquine (CQ) are not effective for COVID-19 prevention or treatment.
Pooling individual participant data, including unanalyzed data from
trials terminated early, enables more detailed investigation of the
efficacy and safety of HCQ/CQ among subgroups of hospitalized patients.
We searched ClinicalTrials.gov in May and June 2020 for US-based RCTs
evaluating HCQ/CQ in hospitalized COVID-19 patients in which the
outcomes defined in this study were recorded or could be extrapolated.
The primary outcome was a 7-point ordinal scale measured between day 28
and 35 post enrollment; comparisons used proportional odds ratios.
Harmonized de-identified data were collected via a common template
spreadsheet sent to each principal investigator. The data were analyzed
by fitting a prespecified Bayesian ordinal regression model and
standardizing the resulting predictions. Eight of 19 trials met
eligibility criteria and agreed to participate. Patient-level data were
available from 770 participants (412 HCQ/CQ vs 358 control). Baseline
characteristics were similar between groups. We did not find evidence of
a difference in COVID-19 ordinal scores between days 28 and 35
post-enrollment in the pooled patient population (odds ratio, 0.97; 95%
credible interval, 0.76-1.24; higher favors HCQ/CQ), and found no
convincing evidence of meaningful treatment effect heterogeneity among
prespecified subgroups. Adverse event and serious adverse event rates
were numerically higher with HCQ/CQ vs control (0.39 vs 0.29 and 0.13 vs
0.09 per patient, respectively). The findings of this individual
participant data meta-analysis reinforce those of individual RCTs that
HCQ/CQ is not efficacious for treatment of COVID-19 in hospitalized
patients. \| \|Intensive care medicine \|Sepsis is a heterogeneous
syndrome and identification of sub-phenotypes is essential. This study
used trajectories of vital signs to develop and validate sub-phenotypes
and investigated the interaction of sub-phenotypes with treatment using
randomized controlled trial data. All patients with suspected infection
admitted to four academic hospitals in Emory Healthcare between
2014-2017 (training cohort) and 2018-2019 (validation cohort) were
included. Group-based trajectory modeling was applied to vital signs
from the first 8 h of hospitalization to develop and validate vitals
trajectory sub-phenotypes. The associations between sub-phenotypes and
outcomes were evaluated in patients with sepsis. The interaction between
sub-phenotype and treatment with balanced crystalloids versus saline was
tested in a secondary analysis of SMART (Isotonic Solutions and Major
Adverse Renal Events Trial). There were 12,473 patients with suspected
infection in training and 8256 patients in validation cohorts, and 4
vitals trajectory sub-phenotypes were found. Group A (N = 3483, 28%)
were hyperthermic, tachycardic, tachypneic, and hypotensive. Group B (N
= 1578, 13%) were hyperthermic, tachycardic, tachypneic (not as
pronounced as Group A) and hypertensive. Groups C (N = 4044, 32%) and D
(N = 3368, 27%) had lower temperatures, heart rates, and respiratory
rates, with Group C normotensive and Group D hypotensive. In the 6,919
patients with sepsis, Groups A and B were younger while Groups C and D
were older. Group A had the lowest prevalence of congestive heart
failure, hypertension, diabetes mellitus, and chronic kidney disease,
while Group B had the highest prevalence. Groups A and D had the highest
vasopressor use (p \< 0.001 for all analyses above). In logistic
regression, 30-day mortality was significantly higher in Groups A and D
(p \< 0.001 and p = 0.03, respectively). In the SMART trial,
sub-phenotype significantly modified treatment effect (p = 0.03). Group
D had significantly lower odds of mortality with balanced crystalloids
compared to saline (odds ratio (OR) 0.39, 95% confidence interval (CI)
0.23-0.67, p \< 0.001). Sepsis sub-phenotypes based on vital sign
trajectory were consistent across cohorts, had distinct outcomes, and
different responses to treatment with balanced crystalloids versus
saline. © 2022. Springer-Verlag GmbH Germany, part of Springer Nature.
\| \|Vaccines \|Vaccine hesitancy remains a significant barrier to
achieving herd immunity and preventing the further spread of COVID-19.
Understanding contributors to vaccine hesitancy and how they change over
time may improve COVID-19 mitigation strategies and public health
policies. To date, no mechanism explains how trust in and consumption of
different sources of information affect vaccine uptake. A total of 1594
adults enrolled in our COVID-19 testing program completed standardized
surveys on demographics, vaccination status, use, reliance, and trust in
sources of COVID-19 information, from September to October 2021, during
the COVID-19 Delta wave. Of those, 802 individuals (50.3%) completed a
follow-up survey, from January to February 2022, during the
Omicron-wave. Regression analyses were performed to understand
contributors to vaccine and booster uptake over time. Individuals
vaccinated within two months of eligibility (early vaccinees) tended to
have more years of schooling, with greater trust in and consumption of
official sources of COVID-19 information, compared to those who waited
3-6 months (late vaccinees), or those who remained unvaccinated at 6
months post-eligibility (non-vaccinees). Most (70.1%) early vaccinees
took the booster shot, compared to only 30.5% of late vaccinees, with
the latter group gaining trust and consumption of official information
after four months. These data provide the foundation for a mechanism
based on the level of trust in and consumption of official information
sources, where those who increased their level of trust in and
consumption of official information sources were more likely to receive
a booster. This study shows that social factors, including education and
individual-level degree of trust in (and consumption of) sources of
COVID-19 information, interact and change over time to be associated
with vaccine and booster uptakes. These results are critical for the
development of effective public health policies and offer insights into
hesitancy over the course of the COVID-19 vaccine and booster rollout.
\| \|The Journal of school nursing : the official publication of the
National Association of School Nurses \|This mixed-method study examined
school nurses’ experiences during the Coronavirus Disease 2019 pandemic
related to role change, psychological feelings, and coping/resiliency in
the State of Hawaii. A total of 30 school nurses completed a Brief
Resilience Coping Scale plus a series of open-ended questions in January
2022. On the coping scale, over 40% of participants scored high, 52%
scored medium, and 7% scored a low resilient/coping level. We did not
identify any association between coping level and participant
characteristics. Three qualitative themes emerged: 1) school nurses
experience chronic negative emotions related to the pandemic, 2) school
nurses demonstrate attributes of resilience, and 3) school nurses
utilize positive coping techniques. The pandemic created significant
stresses and negative emotions among school nurses. Yet, school nurses
reported effective coping strategies and demonstrated
strength/resilience. Support and open communication between school
nurses, their employers, and other school-based stakeholders is needed
to provide continued support for school nurses. \| \|Medical care
\|Since the onset of the COVID-19 pandemic, telehealth has been an
option for Veterans receiving urgent care through Veterans Health
Administration Community Care (CC). We assessed use, arrangements,
Veteran decision-making, and experiences with CC urgent care delivered
via telehealth. Convergent parallel mixed methods, combining
multivariable regression analyses of claims data with semistructured
Veteran interviews. Veterans residing in the Western United States and
Hawaii, with CC urgent care claims March 1 to September 30, 2020. In
comparison to having in-person only visits, having a telehealth-only
visit was more likely for Veterans who were non-Hispanic Black, were
urban-dwelling, lived further from the clinic used, had a COVID-related
visit, and did not require an in-person procedure. Predictors of having
both telehealth and in-person (compared with in-person only) visits were
other (non-White, non-Black) non-Hispanic race/ethnicity, urban-dwelling
status, living further from the clinic used, and having had a
COVID-related visit. Care arrangements varied widely; telephone-only
care was common. Veteran decisions about using telehealth were driven by
limitations in in-person care availability and COVID-related concerns.
Veterans receiving care via telehealth generally reported high
satisfaction. CC urgent care via telehealth played an important role in
providing Veterans with care access early in the COVID-19 pandemic. Use
of telehealth differed by Veteran characteristics; lack of in-person
care availability was a driver. Future work should assess for changes in
telehealth use with pandemic progression, geographic differences, and
impact on care quality, care coordination, outcomes, and costs to ensure
Veterans’ optimal and equitable access to care. \| \|Public health
reports (Washington, D.C. : 1974) \|Minimal research has assessed
COVID-19’s unique impact on the Native Hawaiian/Pacific Islander (NH/PI)
population-an Indigenous-colonized racial group with social and health
disparities that increase their risk for COVID-19 morbidity and
mortality. To address this gap, we explored the scope of COVID-19
outcomes, vaccination status, and health in diverse NH/PI communities.
NH/PI staff at partner organizations collected survey data from April
through November 2021 from 319 community-dwelling NH/PI adults in 5
states with large NH/PI populations: Arkansas, California, Oregon, Utah,
and Washington. Data were analyzed with descriptive statistics, Pearson
χ2 tests, independent and paired t tests, and linear and logistic
regression analyses. During the COVID-19 pandemic, 30% of survey
participants had contracted COVID-19, 16% had a close family member who
died of the disease, and 64% reported COVID-19 vaccine uptake. Thirty
percent reported fair/poor health, 21% currently smoked cigarettes, and
58% reported obesity. Survey participants reported heightened
COVID-19-related psychosocial distress (mean score = 4.9 on 10-point
scale), which was more likely when health outcomes (general health,
sleep, obesity) were poor or a family member had died of COVID-19.
Logistic regression indicated that age, experiencing COVID-19 distress,
and past-year use of influenza vaccines were associated with higher odds
of COVID-19 vaccine uptake (1.06, 1.18, and 7.58 times, respectively).
Our empirical findings highlight the acute and understudied negative
impact of COVID-19 on NH/PI communities in the United States and suggest
new avenues for improving NH/PI community health, vaccination, and
recovery from COVID-19. \| \|American journal of public health \|Native
Hawaiians and other Pacific Islanders (NHPIs) across the country have
experienced significant disparities because of the COVID-19 pandemic.
The Pacific Alliance Against COVID-19 used a community-based
participatory approach involving academic and community partners to
expand sustainable COVID-19 testing capacity and mitigate the severe
consequences among NHPI communities in Hawaii. We describe the approach
of this one-year study, some of the results, and how the data are being
used to inform next steps for the communities. Clinical Trials.gov
identifier: NCT04766333. (Am J Public Health. 2022;112(S9):S896-S899.
<https://doi.org/10.2105/AJPH.2022.306973>). \| \|CABI agriculture and
bioscience \|The COVID-19 pandemic is interrupting domestic and global
food supply chains resulting in reduced access to healthy diverse diets.
Hawai’i has been described as a model social-ecological system and it
has been suggested that indigenous agro-ecosystems have the potential to
be highly productive and resilient under changing land-use and climate
change disturbance. However, little research has yet been conducted
exploring the disruption and resilience of agro-ecosystems in Hawai’i
caused by the COVID-19 pandemic. The breadfruit tree (Artocarpus
altilis; Moraceae) is a signature, multi-purpose-tree of the complex
perennial agro-ecosystems systems in Oceania. This case study explores
the ways in which the breadfruit agro-ecosystems of Hawai’i have shown
resilience during the COVID-19 pandemic. Our study suggests that
breadfruit has increased its value as a subsistence crop during the
COVID-19 pandemic, even in a developed economy like Hawai’i, and that
resilience of Hawaiian breadfruit agroe-cosystems during a crisis can be
supported through cooperatives and food-hubs. The online version
contains supplementary material available at 10.1186/s43170-022-00125-3.
© The Author(s) 2022. \| \|Orthopaedic journal of sports medicine
\|Health and safety concerns surrounding the coronavirus 2019 (COVID-19)
pandemic led the National Basketball Association (NBA) to condense and
accelerate the 2020 season. Although prior literature has suggested that
inadequate rest may lead to an increased injury risk, the unique
circumstances surrounding this season offer a unique opportunity to
evaluate player safety in the setting of reduced interval rest. We
hypothesized that the condensed 2020 NBA season resulted in an increased
overall injury risk as compared with the 2015 to 2018 seasons.
Descriptive epidemiology study. A publicly available database, Pro
Sports Transactions, was queried for injuries that forced players to
miss ≥1 game between the 2015 and 2020 seasons. Data from the 2019
season were omitted given the abrupt suspension of the league year. All
injury incidences were calculated per 1000 game-exposures (GEs). The
primary outcome was the overall injury proportion ratio (IPR) between
the 2020 season and previous seasons. Secondary measures included injury
incidences stratified by type, severity, age, position, and minutes per
game. A total of 4346 injuries occurred over a 5-season span among 2572
unique player-seasons. The overall incidence of injury during the 2020
season was 48.20 per 1000 GEs but decreased to 39.97 per 1000 GEs when
excluding COVID-19. Despite this exclusion, the overall injury rate in
2020 remained significantly greater (IPR, 1.42 \[95% CI, 1.32-1.52\])
than that of the 2015 to 2018 seasons (28.20 per 1000 GEs). On closer
evaluation, the most notable increases seen in the 2020 season occurred
within minor injuries requiring only a 1-game absence (IPR, 1.53 \[95%
CI, 1.37-1.70\]) and in players who were aged 25 to 29 years (IPR, 1.57
\[95% CI, 1.40-2.63\]), averaging ≥30.0 minutes per game (IPR, 1.67
\[95% CI, 1.47-1.90\]), and playing the point guard position (IPR, 1.67
\[95% 1.44-1.95\]). Players in the condensed 2020 NBA season had a
significantly higher incidence of injuries when compared with the prior
4 seasons, even when excluding COVID-19-related absences. This rise is
consistent with the other congested NBA seasons of 1998 and 2011. These
findings suggest that condensing the NBA schedule is associated with an
increased risk to player health and safety. © The Author(s) 2022. \|
\|Journal of traumatic stress \|Trauma-exposed veterans receiving mental
health care may have an elevated risk of experiencing COVID-19-related
difficulties. Using data from several ongoing clinical trials (N = 458),
this study examined exposure to COVID-19-related stressors and their
associations with key sociodemographic factors and mental health
outcomes. The results showed that exposure to COVID-19-related stressors
was common, higher among veterans who were racial/ethnic minorities d =
0.32, and associated with elevated posttraumatic stress disorder (PTSD),
r = .288, and depressive symptom severity, r = .246. Women veterans
experienced more difficulty accessing social support, d = 0.31, and
higher levels of COVID-19-related distress, d = 0.31, than men.
Qualitative data were consistent with survey findings and highlighted
the broader societal context in veterans’ experience of COVID-19-related
distress. These findings may inform future research on the impact of the
pandemic on veterans, particularly those who are women and members of
minoritized racial/ethnic groups, as well as mental health treatment
planning for this population. © 2022 International Society for Traumatic
Stress Studies. \| \|The journal of education in perioperative medicine
: JEPM \|This study’s primary aim was to determine how training programs
use simulation-based medical education (SBME), because SBME is linked to
superior clinical performance. An anonymous 10-question survey was
distributed to anesthesiology residency program directors across the
United States. The survey aimed to assess where and how SBME takes
place, which resources are available, frequency of and barriers to its
use, and perceived utility of a dedicated departmental education
laboratory. The survey response rate was 30.4% (45/148). SBME typically
occurred at shared on-campus laboratories, with residents typically
participating in SBME 1 to 4 times per year. Frequently practiced skills
included airway management, trauma scenarios, nontechnical skills, and
ultrasound techniques (all ≥ 77.8%). Frequently cited logistical
barriers to simulation laboratory use included COVID-19 precautions
(75.6%), scheduling (57.8%), and lack of trainers (48.9%). Several
respondents also acknowledged financial barriers. Most respondents
believed a dedicated departmental education laboratory would be a useful
or very useful resource (77.8%). SBME is a widely incorporated activity
but may be impeded by barriers that our survey helped identify. Barriers
can be addressed by departmental education laboratories. We discuss how
such laboratories increase capabilities to support structured SBME
events and how costs can be offset. Other academic departments may also
benefit from establishing such laboratories. \| \|The journal of
physical chemistry letters \|Pulmonary surfactant has been attempted as
a supportive therapy to treat COVID-19. Although it is mechanistically
accepted that the fusion peptide in the S2 subunit of the S protein
plays a predominant role in mediating viral fusion with the host cell
membrane, it is still unknown how the S2 subunit interacts with the
natural surfactant film. Using combined bio-physicochemical assays and
atomic force microscopy imaging, it was found that the S2 subunit
inhibited the biophysical properties of the surfactant and induced
microdomain fusion in the surfactant monolayer. The surfactant
inhibition has been attributed to membrane fluidization caused by
insertion of the S2 subunit mediated by its fusion peptide. These
findings may provide novel insight into the understanding of
bio-physicochemical mechanisms responsible for surfactant interactions
with SARS-CoV-2 and may have translational implications in the further
development of surfactant replacement therapy for COVID-19 patients. \|
\|JAMA network open \|Widespread distribution of rapid antigen tests is
integral to the US strategy to address COVID-19; however, it is
estimated that few rapid antigen test results are reported to local
departments of health. To characterize how often individuals in 6
communities throughout the United States used a digital assistant to log
rapid antigen test results and report them to their local departments of
health. This prospective cohort study is based on anonymously collected
data from the beneficiaries of the Say Yes! Covid Test program, which
distributed more than 3 000 000 rapid antigen tests at no cost to
residents of 6 communities (Louisville, Kentucky; Indianapolis, Indiana;
Fulton County, Georgia; O’ahu, Hawaii; Ann Arbor and Ypsilanti,
Michigan; and Chattanooga, Tennessee) between April and October 2021. A
descriptive evaluation of beneficiary use of a digital assistant for
logging and reporting their rapid antigen test results was performed.
Widespread community distribution of rapid antigen tests. Number and
proportion of tests logged and reported to the local department of
health through the digital assistant. A total of 313 000 test kits were
distributed, including 178 785 test kits that were ordered using the
digital assistant. Among all distributed kits, 14 398 households (4.6%)
used the digital assistant, but beneficiaries reported three-quarters of
their rapid antigen test results to their state public health
departments (30 965 tests reported of 41 465 total test results
\[75.0%\]). The reporting behavior varied by community and was
significantly higher among communities that were incentivized for
reporting test results vs those that were not incentivized or partially
incentivized (90.5% \[95% CI, 89.9%-91.2%\] vs 70.5%; \[95% CI,
70.0%-71.0%\]). In all communities, positive tests were less frequently
reported than negative tests (60.4% \[95% CI, 58.1%-62.8%\] vs 75.5%
\[95% CI, 75.1%-76.0%\]). These results suggest that application-based
reporting with incentives may be associated with increased reporting of
rapid tests for COVID-19. However, increasing the adoption of the
digital assistant may be a critical first step. \| \|JAMA network open
\|The 2 primary efforts of Medicare to advance value-based care are
Medicare Advantage (MA) and the fee-for-service-based Medicare Shared
Savings Program (MSSP). It is unknown how spending differs between the 2
programs after accounting for differences in patient clinical risk. To
examine how spending and utilization differ between MA and MSSP
beneficiaries after accounting for differences in clinical risk using
data from administrative claims and electronic health records. This
retrospective economic evaluation used data from 15 763 propensity
score-matched beneficiaries who were continuously enrolled in MA or MSSP
from January 1, 2014, to December 31, 2018, with diabetes, congestive
heart failure (CHF), chronic kidney disease (CKD), or hypertension.
Participants received care at a large nonprofit academic health system
in the southern United States that bears risk for Medicare beneficiaries
through both the MA and MSSP programs. Differences in beneficiary risk
were mitigated by propensity score matching using validated clinical
criteria based on data from administrative claims and electronic health
records. Data were analyzed from January 2019 to May 2022. Enrollment in
MA or attribution to an accountable care organization in the MSSP
program. Per-beneficiary annual total spending and subcomponents,
including inpatient hospital, outpatient hospital, skilled nursing
facility, emergency department, primary care, and specialist spending.
The sample of 15 763 participants included 12 720 (81%) MA and 3043
(19%) MSSP beneficiaries. MA beneficiaries, compared with MSSP
beneficiaries, were more likely to be older (median \[IQR\] age, 75.0
\[69.9-81.8\] years vs 73.1 \[68.3-79.8\] years), male (5515 \[43%\] vs
1119 \[37%\]), and White (9644 \[76%\] vs 2046 \[69%\]) and less likely
to live in low-income zip codes (2338 \[19%\] vs 750 \[25%\]). The mean
unadjusted per-member per-year spending difference between MSSP and MA
disease-specific subcohorts was \$2159 in diabetes, \$4074 in CHF,
\$2560 in CKD, and \$2330 in hypertension. After matching on clinical
risk and demographic factors, MSSP spending was higher for patients with
diabetes (mean per-member per-year spending difference in 2015: \$2454;
95% CI, \$1431-\$3574), CHF (\$3699; 95% CI, \$1235-\$6523), CKD
(\$2478; 95% CI, \$1172-\$3920), and hypertension (\$2258; 95% CI,
\$1616-2,939). Higher MSSP spending among matched beneficiaries was
consistent over time. In the matched cohort in 2018, MSSP total spending
ranged from 23% (CHF) to 30% (CKD) higher than MA. Adjusting for
differential trends in coding intensity did not affect these results.
Higher outpatient hospital spending among MSSP beneficiaries contributed
most to spending differences between MSSP and MA, representing 49% to
62% of spending differences across disease cohorts. In this study,
utilization and spending were consistently higher for MSSP than MA
beneficiaries within the same health system even after adjusting for
granular metrics of clinical risk. Nonclinical factors likely contribute
to the large differences in MA vs MSSP spending, which may create
challenges for health systems participating in MSSP relative to their
participation in MA. \| \|JAMA neurology \|NA \| \|International journal
of environmental research and public health \|In response to the second
surge of COVID-19 cases in Hawaii in the fall of 2020, the Hawaii State
Department of Health Behavioral Health Administration led and contracted
a coalition of agencies to plan and implement an isolation and
quarantine facility placement service that included food, testing, and
transportation assistance for a state capitol and major urban center.
The goal of the program was to provide safe isolation and quarantine
options for individual residents at risk of not being able to comply
with isolation and quarantine mandates. Drawing upon historical lived
experiences in planning and implementing the system for isolation and
quarantine facilities, this qualitative public health case study report
applies the plan-do-study-act (PDSA) improvement model and framework to
review and summarize the implementation of this system. This case study
also offers lessons for a unique opportunity for collaboration led by a
public behavioral health leadership that expands upon traditionally
narrow infectious disease control, by developing a continuum of care
that not only addresses immediate COVID-19 concerns but also longer-term
supports and services including housing, access to mental health
services, and other social services. This case study highlights the role
of a state agency in building a coalition of agencies, including a
public university, to respond to the pandemic. The case study also
discusses how continuous learning was executed to improve delivery of
care. \| \|Drug and alcohol review \|Before COVID-19, Native
Hawaiians/Pacific Islanders (NH/PI) endured a heavy burden of alcohol,
tobacco and other drug (ATOD) use in prior US data. Responding to
reports that many NH/PI communities experienced severe COVID-19
disparities that could exacerbate their ATOD burden, we partnered with
NH/PI communities to assess the substance use patterns and treatment
needs of diverse NH/PIs during COVID-19. Collaborating with NH/PI
community organisations across five states with large NH/PI populations,
we conducted a large-scale investigation of NH/PI ATOD use, mental
health and treatment need during COVID-19. Between April and November
2021, NH/PI-heritage research staff from our community partners
collected data involving 306 NH/PI adults using several community-based
recruitment methods (e-mail, telephone, in-person) and two survey
approaches: online and paper-and-pencil. Multivariate regressions were
conducted to examine potential predictors of NH/PI alcohol use disorder
and need for behavioural health treatment. During COVID-19, 47% and 22%
of NH/PI adults reported current alcohol and cigarette use, while 35%
reported lifetime illicit substance use (e.g., cannabis, opioid).
Depression and anxiety were high, and alcohol use disorder, major
depression and generalised anxiety disorder prevalence were 27%, 27% and
19%, respectively. One-third of participants reported past-year
treatment need with lifetime illicit substance use, COVID-19 distress
and major depression respectively associating with 3.0, 1.2, and 5.3
times greater adjusted odds for needing treatment. NH/PI adults reported
heavy ATOD use, depression, anxiety and treatment need during COVID-19.
Targeted research and treatment services may be warranted to mitigate
COVID-19’s negative behavioural health impact on NH/PI communities. ©
2022 The Authors. Drug and Alcohol Review published by John Wiley & Sons
Australia, Ltd on behalf of Australasian Professional Society on Alcohol
and other Drugs. \| \|Medical journal (Fort Sam Houston, Tex.) \|Since
March of 2020, thousands of National Guard service members have played a
key role in the domestic response to COVID-19, ranging from medical
support, health screening, decontamination, personal protective
equipment (PPE) training, and more. As a result of these missions, there
was a hypothesized potential increase in COVID-19 exposure risk. Assess
COVID-19 transmission rates and mortality rates in the US population
compared to the National Guard. Six months of retrospective data were
assessed with analysis of a snapshot in time for pandemic data on 29
July 2020. Potential relationships between National Guard COVID-19
response personnel, cumulative US COVID-19 cases, National Guard
COVID-19 cases, and National Guard COVID-19 fatalities were assessed. No
evidence of correlations exist between the number of National Guard
personnel supporting the COVID-19 response and the number of deaths in
the National Guard due to COVID-19 (p=0.547), and the number of National
Guard COVID-19 cases and the number of deaths in the National Guard due
to COVID-19 (p=0.214). The number of COVID-19 cases in the US was
positively correlated to the number of deaths in the US due to COVID-19
(rs=0.947, p is less than.001). Though much of the data could not be
reported due to operational security (OPSEC) and capabilities,
activities, limitations, and intentions (CALI) concerns, the data herein
demonstrate National Guard service members are significantly less likely
to suffer COVID-19 related mortality compared to US civilians. Since the
National Guard adheres the same medical and physical fitness standards
as set by their parent service (Army and Air Force), it follows overall
levels of medical readiness and fitness should start with a higher
baseline. Age, medical screening, PPE, and physical fitness requirements
have likely contributed to this phenomenon. These results should empower
National Guard service members to feel more confident in their roles as
they continue to support the COVID-19 response efforts. \| \|PloS one
\|Venous phlebotomy performed by trained personnel is critical for
patient diagnosis and monitoring of chronic disease, but has limitations
in resource-constrained settings, and represents an infection control
challenge during outbreaks. Self-collection devices have the potential
to shift phlebotomy closer to the point of care, supporting telemedicine
strategies and virtual clinical trials. Here we assess a capillary blood
micro-sampling device, the Tasso Serum Separator Tube (SST), for
measuring blood protein levels in healthy subjects and non-hospitalized
COVID-19 patients. 57 healthy controls and 56 participants with
mild/moderate COVID-19 were recruited at two U.S. military healthcare
facilities. Healthy controls donated Tasso SST capillary serum, venous
plasma and venous serum samples at multiple time points, while COVID-19
patients donated a single Tasso SST serum sample at enrolment.
Concentrations of 17 protein inflammatory biomarkers were measured in
all biospecimens by Ella multi-analyte immune-assay. Tasso SST serum
protein measurements in healthy control subjects were highly
reproducible, but their agreements with matched venous samples varied.
Most of the selected proteins, including CRP, Ferritin, IL-6 and PCT,
were well-correlated between Tasso SST and venous serum with little
sample type bias, but concentrations of D-dimer, IL-1B and IL-1Ra were
not. Self-collection at home with delayed sample processing was
associated with significant concentrations differences for several
analytes compared to supervised, in-clinic collection with rapid
processing. Finally, Tasso SST serum protein concentrations were
significantly elevated in in non-hospitalized COVID-19 patients compared
with healthy controls. Self-collection of capillary blood with
micro-sampling devices provides an attractive alternative to routine
phlebotomy. However, concentrations of certain analytes may differ
significantly from those in venous samples, and factors including user
proficiency, temperature control and time lags between specimen
collection and processing need to be considered for their effect on
sample quality and reproducibility. \| \|Fisheries management and
ecology \|The COVID-19 pandemic transformed social and economic systems
globally, including fisheries systems. Decreases in seafood demand,
supply chain disruptions, and public safety regulations required
numerous adaptations to maintain the livelihoods and social resilience
of fishing communities. Surveys, interviews, and focus groups were
undertaken to assess impacts from and adaptive responses to the pandemic
in commercial fisheries in five U.S. regions: the Northeast, California,
Alaska, the U.S. Caribbean, and the Pacific Islands. Fishery adaptation
strategies were categorized using the Resist-Accept-Direct (RAD)
framework, a novel application to understand social transformation in a
social-ecological system in response to a disturbance. A number of
innovations emerged, or were facilitated, that could improve the
fisheries’ resilience to future disruptions. Fishers with diversified
options and strategic flexibility generally fared better, i.e., had
fewer disruptions to their livelihoods. Using the RAD framework to
identify adaptation strategies from fishery system actors highlights
opportunities for improving resilience of fisheries social-ecological
systems to future stressors. © 2022 The Authors. Fisheries Management
and Ecology published by John Wiley & Sons Ltd. \| \|Hawai’i journal of
health & social welfare \|NA \| \|Internal and emergency medicine
\|Colorectal cancer (CRC) is one of the leading causes of cancer death
worldwide. Many communities remain under the 80% CRC screening goal. We
aimed to identify factors associated with non-adherence to CRC screening
and to describe the effect of the COVID-19 pandemic in CRC screening
patterns. A retrospective review of patients aged 50-75 years seen at
the Griffin Faculty Physicians primary care offices between January 2019
and December 2020 was performed. Logistic regression models were used to
identify factors associated with CRC screening non-adherence. Of 12,189
patients, 66.2% had an updated CRC screen. On univariable logistic
regression, factors associated with CRC screening non-adherence included
age ≤ 55 years \[odds ratio (OR) 2.267, p \< 0.001\], White/Caucasian
race (OR 0.858, p = 0.030), Medicaid insurance (OR 2.097, p \< 0.001),
morbid obesity (OR 1.436, p \< 0.001), current cigarette smoking (OR
1.849, p \< 0.001), and elevated HbA1c (OR 1.178, p = 0.004). Age,
Medicaid insurance, morbid obesity, current smoking, and HbA1c ≥ 6.5%
remained significant in the final multivariable model. Compared to 2019,
there was an 18.2% decrease in the total number of CRC screening tests
in 2020. The proportion of colonoscopy procedures was lower in 2020
compared to the proportion of colonoscopy procedures conducted in 2019
(65.9% vs 81.7%, p \< 0.001), with a concurrent increase in stool-based
tests. CRC screening rates in our population are comparable to national
statistics but below the 80% goal. COVID-19 affected CRC screening. Our
results underscore the need to identify patient groups most vulnerable
to missing CRC screening and highlight the importance of stool-based
testing to bridge screening gaps. © 2022. The Author(s), under exclusive
licence to Società Italiana di Medicina Interna (SIMI). \| \|JAMA
network open \|Data about the duration of protection of 2 and 3 doses of
BNT162b2 in children and adolescents are needed to help inform
recommendations for boosters in this age group. To evaluate vaccine
effectiveness (VE) and durability associated with 2 doses of BNT162b2
against Delta- and Omicron-related emergency department (ED) and urgent
care (UC) encounters among adolescents aged 12 to 17 years and to
estimate VE associated with 3 doses against these same outcomes. This
test-negative case-control study was conducted at Kaiser Permanente
Southern California, an integrated health care system using electronic
health records in the US. Participants included Kaiser Permanente
Southern California members ages 12 to 17 years with an ED or UC
encounter from November 1, 2021, through March 18, 2022, for acute
respiratory infection who were tested for SARS-CoV-2 via a reverse
transction-polymerase chain reaction test. Analyses were conducted from
March 21 to June 22, 2022. BNT162b2 vaccination status ascertained from
electronic health records and state registry data. The main outcome was
VE associated with BNT162b2 against ED and UC encounters related to
Delta or Omicron variant SARS-CoV-2 infection. Analyses were conducted
among 3168 adolescents, including 1004 with ED visits and 2164 with UC
visits. Median (IQR) age was 15 (13-16) years, and 1461 (46.1%) were
boys. In adjusted analyses, VE associated with 2 doses of BNT162b2
against ED or UC encounters was highest within the first 2 months for
both Delta (89% \[95% CI, 69% to 96%\]) and Omicron (73% \[95% CI, 54%
to 84%\]) variants but waned to 49% (95% CI, 27% to 65%) for the Delta
variant and 16% (95% CI, -7% to 34%) for the Omicron variant at 6 months
and beyond. A third dose of BNT162b2 was associated with improved
protection against the Omicron variant (87% \[95% CI, 72% to 94%\])
after a median (IQR) of 19 (9-32) days after dose 3. These findings
suggest that 2 doses of the BNT162b2 COVID-19 vaccine were associated
with high levels of protection against ED and UC encounters related to
the Delta and Omicron variants of SARS-CoV-2 in the first few months
after vaccination. However, effectiveness waned over time, especially
against Omicron. A third dose of BNT162b2 was associated with improved
protection against Omicron beyond that seen initially after 2 doses,
underscoring the importance of boosters for adolescents aged 12 to 17
years. \| \|Cancer epidemiology, biomarkers & prevention : a publication
of the American Association for Cancer Research, cosponsored by the
American Society of Preventive Oncology \|Cancer screening is a complex
process involving multiple steps and levels of influence (e.g., patient,
provider, facility, health care system, community, or neighborhood). We
describe the design, methods, and research agenda of the
Population-based Research to Optimize the Screening Process (PROSPR II)
consortium. PROSPR II Research Centers (PRC), and the Coordinating
Center aim to identify opportunities to improve screening processes and
reduce disparities through investigation of factors affecting cervical,
colorectal, and lung cancer screening in U.S. community health care
settings. We collected multilevel, longitudinal cervical, colorectal,
and lung cancer screening process data from clinical and administrative
sources on &gt;9 million racially and ethnically diverse individuals
across 10 heterogeneous health care systems with cohorts beginning
January 1, 2010. To facilitate comparisons across organ types and
highlight data breadth, we calculated frequencies of multilevel
characteristics and volumes of screening and diagnostic tests/procedures
and abnormalities. Variations in patient, provider, and facility
characteristics reflected the PROSPR II health care systems and
differing target populations. PRCs identified incident diagnoses of
invasive cancers, in situ cancers, and precancers (invasive: 372
cervical, 24,131 colorectal, 11,205 lung; in situ: 911 colorectal, 32
lung; precancers: 13,838 cervical, 554,499 colorectal). PROSPR II’s
research agenda aims to advance: (i) conceptualization and measurement
of the cancer screening process, its multilevel factors, and quality;
(ii) knowledge of cancer disparities; and (iii) evaluation of the
COVID-19 pandemic’s initial impacts on cancer screening. We invite
researchers to collaborate with PROSPR II investigators. PROSPR II is a
valuable data resource for cancer screening researchers. ©2022 American
Association for Cancer Research. \| \|Open forum infectious diseases
\|There is limited information on the functional consequences of
coronavirus disease 2019 (COVID-19) vaccine side effects. To support
patient counseling and public health messaging, we describe the risk and
correlates of COVID-19 vaccine side effects sufficient to prevent work
or usual activities and/or lead to medical care (“severe” side effects).
The EPICC study is a longitudinal cohort study of Military Healthcare
System beneficiaries including active duty service members, dependents,
and retirees. We studied 2789 adults who were vaccinated between
December 2020 and December 2021. Severe side effects were most common
with the Ad26.COV2.S (Janssen/Johnson and Johnson) vaccine, followed by
mRNA-1273 (Moderna) then BNT162b2 (Pfizer/BioNTech). Severe side effects
were more common after the second than first dose (11% vs 4%; P \<
.001). First (but not second) dose side effects were more common in
those with vs without prior severe acute respiratory syndrome
coronavirus 2 infection (9% vs 2%; adjusted odds ratio \[aOR\], 5.84;
95% CI, 3.8-9.1), particularly if the prior illness was severe or
critical (13% vs 2%; aOR, 10.57; 95% CI, 5.5-20.1) or resulted in
inpatient care (17% vs 2%; aOR, 19.3; 95% CI, 5.1-72.5). Side effects
were more common in women than men but not otherwise related to
demographic factors. Vaccine side effects sufficient to prevent usual
activities were more common after the second than first dose and varied
by vaccine type. First dose side effects were more likely in those with
a history of COVID-19-particularly if that prior illness was severe or
associated with inpatient care. These findings may assist clinicians and
patients by providing a real-world evaluation of the likelihood of
experiencing impactful postvaccine symptoms. Published by Oxford
University Press on behalf of Infectious Diseases Society of America
2022.This work is written by (a) US Government employee(s) and is in the
public domain in the US. \| \|Open forum infectious diseases
\|Patient-reported outcomes of severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2) infection are an important measure of the
full burden of coronavirus disease (COVID). Here, we examine how (1)
infecting genotype and COVID-19 vaccination correlate with inFLUenza
Patient-Reported Outcome (FLU-PRO) Plus score, including by symptom
domains, and (2) FLU-PRO Plus scores predict return to usual activities
and health. The epidemiology, immunology, and clinical characteristics
of pandemic infectious diseases (EPICC) study was implemented to
describe the short- and long-term consequences of SARS-CoV-2 infection
in a longitudinal, observational cohort. Multivariable linear regression
models were run with FLU-PRO Plus scores as the outcome variable, and
multivariable Cox proportional hazards models evaluated effects of
FLU-PRO Plus scores on return to usual health or activities. Among the
764 participants included in this analysis, 63% were 18-44 years old,
40% were female, and 51% were White. Being fully vaccinated was
associated with lower total scores (β = -0.39; 95% CI, -0.57 to -0.21).
The Delta variant was associated with higher total scores (β = 0.25; 95%
CI, 0.05 to 0.45). Participants with higher FLU-PRO Plus scores were
less likely to report returning to usual health and activities (health:
hazard ratio \[HR\], 0.46; 95% CI, 0.37 to 0.57; activities: HR, 0.56;
95% CI, 0.47 to 0.67). Fully vaccinated participants were more likely to
report returning to usual activities (HR, 1.24; 95% CI, 1.04 to 1.48).
Full SARS-CoV-2 vaccination is associated with decreased severity of
patient-reported symptoms across multiple domains, which in turn is
likely to be associated with earlier return to usual activities. In
addition, infection with the Delta variant was associated with higher
FLU-PRO Plus scores than previous variants, even after controlling for
vaccination status. Published by Oxford University Press on behalf of
the Infectious Diseases Society of America 2022. \| \|Current
dermatology reports \|Neutrophilic dermatoses are a heterogeneous group
of disorders with significant overlap in associated conditions, clinical
presentation, and histopathologic features. This review provides a
structural overview of neutrophilic dermatoses that may present in the
inpatient setting along with diagnostic work-up and management
strategies. Sweet’s syndrome has been found in patients with coronavirus
disease 2019 (COVID-19). Pyoderma gangrenosum (PG) has been shown to be
equally associated with ulcerative colitis and Crohn’s disease. A
clinical trial shows that cyclosporine is equally effective as
prednisone in treating PG. Neutrophilic eccrine hidradenitis has been
found in the setting of newer antineoplastic medications, such as BRAF
inhibitors, as well as in the setting of malignancy without chemotherapy
exposure. Neutrophilic dermatoses are a rare and complex group of
dermatoses with varying and overlapping clinical presentations.
Physicians should be aware of the growing list of associated diseases in
order to build a better differential diagnosis or to potentially
investigate for co-existing disease. © The Author(s), under exclusive
licence to Springer Science+Business Media, LLC, part of Springer Nature
2022. \| \|International journal of endocrinology \|\[This corrects the
article DOI: 10.1155/2022/9322332.\]. Copyright © 2022 Yuanyuan Fu et
al. \| \|Hawai’i journal of health & social welfare \|The Next Gen
Hawai’i social media project was initiated in the fall of 2020 to
address ongoing public health concerns and the need for accessible and
reliable information across Hawai’i’s diverse communities by
strategically amplifying the voices of Hawai’i’s youth in their Native
languages. The collaborative effort arose from conversations within the
Hawai’i’s Native Hawaiian & Pacific Islander COVID-19 Response,
Recovery, and Resilience Team, composed of diverse public and private
organizations involved in statewide COVID-19 response efforts for Native
Hawaiian and Pacific Islander communities. Next Gen Hawai’i’s focus was
on Native Hawaiian, Pacific Islander, Filipino, and other populations
disproportionately suffering from COVID-19. Five social media platforms
were developed to spread messaging to youth and young adults about
COVID-19. Public Health Ambassadors (from high school to young adults)
were recruited and engaged to create culturally and linguistically
rooted messaging to promote public health and prevention-based social
norms. This strength-based approach recognized youth as important
community leaders and ambassadors for change and empowered them to
create content for dissemination on platforms with national and global
reach. Messaging was designed to build individual, community, and
digital health literacy while integrating core cultural values and
strengths of Native Hawaiian, Pacific Islander, and Filipino
communities. Over 250 messages have been delivered across Next Gen
Hawai’i social media channels on topics including vaccine information,
mask-wearing, staying together over distances, mental health, and
in-languages resources in Chuukese, Chamorro, Marshallese, Samoan,
Hawaiian, Ilocano, Tagalog, and other Pacific-basin languages. Reach has
included more than 75 000 views from various social media channels,
media features, successful webinars, and relevant conference
presentations. This Public Health Insights article provides an overview
of Next Gen Hawai’i’s activities and achievements as well as lessons
learned for other youth-focused public health social media campaigns and
organizations. ©Copyright 2022 by University Health Partners of Hawai‘i
(UHP Hawai‘i). \| \|Open forum infectious diseases \|As the number of
coronavirus disease 2019 (COVID-19) cases continue to surge worldwide
and new variants emerge, additional accurate, rapid, and noninvasive
screening methods to detect severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2) are needed. The number of COVID-19 cases
reported globally is \>455 million, and deaths have surpassed 6 million.
Current diagnostic methods are expensive, invasive, and produce delayed
results. While COVID-19 vaccinations are proven to help slow the spread
of infection and prevent serious illness, they are not equitably
available worldwide. Almost 40% of the world’s population remains
unvaccinated. Evidence suggests that SARS-CoV-2 virus-associated
volatile organic compounds found in the breath, urine, and sweat of
infected individuals can be detected by canine olfaction. Medical
detection dogs may be a feasible, accurate, and affordable SARS-CoV-2
screening method. In this double-blinded, case-control, validation
study, we obtained sweat samples from inpatients and outpatients tested
for SARS-CoV-2 by a polymerase chain reaction test. Medical detection
dogs were trained to distinguish SARS-CoV-2-positive samples from
SARS-CoV-2-negative samples using reward-based reinforcement. Samples
were obtained from 584 individuals (6-97 years of age; 24% positive
SARS-CoV-2 samples and 76% negative SARS-CoV-2 samples). In the testing
phase, all dogs performed with high accuracy in detecting SARS-CoV-2.
The overall diagnostic sensitivity was 98%, and specificity was 92%. In
a follow-up phase, 1 dog screened 153 patients for SARS-CoV-2 in a
hospital setting with 96% diagnostic sensitivity and 100% specificity.
Canine olfaction is an accurate and feasible method for diagnosis of
SARS-CoV-2, including asymptomatic and presymptomatic infected
individuals. © The Author(s) 2022. Published by Oxford University Press
on behalf of Infectious Diseases Society of America. \| \|Journal of
cosmetic dermatology \|While there are literature reporting increased
incidence of hair loss in COVID-19 patients, insufficient evidence
exists on the topic to date. This review aims to identify the existing
evidence and clinical characteristics of hair loss with COVID-19
infection. Following the PRISMA Extension for Scoping Reviews, MEDLINE
and EMBASE were searched for all peer-reviewed articles with relevant
keywords including “Alopecia,” “Telogen Effluvium (TE),” and “COVID-19”
from their inception to November 20, 2021. A total of 26 articles, with
9 observational studies and 17 case reports or series (a total of 58
cases), were included. Most studies dealt with TE. There were no clear
trends between COVID-19 severity and the extent of hair loss. Analysis
of the 58 cases also found similar results with most of the cases being
female (82.8%), the median onset of hair loss of 2.0 months, and the
median time to recovery of hair loss of 5.0 months with a resolution
rate of 95%. While this systematic review revealed uncertainty and a
lack of strong evidence regarding the association of COVID-19 and hair
loss, hair loss in COVID-19 may mainly include TE and be reversible in
nature. Future studies are warranted to determine the detailed
pathophysiology and risk factors of hair loss in COVID-19, including
possible roles of estrogen, progesterone, and pro-inflammatory
cytokines. © 2022 Wiley Periodicals LLC. \| \|Frontiers in medicine
\|The US recently suffered the fourth and most severe wave of the
COVID-19 pandemic. This wave was driven by the SARS-CoV-2 Omicron, a
highly transmissible variant that infected even vaccinated people.
Vaccination coverage disparities have played an important role in
shaping the epidemic dynamics. Analyzing the epidemiological impact of
this uneven vaccination coverage is essential to understand local
differences in the spread and outcomes of the Omicron wave. Therefore,
the objective of this study was to quantify the impact of vaccination
coverage disparity in the US in the dynamics of the COVID-19 pandemic
during the third and fourth waves of the pandemic driven by the Delta
and Omicron variants. This cross-sectional study used COVID-19 cases,
deaths, and vaccination coverage from 2,417 counties. The main outcomes
of the study were new COVID-19 cases (incidence rate per 100,000 people)
and new COVID-19 related deaths (mortality rate per 100,000 people) at
county level and the main exposure variable was COVID-19 vaccination
rate at county level. Geospatial and data visualization analyses were
used to estimate the association between vaccination rate and COVID-19
incidence and mortality rates for the Delta and Omicron waves. During
the Omicron wave, areas with high vaccination rates (\>60%) experienced
1.4 (95% confidence interval \[CI\] 1.3-1.7) times higher COVID-19
incidence rate compared to areas with low vaccination rates (\<40%).
However, mortality rate was 1.6 (95% CI 1.5-1.7) higher in these
low-vaccinated areas compared to areas with vaccination rates higher
than 60%. As a result, areas with low vaccination rate had a 2.2 (95% CI
2.1-2.2) times higher case-fatality ratio. Geospatial clustering
analysis showed a more defined spatial structure during the Delta wave
with clusters with low vaccination rates and high incidence and
mortality located in southern states. Despite the emergence of new virus
variants with differential transmission potential, the protective effect
of vaccines keeps generating marked differences in the distribution of
critical health outcomes, with low vaccinated areas having the largest
COVID-19 related mortality during the Delta and Omicron waves in the US.
Vulnerable communities residing in low vaccinated areas, which are
mostly rural, are suffering the highest burden of the COVID-19 pandemic
during the vaccination era. Copyright © 2022 Cuadros, Moreno, Musuka,
Miller, Coule and MacKinnon. \| \|Vaccines \|Native Hawaiians and other
Pacific Islanders (NHPIs) were disproportionately impacted by COVID-19
and remain significantly under-vaccinated against SARS-CoV-2. To
understand vaccine hesitancy, we surveyed 1124 adults residing in a
region with one of the lowest vaccination rates in Hawaii during our
COVID-19 testing program. Probit regression analysis revealed that
race/ethnicity was not directly associated with the probability of
vaccine uptake. Instead, a higher degree of trust in official sources of
COVID-19 information increased the probability of vaccination by 20.68%,
whereas a higher trust in unofficial sources decreased the probability
of vaccination by 12.49% per unit of trust. These results revealed a
dual and opposing role of trust on vaccine uptake. Interestingly, NHPIs
were the only racial/ethnic group to exhibit a significant positive
association between trust in and consumption of unofficial sources of
COVID-19 information, which explained the vaccine hesitancy observed in
this indigenous population. These results offer novel insight relevant
to COVID-19 mitigation efforts in minority populations. \|
\|International journal of molecular sciences \|The receptor of advanced
glycation end products (RAGE) is a receptor that is thought to be a key
driver of inflammation in pregnancy, SARS-CoV-2, and also in the
comorbidities that are known to aggravate these afflictions. In addition
to this, vulnerable populations are particularly susceptible to the
negative health outcomes when these afflictions are experienced in
concert. RAGE binds a number of ligands produced by tissue damage and
cellular stress, and its activation triggers the proinflammatory
transcription factor Nuclear Factor Kappa B (NF-κB), with the subsequent
generation of key proinflammatory cytokines. While this is important for
fetal membrane weakening, RAGE is also activated at the end of pregnancy
in the uterus, placenta, and cervix. The comorbidities of hypertension,
cardiovascular disease, diabetes, and obesity are known to lead to poor
pregnancy outcomes, and particularly in populations such as Native
Hawaiians and Pacific Islanders. They have also been linked to RAGE
activation when individuals are infected with SARS-CoV-2. Therefore, we
propose that increasing our understanding of this receptor system will
help us to understand how these various afflictions converge, how forms
of RAGE could be used as a biomarker, and if its manipulation could be
used to develop future therapeutic targets to help those at risk. \|
\|The Science of the total environment \|Methane (CH4) is a potent
greenhouse gas and also plays a significant role in tropospheric
chemistry. High-frequency (sub-hourly) measurements of CH4 and carbon
isotopic ratio (δ13CH4) have been conducted at Pune (18.53°N, 73.80°E),
an urban environment in India, during 2018-2020. High CH4 concentrations
were observed, with a mean of 2100 ± 196 ppb (1844-2749 ppb), relative
to marine background concentrations. The δ13CH4 varied between -45.11
and -50.03 ‰ for the entire study period with an average of -47.41 ±
0.94 ‰. The diurnal variability of CH4 typically showed maximum values
in the morning (08:00-09:00 local time) and minimum in the afternoon
(15:00 local time). The deepest diurnal amplitude of \~500 ppb was
observed during winter (December-February), which was reduced to less
than half, \~200 ppb, during the summer (March-May). CH4 concentration
at Pune showed a strong seasonality (470 ppb), much higher than that at
Mauna Loa, Hawaii. On the other hand, δ13CH4 records did not show
distinct seasonality at Pune. The δ13CH4 values revealed that the
significant sources of CH4 in Pune were from the waste sector (enhanced
during the monsoon season; signature of depleted δ13CH4), followed by
the natural gas sector with a signature of enriched δ13CH4. Our analysis
of Covid-19 lockdown (April to May 2020) effect on the CH4 variability
showed no signal in the CH4 variability; however, the isotopic analysis
indicated a transient shift in the CH4 source to the waste sector (early
summer of 2020). Copyright © 2022 Elsevier B.V. All rights reserved. \|
\|Journal of general internal medicine \|Animation in medical education
has boomed over the past two decades, and demand for distance learning
technologies will likely continue in the context of the COVID-19
pandemic. However, experimental data guiding best practices for
animation in medical education are scarce. To compare the efficacy of
two animated video styles in a diabetes pharmacotherapy curriculum for
internal medicine residents. Learners were randomized to receive one of
two versions of the same multimodal didactic curriculum. They received
identical lectures, group activities, and quizzes, but were randomized
to either digital chalk talk (DCT) videos or Sugar-Coated Science (SCS).
SCS is an animated series using anthropomorphic characters, stories, and
mnemonics to communicate knowledge. Ninety-two internal medicine
residents at a single academic medical center received the curriculum
within ambulatory medicine didactics. Knowledge was measured at multiple
time points, as was residents’ self-reported comfort using each
medication class covered. Surveys assessed video acceptability and
telepresence. Key themes were identified from open-ended feedback.
Baseline knowledge was low, consistent with prior needs assessments. On
immediate posttest, mean scores were higher with SCS than DCT (74.8%
versus 68.4%), but the difference was not statistically significant, p =
0.10. Subgroup analyses revealed increased knowledge in the SCS group
for specific medication classes. Delayed posttest showed significant
knowledge gains averaging 17.6% across all participants (p \< 0.05);
these gains were similar between animation types. SCS achieved
significantly higher telepresence, entertainment, and acceptability
scores than DCT. Qualitative data suggested that residents prioritize
well-designed, multimodal curricula over specific animation
characteristics. SCS and DCTs both led to learning within a multimodal
curriculum, but SCS significantly enhanced learner experience. Animation
techniques exemplified by both SCS and DCTs have roles in the medical
educator toolkit. Selection between them should incorporate context,
learner factors, and production resources. © 2022. The Author(s), under
exclusive licence to Society of General Internal Medicine. \| \|CMAJ
open \|Asian Canadians have experienced increased cases of racialized
discrimination after the first emergence of SARS-CoV-2 in China. This
study examined how the COVID-19 pandemic has affected Asian Canadians’
sense of safety and belonging in their Canadian (i.e., geographical)
communities. We applied a qualitative description study design in which
semistructured interviews were conducted from Mar. 23 to May 27, 2021.
Purposive and snowball sampling methods were used to recruit Asian
Canadians diverse in region, gender and age. Interviews were conducted
through Zoom videoconference or telephone, and independent qualitative
thematic analysis in duplicate was used to derive primary themes and
subthemes. Thirty-two Asian Canadians (median age 35 \[interquartile
range 24-46\] yr, 56% female, 44% East Asian) participated in the study.
We identified 5 predominant themes associated with how the COVID-19
pandemic affected the participants’ sense of security and belonging to
their communities: relation between socioeconomic status (SES) and
exposure to discrimination (i.e., how SES insulates or exposes
individuals to increased discrimination); politics, media and the
COVID-19 pandemic (i.e., the key role that politicians and media played
in enabling spread of discrimination against and fear of Asian people);
effect of discrimination on mental and social health (i.e., people’s
ability to interact and form meaningful relationships with others);
coping with the impact of discrimination (i.e., the way people appraise
and move forward in identity-threatening situations); and implications
for sense of safety and sense of belonging (i.e., people feeling unable
to safely use public spaces in person, including the need to remain
alert in anticipation of harm, leading to distress and exhaustion).
During the COVID-19 pandemic, Asian Canadians in our study felt unsafe
owing to the uncertain, unexpected and unpredictable nature of
discrimination, but also felt a strong sense of belonging to Canadian
society and felt well connected to their Asian Canadian communities.
Future work should seek to explore the influence of social media on
treatment of and attitudes toward Asian Canadians. © 2022 CMA Impact
Inc. or its licensors. \| \|Journal of general internal medicine \|NA \|
\|Hawai’i journal of health & social welfare \|NA \| \|Ecology
\|Managing wildlife populations in the face of global change requires
regular data on the abundance and distribution of wild animals, but
acquiring these over appropriate spatial scales in a sustainable way has
proven challenging. Here we present the data from Snapshot USA 2020, a
second annual national mammal survey of the USA. This project involved
152 scientists setting camera traps in a standardized protocol at 1485
locations across 103 arrays in 43 states for a total of 52,710
trap-nights of survey effort. Most (58) of these arrays were also
sampled during the same months (September and October) in 2019,
providing a direct comparison of animal populations in 2 years that
includes data from both during and before the COVID-19 pandemic. All
data were managed by the eMammal system, with all species
identifications checked by at least two reviewers. In total, we recorded
117,415 detections of 78 species of wild mammals, 9236 detections of at
least 43 species of birds, 15,851 detections of six domestic animals and
23,825 detections of humans or their vehicles. Spatial differences
across arrays explained more variation in the relative abundance than
temporal variation across years for all 38 species modeled, although
there are examples of significant site-level differences among years for
many species. Temporal results show how species allocate their time and
can be used to study species interactions, including between humans and
wildlife. These data provide a snapshot of the mammal community of the
USA for 2020 and will be useful for exploring the drivers of spatial and
temporal changes in relative abundance and distribution, and the impacts
of species interactions on daily activity patterns. There are no
copyright restrictions, and please cite this paper when using these
data, or a subset of these data, for publication. © 2022 The Authors.
Ecology © 2022 The Ecological Society of America. \| \|Infectious
disease clinics of North America \|The violence and victimization
brought by colonization and slavery and justified for over a century by
race-based science have resulted in enduring inequities for black,
Indigenous and people of color (BIPOC) across the United States. This is
particularly true if BIPOC individuals have other intersecting devalued
identities. We highlight how such longstanding inequities paved the way
for the disproportionate burdens of coronavirus disease 2019 (COVID-19)
among the BIPOC populations across the country and provide
recommendations on how to improve COVID-19 mitigation strategies with
the goal of eliminating disparities. Copyright © 2022 Elsevier Inc. All
rights reserved. \| \|Vaccines \|Having been affected by the highest
increase in COVID-19 cases since the start of the pandemic, Honolulu and
Maui counties in Hawaii implemented vaccine passport mandates for select
industries in September 2021. However, the degree to which such mandates
impacted COVID-19 mitigation efforts and economics remains poorly
understood. Herein, we describe the effects of these mandates on changes
in three areas using difference-in-difference regression models: (1)
business foot traffic; (2) number of COVID-19 cases per 100,000
individuals, and (3) COVID-19 vaccination rates across counties affected
or unaffected by the mandates. We observed that although businesses
affected by mandates experienced a 6.7% decrease in foot traffic over
the 14 weeks after the mandates were implemented, the number of COVID-19
cases decreased by 19.0%. Notably, the vaccination rate increased by
1.41% in counties that implemented mandates. In addition, towards the
end of the studied period, the level of foot traffic at impacted
businesses converged towards the level of that of non-impacted
businesses. As such, the trade-off in temporary losses at businesses was
met with significant gains in public health and safety. \|
\|International journal of environmental research and public health \|In
the face of great uncertainty and a global crisis from COVID-19,
mathematical and epidemiologic COVID-19 models proliferated during the
pandemic. Yet, many models were not created with the explicit audience
of policymakers, the intention of informing specific scenarios, or
explicit communication of assumptions, limitations, and complexities.
This study presents a case study of the roles, uses, and approaches to
COVID-19 modeling and forecasting in one state jurisdiction in the
United States. Based on an account of the historical real-world events
through lived experiences, we first examine the specific modeling
considerations used to inform policy decisions. Then, we review the
real-world policy use cases and key decisions that were informed by
modeling during the pandemic including the role of modeling in informing
planning for hospital capacity, isolation and quarantine facilities, and
broad public communication. Key lessons are examined through the
real-world application of modeling, noting the importance of locally
tailored models, the role of a scientific and technical advisory group,
and the challenges of communicating technical considerations to a public
audience. \| \|Clinical infectious diseases : an official publication of
the Infectious Diseases Society of America \|Comparison of humoral
responses in severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) vaccinees, those with SARS-CoV-2 infection, or combinations
of vaccine/ infection (“hybrid immunity”) may clarify predictors of
vaccine immunogenicity. We studied 2660 US Military Health System
beneficiaries with a history of SARS-CoV-2 infection-alone (n = 705),
vaccination-alone (n = 932), vaccine-after-infection (n = 869), and
vaccine-breakthrough-infection (n = 154). Peak anti-spike-immunoglobulin
G (IgG) responses through 183 days were compared, with adjustment for
vaccine product, demography, and comorbidities. We excluded those with
evidence of clinical or subclinical SARS-CoV-2 reinfection from all
groups. Multivariable regression results indicated that
vaccine-after-infection anti-spike-IgG responses were higher than
infection-alone (P \< .01), regardless of prior infection severity. An
increased time between infection and vaccination was associated with
greater post-vaccination IgG response (P \< .01). Vaccination-alone
elicited a greater IgG response but more rapid waning of IgG (P \< .01)
compared with infection-alone (P \< .01). BNT162b2 and mRNA-1273
vaccine-receipt was associated with greater IgG responses compared with
JNJ-78436735 vaccine-receipt (P \< .01), regardless of infection
history. Those with vaccine-after-infection or
vaccine-breakthrough-infection had a more durable anti-spike-IgG
response compared to infection-alone (P \< .01). Vaccine-receipt
elicited higher anti-spike-IgG responses than infection-alone, although
IgG levels waned faster in those vaccinated (compared to
infection-alone). Vaccine-after-infection elicits a greater humoral
response compared with vaccine or infection alone; and the timing, but
not disease severity, of prior infection predicted these
post-vaccination IgG responses. While differences between groups were
small in magnitude, these results offer insights into vaccine
immunogenicity variations that may help inform vaccination timing
strategies. © The Author(s) 2022. Published by Oxford University Press
on behalf of the Infectious Diseases Society of America. All rights
reserved. For permissions, please e-mail:
<journals.permissions@oup.com>. \| \|Reproductive toxicology (Elmsford,
N.Y.) \|Remdesivir (RDV) is the first antiviral drug to be approved by
the US Food and Drug Administration for the treatment of COVID-19. While
the general safety of RDV has been studied, its reproductive risk,
including embryotoxicity, is largely unknown. Here, to gain insights
into its embryotoxic potential, we investigated the effects of RDV on
mouse preimplantation embryos cultured in vitro at the concentrations
comparable to the therapeutic plasma levels. Exposure to RDV (2-8 µM)
did not affect the initiation of blastocyst formation, although the
maintenance of the cavity failed at 8 µM due to increased cell death.
While exposure to 2-4 µM permitted the cavity maintenance, expressions
of developmental regulator genes associated with the inner cell mass
(ICM) lineage were significantly diminished. Adverse effects of RDV
depended on the duration and timing of exposure, as treatment between
the 8-cell to early blastocyst stage most sensitively affected cavity
expansion, gene expressions, and cell proliferation, particularly of the
ICM than the trophectoderm lineage. GS-441524, a major metabolite of
RDV, did not impair blastocyst formation or cavity expansion, although
it altered gene expressions in a manner differently from RDV.
Additionally, RDV reduced the viability of human embryonic stem cells,
which were used as a model for the human ICM lineage, more potently than
GS-441524. These findings suggest that RDV is potentially embryotoxic to
impair the pluripotent lineage, and will be useful for designing and
interpreting further in vitro and in vivo studies on the reproductive
toxicity of RDV. Copyright © 2022 Elsevier Inc. All rights reserved. \|
\|Journal of clinical virology : the official publication of the Pan
American Society for Clinical Virology \|NA \| \|British journal of
educational technology : journal of the Council for Educational
Technology \|The annual instructional virtual team Project X brings
together professors and students from across the globe to engage in
client projects. The 2020 project was challenged by the global
disruption of the COVID-19 pandemic. This paper draws on a quantitative
dataset from a post-project survey among 500 participating students and
a qualitative narrative inquiry of personal experiences of the faculty
members. The findings reveal how innovative use of a variety of
collaboration and communication technologies helped students and their
professors in building emotional connection and compassion to support
each other in the midst of the crisis, and to accomplish the project
despite connectivity disruptions. The results suggest that the role of
an instructor changed to a coach and mentor, and technology was used to
create a greater sense of inclusion and co-presence in student-faculty
interactions. Ultimately, the paper highlights the role of technology to
help the participants navigate sudden crisis affecting a global online
instructional team project. The adaptive instructional teaching
strategies and technologies depicted in this study offer transformative
potential for future developments in higher education. © 2022 British
Educational Research Association. \| \|Asian journal of psychiatry
\|While the coronavirus disease 2019 (COVID-19) pandemic has led to
increased burnout among frontline healthcare workers (HCWs), little
research has been done regarding the potential psychological burden
among public health officials who have worked tirelessly to tackle the
pandemic from an administrative perspective. This study aimed to
determine the prevalence of burnout, depression, and job-related stress
in Japanese public health officers amid the COVID-19 pandemic. We
conducted an anonymous, self-administered web-based cross-sectional
survey including basic demographics, work-related questions, the Maslach
Burnout Inventory, Patient Health Questionnaire-9, Utrecht Work
Engagement Scale-3, and Brief Job Stress Questionnaire. 100 public
health officers working in the public health centers (PHCs) in Okayama,
Japan, answered the survey in December 2021 when the 5th surge in the
number of COVID-19 was over. The prevalence of burnout, depression, and
job-related stress was 27%, 43%, and 62%, respectively. The multivariate
logistic analysis demonstrated that females, public health nurses, and
those who suffered from a lack of support from their workplaces were
significantly associated with psychological distress. While we tend to
focus on mitigation plans to help alleviate burnout of frontline HCWs,
more focus is needed to help public health officers, and public health
nurses, in particular, to alleviate their psychological distress and
job-related stress to prevent further staff shortages and secure
sustainable health systems. Copyright © 2022 Elsevier B.V. All rights
reserved. \| \|PloS one \|It is critical to capture data and modeling
from the COVID-19 pandemic to understand as much as possible and prepare
for future epidemics and possible pandemics. The Hawaiian Islands
provide a unique opportunity to study heterogeneity and demographics in
a controlled environment due to the geographically closed borders and
mostly uniform pandemic-induced governmental controls and restrictions.
The goal of the paper is to quantify the differences and similarities in
the spread of COVID-19 among different Hawaiian islands as well as
several other archipelago and islands, which could potentially help us
better understand the effect of differences in social behavior and
various mitigation measures. The approach should be robust with respect
to the unavoidable differences in time, as the arrival of the virus and
promptness of mitigation measures may vary significantly among the
chosen locations. At the same time, the comparison should be able to
capture differences in the overall pandemic experience. We examine
available data on the daily cases, positivity rates, mobility, and
employ a compartmentalized model fitted to the daily cases to develop
appropriate comparison approaches. In particular, we focus on merge
trees for the daily cases, normalized positivity rates, and baseline
transmission rates of the models. We observe noticeable differences
among different Hawaiian counties and interesting similarities between
some Hawaiian counties and other geographic locations. The results
suggest that mitigation measures should be more localized, that is,
targeting the county level rather than the state level if the counties
are reasonably insulated from one another. We also notice that the
spread of the disease is very sensitive to unexpected events and certain
changes in mitigation measures. Despite being a part of the same
archipelago and having similar protocols for mitigation measures,
different Hawaiian counties exhibit quantifiably different dynamics of
the spread of the disease. One potential explanation is that not
sufficiently targeted mitigation measures are incapable of handling
unexpected, localized outbreak events. At a larger-scale view of the
general spread of the disease on the Hawaiian island counties, we find
very interesting similarities between individual Hawaiian islands and
other archipelago and islands. \| \|Frontiers in public health
\|Service-learning is a high-impact educational practice at the core of
the undergraduate public health degree at the University of Hawai’i at
Mānoa (UHM). This practice provides an invaluable learning experience
and professional opportunity for students to collaborate with community
partners and make significant contributions in the field. The COVID-19
pandemic halted or disrupted service-learning experiences as community
partners adapted to shifting mandates and emergency orders. Surveying
the rapidly evolving landscape of partner organizations to support
service-learning is a challenge. Assessing changes to the program
mentorship or satisfaction is the first step to developing protocols to
ensure standardization of service-learning during times of crisis. This
study will address if and how the pandemic impacted students’
satisfaction with required service-learning experiences. Furthermore,
authors hope to create a comprehensive list of practicum partnering
organizations, both focused on pandemic response and, more generally, of
the service-learning students at UHM, with the intent to increase
students and community partners in local service-learning. Assessments
were conducted to assess the impact of COVID-19 on undergraduate
students’ experiences with service-learning through use of a program
exit survey. The authors hypothesized pandemic-related adjustments would
not affect student satisfaction or skill development. Despite challenges
associated with the pandemic and emergency online transitions, students
persisted in personal and professional growth associated with
service-learning. This developed resilience supports students as they
graduate and enter a workforce adapting to remote work demands and
community needs. Copyright © 2022 Kehl, Patil, Tagorda and
Nelson-Hurwitz. \| \|Hawai’i journal of health & social welfare \|The
coronavirus disease (COVID-19) pandemic has placed extraordinary strain
on health care systems. This has led to increased stress among health
care workers, and nurses in particular, which has had a negative impact
on their physical and psychosocial wellbeing. This is likely to
negatively impact the nursing workforce at the state and national levels
as the pandemic continues. The purpose of this study was to assess
whether nurses licensed in Hawai’i have considered leaving the
workforce. A cross-sectional online survey was conducted among Hawai’i
nurses at all levels of licensure, with 421 responding. Of these nurses,
97 (23.0%) reported considering leaving the workforce, with safety
(39.2%) and family/caregiver strain (32.0%) being the most common
reasons. Reconsidering whether they should stay employed in their
current roles (Odds ratio \[OR\] 2.05; 95% CI 1.56 - 2.69) and fear to
continue providing direct patient care (OR 1.97; 95% CI 1.54 - 2.54)
were associated with increased odds of having considered leaving the
workforce. Based on these results, the State of Hawai’i and local health
care organizations need to adjust their nursing workforce estimates and
address how to alleviate nurses’ stressors and safety concerns to
mitigate a potential workforce shortage. Research is needed to develop
interventions to support and empower nurses in their current roles but
also address future emergency preparedness. ©Copyright 2022 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|NA \| \|International journal of
dermatology \|Although there is literature reporting correlations
between varicella zoster virus (VZV) infections and COVID-19,
insufficient evidence exists in this regard. This scoping review aims to
identify the existing evidence regarding clinical characteristics of
primary VZV infection or reactivation in COVID-19. Following the PRISMA
Extension for Scoping Reviews, MEDLINE and EMBASE were searched for all
peer-reviewed articles with relevant keywords including “Zoster,”
“Herpes,” and “COVID-19” from their inception to November 20, 2021. A
total of 19 articles with three observational studies and 16 case
reports or series were included. Primary VZV infections or reactivation
were observed in 25 patients. Forty-eight percent of the patients had
disseminated VZV infection. The median time of VZV-related rash after
the onset of respiratory symptoms was 7.0 days (interquartile range:
0-18.8). Those with COVID-19 and primary VZV infection or reactivation
had low lymphocyte counts with a median of 0.67 × 103 /μl. This scoping
review identified uncertainty and a lack of strong evidence to see the
association between primary VZV infection or reactivation and COVID-19.
However, those with COVID-19 may be more likely to have disseminated
VZV, which poses an additional challenge from an infection prevention
standpoint. Future studies are warranted to determine the association
between primary VZV infection or reactivation and long-term consequences
related to COVID-19. © 2022 the International Society of Dermatology. \|
\|Advances in virology \|Septic shock is a severe complication of
COVID-19 patients. We aim to identify risk factors associated with
septic shock and mortality among COVID-19 patients. A total of 212
COVID-19 confirmed patients in Wuhan were included in this retrospective
study. Clinical outcomes were designated as nonseptic shock and septic
shock. Log-rank test was conducted to determine any association with
clinical progression. A prediction model was established using random
forest. The mortality of septic shock and nonshock patients with
COVID-19 was 96.7% (29/30) and 3.8% (7/182). Patients taking hypnotics
had a much lower chance to develop septic shock (HR = 0.096, p=0.0014).
By univariate logistic regression analysis, 40 risk factors were
significantly associated with septic shock. Based on multiple regression
analysis, eight risk factors were shown to be independent risk factors
and these factors were then selected to build a model to predict septic
shock with AUC = 0.956. These eight factors included disease severity
(HR = 15, p \< 0.001), age \> 65 years (HR = 2.6, p=0.012), temperature
\> 39.1°C (HR = 2.9, p=0.047), white blood cell count \> 10 × 10⁹ (HR =
6.9, p \< 0.001), neutrophil count \> 75 × 10⁹ (HR = 2.4, p=0.022),
creatine kinase \> 5 U/L (HR = 1.8, p=0.042), glucose \> 6.1 mmol/L (HR
= 7, p \< 0.001), and lactate \> 2 mmol/L (HR = 22, p \< 0.001). We
found 40 risk factors were significantly associated with septic shock.
The model contained eight independent factors that can accurately
predict septic shock. The administration of hypnotics could potentially
reduce the incidence of septic shock in COVID-19 patients. Copyright ©
2022 Shaoqiu Chen et al. \| \|Hawai’i journal of health & social welfare
\|The Hawai’i Physician Workforce project, launched in 2010,
investigates state physician workforce trends. Over the past decade,
workforce demands have continued to climb as the state struggles to
maintain the physician supply. This article describes the current state
of the physician workforce, the physician age landscape, past trends, as
well as initial changes to the physician supply with the COVID-19
pandemic. Data on practice location, full time equivalency of time spent
providing patient care in Hawai’i, and specialty of non-military
physicians were clarified and informed via survey, internet search, and
direct calling methodologies. A proprietary microsimulation modeling
methodology was used to assess physician demand. The current estimated
physician shortage is between 710 and 1,008 full time equivalents, the
largest shortage in a decade. The unmet demand for numbers of additional
physicians is greatest on the largely urban island of O’ahu, however
O’ahu’s neighboring islands have the largest shortages by percentage of
demand. In fact, Hawai’i island has over a 50% shortage of physicians
for the first time since the supply has been calculated starting in
2010. Primary care has the greatest demand with a statewide shortage of
412 full time equivalents. The average age of physicians in Hawai’i is
54 compared to the national average of 52. The authors estimate that
more than 52% of providers are utilizing telehealth and that 10% of
providers have retired or closed their practices since the start of the
COVID-19 pandemic. Hawai’i is now in an urgent state of need for
recruitment and retention of physicians. ©Copyright 2022 by University
Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Healthcare (Basel,
Switzerland) \|Since the onset of the severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2) pandemic, various potential targeted
therapies for SARS-CoV-2 infection have been proposed. The protective
effects of mineralocorticoid receptor antagonists (MRA) against tissue
fibrosis, pulmonary and systemic vasoconstriction, and inflammation have
been implicated in potentially attenuating the severity of SARS-CoV-2
infection by inhibiting the deleterious effects of aldosterone.
Furthermore, spironolactone, a type of MRA, has been suggested to have a
beneficial effect on SARS-CoV-2 outcomes through its dual action as an
MRA and antiandrogen, resulting in reduced transmembrane protease
receptor serine type 2 (TMPRSS2)-related viral entry to host cells. In
this study, we sought to investigate the association between MRA
antagonist therapy and mortality in SARS-CoV-2 patients via systematic
review and meta-analysis. The systematic review was performed according
to the Preferred Reporting Items for Systematic Reviews and
Meta-Analyses (PRISMA) guidelines. MEDLINE and EMBASE databases were
searched for studies that reported the incidence of mortality in
patients on MRA with SARS-CoV-2 infection. Pooled odds ratio (OR) and
95% confidence interval (CI) of the outcome were obtained using the
random-effects model. Five studies with a total of 1,388,178 subjects
(80,903 subjects receiving MRA therapy) met the inclusion criteria. We
included studies with all types of MRA therapy including spironolactone
and canrenone and found no association between MRA therapy and mortality
in SARS-CoV-2 infection (OR = 0.387, 95% CI: 0.134-1.117, p = 0.079). \|
\|Hawai’i journal of health & social welfare \|A mixed-methods study was
performed to identify the physical and emotional needs of
Hawai`i health care workers during the COVID-19 pandemic, and the degree to which these needs are being met by their clinic or hospital. Qualitative interviews and demographic surveys were conducted with two cohorts of health care workers. Cohort 1 (N=15) was interviewed between July 20 - August 7, 2020, and Cohort 2 (N=16) between September 28 - October 9, 2020. A thematic analysis of the interview data was then performed. Participants' primary concern was contracting the illness at work and transmitting it to their families. Solo practitioners working in outpatient clinics reported more financial challenges and greater difficulty obtaining PPE than those employed by hospitals or group practices. While telehealth visits increased for both inpatient and out-patient settings, the new visit type introduced new barriers to entry for patients. The study findings may serve to better understand the effect of COVID-19 on health care workers and support the development of hospital and clinic procedures. Further research into the impacts of COVID-19 on nurses in Hawai`i
is recommended. ©Copyright 2022 by University Health Partners of Hawai‘i
(UHP Hawai‘i). \| \|The American journal of tropical medicine and
hygiene \|Severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2)-associated pancytopenia is a known but rare complication of
COVID-19 syndrome that is not well described in literature. Severe acute
respiratory syndrome coronavirus 2 has shown the potential to affect any
organ including the bone marrow, which then results in a decrease in all
three blood cell lines. These cases usually resolve with the passage of
time and treatment of underlying risk factors. As COVID pneumonia rates
continue to increase worldwide, it is crucial to be able to recognize
this complication. Additionally, deeper investigation into patient’s
response to COVID infection can be complicated by unexpected underlying
disease. We report a case of a symptomatic 24-year-old active duty male
in Hawaii with post-COVID pancytopenia that was found to have previously
undiagnosed pernicious anemia and his response to standard treatment. \|
\|Marine policy \|The human response to the COVID-19 pandemic set in
motion an unprecedented shift in human activity with unknown long-term
effects. The impacts in marine systems are expected to be highly dynamic
at local and global scales. However, in comparison to terrestrial
ecosystems, we are not well-prepared to document these changes in marine
and coastal environments. The problems are two-fold: 1) manual and
siloed data collection and processing, and 2) reliance on marine
professionals for observation and analysis. These problems are relevant
beyond the pandemic and are a barrier to understanding rapidly evolving
blue economies, the impacts of climate change, and the many other
changes our modern-day oceans are undergoing. The “Our Ocean in
COVID-19″ project, which aims to track human-ocean interactions
throughout the pandemic, uses the new eOceans platform (eOceans.app) to
overcome these barriers. Working at local scales, a global network of
ocean scientists and citizen scientists are collaborating to monitor the
ocean in near real-time. The purpose of this paper is to bring this
project to the attention of the marine conservation community,
researchers, and the public wanting to track changes in their area. As
our team continues to grow, this project will provide important
baselines and temporal patterns for ocean conservation, policy, and
innovation as society transitions towards a new normal. It may also
provide a proof-of-concept for real-time, collaborative ocean monitoring
that breaks down silos between academia, government, and at-sea
stakeholders to create a stronger and more democratic blue economy with
communities more resilient to ocean and global change. © 2022 Elsevier
Ltd. All rights reserved. \| \|Climate dynamics \|Anthropogenic
emissions decreased dramatically during the COVID-19 pandemic, but its
possible effect on monsoon is unclear. Based on coupled models
participating in the COVID Model Intercomparison Project (COVID-MIP), we
show modeling evidence that the East Asian summer monsoon (EASM) is
enhanced by 2.2% in terms of precipitation and by 5.4% in terms of the
southerly wind at lower troposphere, and the amplitude of the forced
response reaches about 1/3 of the standard deviation for interannual
variability. The enhanced EASM during COVID-19 pandemic is a fast
response to reduced aerosols, which is confirmed by the simulated
response to the removal of all anthropogenic aerosols. The observational
evidence, i.e., the anomalously strong EASM observed in 2020 and 2021,
also supports the simulated enhancement of EASM. The essential mechanism
for the enhanced EASM in response to COVID-19 is the enhanced zonal
thermal contrast between Asian continent and the western North Pacific
in the troposphere, due to the reduced aerosol concentration over Asian
continent and the associated latent heating feedback. As the enhancement
of EASM is a fast response to the reduction in aerosols, the effect of
COVID-19 on EASM dampens soon after the rebound of emissions based on
the models participating in COVID-MIP. The online version contains
supplementary material available at 10.1007/s00382-022-06247-8. © The
Author(s), under exclusive licence to Springer-Verlag GmbH Germany, part
of Springer Nature 2022. \| \|Innovations in clinical neuroscience \|The
authors explore the impact of cumulative stress on United States (US)
military service members (SM), including soldiers and medical personnel,
deployed to serve in New York City (NYC) communities. Their mission was
to assist in establishing emergency field hospitals during the COVID-19
pandemic. Causative biopsychosocial factors are presented, as well as
the impact of wellness checks, which were utilized to monitor the mood
and morale of frontline healthcare providers, military personnel, and
infected patients in a 2,500-bed emergency field hospital and a
1,000-bed Naval hospital ship operating in the metropolitan NYC area.
The authors introduce a self-care and wellness tool, which assesses five
core domains (physical, mental, emotional, social, and spiritual) for
the purpose of assessing and improving individual overall well-being
during periods of heightened stress. This instrument could aid attending
medical personnel in identifying patients at risk of suicide. Likewise,
the utility of this self-care tool is applicable to both military SM and
civilians, and includes soldiers and medical personnel. Copyright ©
2022. Matrix Medical Communications. All rights reserved. \| \|Women’s
midlife health \|The COVID-19 pandemic presented challenges that
disproportionately impacted women. Household roles typically performed
by women (such as resource acquisition and caretaking) became more
difficult due to financial strain, fear of infection, and limited
childcare options among other concerns. This research draws from an
on-going study of hot flashes and brown adipose tissue to examine the
health-related effects of the COVID-19 pandemic among 162 women aged
45-55 living in western Massachusetts. We compared women who
participated in the study pre- and early pandemic with women who
participated mid-pandemic and later-pandemic (when vaccines became
widely available). We collected self-reported symptom frequencies (e.g.,
aches/stiffness in joints, irritability), and assessments of stress,
depression, and physical activity through questionnaires as well as
measures of adiposity (BMI and percent body fat). Additionally, we asked
open-ended questions about how the pandemic influenced women’s health
and experience of menopause. Comparisons across pre-/early, mid-, and
later pandemic categories were carried out using ANOVA and Chi-square
analyses as appropriate. The Levene test for homogeneity of variances
was examined prior to each ANOVA. Open-ended questions were analyzed for
yes/no responses and general themes. Contrary to our hypothesis that
women would suffer negative health-related consequences during the
COVID-19 pandemic, we found no significant differences in women’s
health-related measures or physical activity across the pandemic.
However, our analysis of open-ended responses revealed a bi-modal
distribution of answers that sheds light on our unexpected findings.
While some women reported higher levels of stress and anxiety and lower
levels of physical activity, other women reported benefitting from the
remote life that the pandemic imposed and described having more time to
spend on physical activity or in quality time with their families. In
this cross-sectional comparison of women during the pre-/early, mid-,
and later-pandemic, we found no significant differences across means in
multiple health-related variables. However, open-ended questions
revealed that while some women suffered health-related effects during
the pandemic, others experienced conditions that improved their health
and well-being. The differential results of this study highlight a need
for more nuanced and intersectional research on risk, vulnerabilities,
and coping among mid-life women. © 2022. The Author(s). \| \|American
journal of lifestyle medicine \|Chronic health conditions related to
diet are linked with increased risk for COVID-19 infection,
complications, and mortality. Adherence to a healthy diet pattern can be
protective, but a major barrier to healthy eating is the high cost of
healthy foods. Access to healthy foods is especially limited in
households that experience food insecurity, not having enough food or
resources to get food. Individuals who live in these households are also
at increased risk for a number of health conditions. Addressing food
insecurity within lifestyle medicine practice is needed to achieve
optimal nutrition status. Emerging food as medicine and other food
access programs are promising but coverage of such programs is lacking
through healthcare insurers. Medicaid waivers are a potential solution
and have been utilized in a handful of states. Copyright © 2022 The
Author(s). \| \|Public health nursing (Boston, Mass.) \|The COVID-19
pandemic has resulted in major disruption to economic, health,
education, and social systems. Families with preschool children
experienced extraordinary strain during this time. This paper describes
a qualitative study examining the experience of parents of preschool
children in Hawaii during the COVID-19 pandemic. Thirteen (N = 13)
parents of preschool children living on the island of Oahu, Hawaii,
participated in small group discussions occurring in February and March
2021, approximately 1 year after the start of the pandemic in the state.
Discussion transcripts were coded and sorted into themes. Four themes
emerged: stressors due to the COVID-19 pandemic, family coping and
resources, meaning of the COVID-19 crisis to the family, and family
adaptation patterns. Themes mapped to the Family Adjustment and
Adaptation Response model. Families relied on various resources to cope
with stressors experienced due to the COVID-19 pandemic, and adopted new
patterns related to seeking healthcare and household emergency
preparedness. Findings may inform policies and interventions to support
families during the ongoing COVID-19 pandemic and future public health
emergencies. © 2022 Wiley Periodicals LLC. \| \|Academic medicine :
journal of the Association of American Medical Colleges \|Asian
American, Native Hawaiian, and Pacific Islander (AANHPI) populations are
growing rapidly in the United States, yet AANHPIs remain understudied,
overlooked, and misunderstood. During the COVID-19 pandemic, themes from
the tragic history of anti-Asian bias and marginalization have
resurfaced in a surge of renewed bigotry and xenophobic violence against
AANHPIs. In this commentary, the authors discuss the role of medical
schools in combating anti-Asian sentiment as an important step toward
achieving health equity. Based on their collective expertise in health
disparities research, medical education, and policy, they offer
suggestions about how to disrupt the pattern of invisibility and
exclusion faced by AANHPI populations. They consider ways that
representative data, leadership in medical education, research funding,
national policies, and broad partnerships can help address AANHPI health
disparities. Copyright © 2022 by the Association of American Medical
Colleges. \| \|JAMA internal medicine \|Screening for medication
abortion eligibility typically includes ultrasonography or pelvic
examination. To reduce physical contact during the COVID-19 pandemic,
many clinicians stopped requiring tests before medication abortion and
instead screened patients for pregnancy duration and ectopic pregnancy
risk by history alone. However, few US-based studies have been conducted
on the outcomes and safety of this novel model of care. To evaluate the
outcomes and safety of a history-based screening, no-test approach to
medication abortion care. This retrospective cohort study included
patients obtaining a medication abortion without preabortion
ultrasonography or pelvic examination between February 1, 2020, and
January 31, 2021, at 14 independent, Planned Parenthood,
academic-affiliated, and online-only clinics throughout the US.
Medications for abortion provided without preabortion ultrasonography or
pelvic examination and dispensed to patients in person or by mail.
Effectiveness, defined as complete abortion after 200 μg of mifepristone
and up to 1600 μg of misoprostol without additional intervention, and
major abortion-related adverse events, defined as hospital admission,
major surgery, or blood transfusion. The study included data on 3779
patients with eligible abortions. The study participants were racially
and ethnically diverse and included 870 (23.0%) Black patients, 533
(14.1%) Latinx/Hispanic patients, 1623 (42.9%) White patients, and 327
(8.7%) who identified as multiracial or with other racial or ethnic
groups. For most (2626 \[69.5%\]), it was their first medication
abortion. Patients lived in 34 states, and 2785 (73.7%) lived in urban
areas. In 2511 (66.4%) abortions, the medications were dispensed in
person; in the other 1268 (33.6%), they were mailed to the patient.
Follow-up data were obtained for 2825 abortions (74.8%), and multiple
imputation was used to account for missing data. Across the sample, 12
abortions (0.54%; 95% CI, 0.18%-0.90%) were followed by major
abortion-related adverse events, and 4 patients (0.22%; 95% CI,
0.00%-0.45%) were treated for ectopic pregnancies. Follow-up identified
9 (0.40%; 95% CI, 0.00%-0.84%) patients who had pregnancy durations of
greater than 70 days on the date the mifepristone was dispensed that
were not identified at screening. The adjusted effectiveness rate was
94.8% (95% CI, 93.6%-95.9%). Effectiveness was similar when medications
were dispensed in person (95.4%; 95% CI, 94.1%-96.7%) or mailed (93.3%;
95% CI, 90.7%-95.9%). In this cohort study, screening for medication
abortion eligibility by history alone was effective and safe with either
in-person dispensing or mailing of medications, resulting in outcomes
similar to published rates of models involving ultrasonography or pelvic
examination. This approach may facilitate more equitable access to this
essential service by increasing the types of clinicians and locations
offering abortion care. \| \|BMJ (Clinical research ed.) \|NA \|
\|Frontiers in public health \|Native Hawaiians are proud and resilient
people who have endured significant impacts from colonization. Despite
being in a time of vibrant cultural revitalization, Native Hawaiians
have a shorter life expectancy than other racial and ethnic groups in
Hawai’i. The primary aim of this paper was to share data from the first
year of a 5-year study with Native Hawaiian kūpuna (elders) on their
experiences with healthcare, along with barriers to accessing
healthcare. Ten kūpuna living in rural areas of Hawai’i participated in
three interviews each, which were held in an informal, talk-story style.
The first interview focused on establishing rapport. The second
interview focused on the kūpuna’s strengths, resiliencies, and what they
would like to pass to the next generation. The third interview focused
on the elders’ experiences with healthcare, which is the focus of this
paper. All ten kūpuna reported growing up with limited access to Western
healthcare; rather, their families successfully treated many illnesses
and injuries with lā’au lapa’au (Hawaiian herbal medicine) and other
traditional healing practices, as they had done for generations. As
Western medicine became more prevalent and accessible, they used both,
but many preferred holistic treatments such as prayer, a return to the
traditional diet, and lā’au lapa’au. As a group, the kūpuna rated their
health as fair to good; two had diabetes, two had cardiovascular
disease, four had neuropathies, and five were cancer survivors. The
kūpuna reported high turnover among providers in rural communities.
Limited access to specialists often required them to travel to Honolulu
for care, which was costly and especially difficult during coronavirus
disease 2019 (COVID-19). Regardless of provider ethnicity, the kūpuna
appreciated those who took the time to get to know them as people and
respected Hawaiian cultural practices. They advised that Western
providers speak honestly and directly, have compassion, and build
connections to patients and their communities. Copyright © 2022
Kawakami, Muneoka, Burrage, Tanoue, Haitsuka and Braun. \| \|ACS
infectious diseases \|FDA-approved and emergency use-authorized vaccines
using new mRNA and viral-vector technology are highly effective in
preventing moderate to severe disease; however, information on their
long-term efficacy and protective breadth against severe acute
respiratory syndrome coronavirus 2 variants of concern (VOCs) is
currently scarce. Here, we describe the durability and broad-spectrum
VOC immunity of a prefusion-stabilized spike (S) protein adjuvanted with
liquid or lyophilized CoVaccine HT in cynomolgus macaques. This
recombinant subunit vaccine is highly immunogenic and induces robust
spike-specific and broadly neutralizing antibody responses effective
against circulating VOCs (B.1.351 \[Beta\], P.1 \[Gamma\], and B.1.617
\[Delta\]) for at least three months after the final boost. Protective
efficacy and postexposure immunity were evaluated using a heterologous
P.1 challenge nearly three months after the last immunization. Our
results indicate that while immunization with both high and low S doses
shorten and reduce viral loads in the upper and lower respiratory tract,
a higher antigen dose is required to provide durable protection against
disease as vaccine immunity wanes. Histologically, P.1 infection causes
similar COVID-19-like lung pathology as seen with early pandemic
isolates. Postchallenge IgG concentrations were restored to peak
immunity levels, and vaccine-matched and cross-variant neutralizing
antibodies were significantly elevated in immunized macaques indicating
an efficient anamnestic response. Only low levels of P.1-specific
neutralizing antibodies with limited breadth were observed in control
(nonvaccinated but challenged) macaques, suggesting that natural
infection may not prevent reinfection by other VOCs. Overall, these
results demonstrate that a properly dosed and adjuvanted recombinant
subunit vaccine can provide protective immunity against circulating VOCs
for at least three months. \| \|Wellcome open research \|Background:
Industrialised countries had varied responses to the COVID-19 pandemic,
which may lead to different death tolls from COVID-19 and other
diseases. Methods: We applied an ensemble of 16 Bayesian probabilistic
models to vital statistics data to estimate the number of weekly deaths
if the pandemic had not occurred for 40 industrialised countries and US
states from mid-February 2020 through mid-February 2021. We subtracted
these estimates from the actual number of deaths to calculate the
impacts of the pandemic on all-cause mortality. Results: Over this year,
there were 1,410,300 (95% credible interval 1,267,600-1,579,200) excess
deaths in these countries, equivalent to a 15% (14-17) increase, and 141
(127-158) additional deaths per 100,000 people. In Iceland, Australia
and New Zealand, mortality was lower than would be expected in the
absence of the pandemic, while South Korea and Norway experienced no
detectable change. The USA, Czechia, Slovakia and Poland experienced
\>20% higher mortality. Within the USA, Hawaii experienced no detectable
change in mortality and Maine a 5% increase, contrasting with New
Jersey, Arizona, Mississippi, Texas, California, Louisiana and New York
which experienced \>25% higher mortality. Mid-February to the end of May
2020 accounted for over half of excess deaths in Scotland, Spain,
England and Wales, Canada, Sweden, Belgium, the Netherlands and Cyprus,
whereas mid-September 2020 to mid-February 2021 accounted for \>90% of
excess deaths in Bulgaria, Croatia, Czechia, Hungary, Latvia,
Montenegro, Poland, Slovakia and Slovenia. In USA, excess deaths in the
northeast were driven mainly by the first wave, in southern and
southwestern states by the summer wave, and in the northern plains by
the post-September period. Conclusions: Prior to widespread
vaccine-acquired immunity, minimising the overall death toll of the
pandemic requires policies and non-pharmaceutical interventions that
delay and reduce infections, effective treatments for infected patients,
and mechanisms to continue routine health care. Copyright: © 2022 Kontis
V et al. \| \|Pacing and clinical electrophysiology : PACE \|COVID-19
has recently been associated with the development of bradyarrhythmias,
although its mechanism is still unclear. We aim to summarize the
existing evidence regarding bradyarrhythmia in COVID-19 and provide
future directions for research. Following the PRISMA Extension for
Scoping Reviews, we searched MEDLINE and EMBASE for all peer-reviewed
articles using keywords including”Bradycardia,” “atrioventricular
block,” and “COVID-19″ from their inception to October 13, 2021.
Forty-three articles, including 11 observational studies and 59 cases
from case reports and series, were included in the systematic review.
Although some observational studies reported increased mortality in
those with bradyarrhythmia and COVID-19, the lack of comparative groups
and small sample sizes hinder the ability to draw definitive
conclusions. Among 59 COVID-19 patients with bradycardia from case
reports and series, bradycardia most often occurred in those with severe
or critical COVID-19, and complete heart block occurred in the majority
of cases despite preserved LVEF (55.9%). Pacemaker insertion was
required in 76.3% of the patients, most of which were permanent implants
(45.8%). This systematic review summarizes the current evidence and
characteristics of bradyarrhythmia in patients with COVID-19. Further
studies are critical to assess the reversibility of bradyarrhythmia in
COVID-19 patients and to clarify potential therapeutic targets including
the need for permanent pacing. © 2022 Wiley Periodicals LLC. \| \|AIDS
care \|City lockdown is critical to successfully contain the COVID-19
pandemic. The impact of lockdown and COVID-19 pandemic on healthcare
among vulnerable population has yet to be explicated. A cross-sectional
survey was conducted among HIV-infected men who have sex with men (MSM)
in Wuhan with city lockdown and Shanghai without lockdown, and
healthcare interruptions were evaluated and compared. A logistic
regression analysis was employed to examine associates of HIV-related
healthcare interruptions and compromised mental health. Compared to
participants in Shanghai (N = 440), HIV-infected MSM in Wuhan (N = 503)
had significantly higher proportion of untimely availability of
antiretroviral drugs (ARVs) (20.6% vs. 8.4%), obtaining ARVs from
outside institutions (29.1% vs. 8.1%), postponed non-AIDS treatment
(6.4% vs. 2.8%) and untimely follow-up appointments (33.4% vs. 14.5%).
HIV-related healthcare interruptions were positively associated with
lockdown (OR = 4.89, 95% CI: 3.49-6.85) and non-local residence (OR =
1.91, 95% CI: 1.37-2.64). Compromised mental health, including insomnia
and generalized anxiety disorders, was associated with non-local
residence (OR = 1.35, 95% CI: 1.01-1.81) and healthcare interruptions
(OR = 1.34, 95% CI: 1.01-1.79). HIV-infected MSM are vulnerable to
healthcare interruptions and mental health problems during the COVID-19
pandemic, underscoring the need for tailored intervention strategies to
minimize deleterious health consequences. \| \|Hawai’i journal of health
& social welfare \|NA \| \|American journal of ophthalmology case
reports \|Optic capture of sutured scleral fixated posterior chamber
intraocular lenses (PC IOLs) is an occasional complication resulting in
blurred vision and discomfort. A retrospective study of the management
of 18 eyes (3.6%) with optic capture out of 495 eyes with scleral
fixated IOLs during the study period. 54 procedures were performed in
the management of optic capture of sutured scleral fixated PC IOLs. An
in-office technique was utilized to relieve the optic capture by
repositioning the optic posterior to the iris. This technique was
performed after topical anesthesia and topical 5% betadine with the
patient stably positioned at the slit lamp. Using a 30-gauge needle,
sometimes after a 15-degree paracentesis blade, the needle was advanced
in a parallel plane above the iris until the tip reached the edge of the
captured optic. The optic is engaged in the inferior periphery away from
the central visual axis, and pushed gently posteriorly just enough to
reposition the optic posterior to the iris. In some cases, pilocarpine
2% drops were utilized after the procedure to decrease the risk of
recapture of the optic. All 54 procedures were successfully performed in
the office without significant pain or discomfort. Vision before optic
capture, during optic capture, and at the first office visit after optic
capture were comparable. There were not any cases of endophthalmitis,
hyphema, iris trauma, iris prolapse or keratitis. While eight patients
only had one episode of optic capture, 10 patients had multiple episodes
of optic capture, all managed with this in office procedure. Recurrent
optic capture occurred more frequently in eyes with fixation at less
than 2 mm from the limbus than eyes with scleral fixation at 2 mm from
the limbus. Reposition of the optic after pupillary capture of a scleral
fixated PC IOL can be successfully performed in the office without
discomfort or significant complications and is an alternative management
option to a return to the operating room. This procedure may be
especially important when there is poor access to the operating room or
restricted access to the operating room as during the COVID19 pandemic.
© 2022 Published by Elsevier Inc. \| \|JAMA network open \|This
cross-sectional study analyzes the association between vaccination rates
and COVID-19 incidence by county from July to August 2021. \| \|Open
forum infectious diseases \|Nasopharyngeal (NP) swabs are the standard
for SARS-CoV-2 diagnosis. If less invasive alternatives to NP swabs (eg,
oropharyngeal \[OP\] or nasal swabs \[NS\]) are comparably sensitive,
the use of these techniques may be preferable in terms of comfort,
convenience, and safety. This study compared the detection of SARS-CoV-2
in swab samples collected on the same day among participants with at
least one positive PCR test. Overall, 755 participants had at least one
set of paired swabs. Concordance between NP and other swab types was 75%
(NS), 72% (OP), 54% (rectal swabs \[RS\]), and 78% (NS/OP combined).
Kappa values were moderate for the NS, OP, and NS/OP comparisons (0.50,
0.45, and 0.54, respectively). Highest sensitivity relative to NP (0.87)
was observed with a combination of NS/OP tests (positive if either NS or
OP was positive). Sensitivity of the non-NP swab types was highest in
the first week postsymptom onset and decreased thereafter. Similarly,
virus RNA quantity was highest in the NP swabs as compared with NS, OP,
and RS within two weeks postsymptom onset. OP and NS performance
decreased as virus RNA quantity decreased. No differences were noted
between NS specimens collected at home or in clinic. NP swabs detected
more SARS-CoV-2 cases than non-NP swabs, and the sensitivity of the
non-NP swabs decreased with time postsymptom onset. While other swabs
may be simpler to collect, NP swabs present the best chance of detecting
SARS-CoV-2 RNA, which is essential for clinical care as well as genomic
surveillance. Published by Oxford University Press on behalf of
Infectious Diseases Society of America 2021. \| \|The journal of allergy
and clinical immunology. In practice \|NA \| \|PloS one \|Health
inequalities based on race are well-documented, and the COVID-19
pandemic is no exception. Despite the advances in modern medicine,
access to health care remains a primary determinant of health outcomes,
especially for communities of color. African-Americans and other
minorities are disproportionately at risk for infection with COVID-19,
but this problem extends beyond access alone. This study sought to
identify trends in race-based disparities in COVID-19 in the setting of
universal access to care. Tripler Army Medical Center (TAMC) is a
Department of Defense Military Treatment Facility (DoD-MTF) that
provides full access to healthcare to active duty military members,
beneficiaries, and veterans. We evaluated the characteristics of
individuals diagnosed with SARS-CoV-2 infection at TAMC in a
retrospective, case-controlled (1:1) study. Most patients (69%) had
received a COVID-19 test within 3 days of symptom onset. Multivariable
logistic regression analyses were used to identify factors associated
with testing positive and to estimate adjusted odds ratios.
African-American patients and patients who identified as”Other”
ethnicities were two times more likely to test positive for SARS-CoV-2
relative to Caucasian patients. Other factors associated with testing
positive include: younger age, male gender, previous positive test,
presenting with \>3 symptoms, close contact with a COVID-19 positive
patient, and being a member of the US Navy. African-Americans and
patients who identify as “Other” ethnicities had disproportionately
higher rates of positivity of COVID-19. Although other factors
contribute to increased test positivity across all patient populations,
access to care does not appear to itself explain this discrepancy with
COVID-19. \| \|Forensic sciences research \|NA \| \|Critical care
medicine \|Body temperature trajectories of infected patients are
associated with specific immune profiles and survival. We determined the
association between temperature trajectories and distinct manifestations
of coronavirus disease 2019. Retrospective observational study. Four
hospitals within an academic healthcare system from March 2020 to
February 2021. All adult patients hospitalized with coronavirus disease
2019. Using a validated group-based trajectory model, we classified
patients into four previously defined temperature trajectory
subphenotypes using oral temperature measurements from the first 72
hours of hospitalization. Clinical characteristics, biomarkers, and
outcomes were compared between subphenotypes. The 5,903 hospitalized
coronavirus disease 2019 patients were classified into four
subphenotypes: hyperthermic slow resolvers (n = 1,452, 25%),
hyperthermic fast resolvers (1,469, 25%), normothermics (2,126, 36%),
and hypothermics (856, 15%). Hypothermics had abnormal coagulation
markers, with the highest d-dimer and fibrin monomers (p \< 0.001) and
the highest prevalence of cerebrovascular accidents (10%, p = 0.001).
The prevalence of venous thromboembolism was significantly different
between subphenotypes (p = 0.005), with the highest rate in hypothermics
(8.5%) and lowest in hyperthermic slow resolvers (5.1%). Hyperthermic
slow resolvers had abnormal inflammatory markers, with the highest
C-reactive protein, ferritin, and interleukin-6 (p \< 0.001).
Hyperthermic slow resolvers had increased odds of mechanical
ventilation, vasopressors, and 30-day inpatient mortality (odds ratio,
1.58; 95% CI, 1.13-2.19) compared with hyperthermic fast resolvers. Over
the course of the pandemic, we observed a drastic decrease in the
prevalence of hyperthermic slow resolvers, from representing 53% of
admissions in March 2020 to less than 15% by 2021. We found that
dexamethasone use was associated with significant reduction in
probability of hyperthermic slow resolvers membership (27% reduction;
95% CI, 23-31%; p \< 0.001). Hypothermics had abnormal coagulation
markers, suggesting a hypercoagulable subphenotype. Hyperthermic slow
resolvers had elevated inflammatory markers and the highest odds of
mortality, suggesting a hyperinflammatory subphenotype. Future work
should investigate whether temperature subphenotypes benefit from
targeted antithrombotic and anti-inflammatory strategies. Copyright ©
2022 by the Society of Critical Care Medicine and Wolters Kluwer Health,
Inc. All Rights Reserved. \| \|PloS one \|Racial inequities in
Coronavirus 2019 (COVID-19) have been reported over the course of the
pandemic, with Black, Hispanic/Latinx, and Native American individuals
suffering higher case rates and more fatalities than their White
counterparts. We used a unique statewide dataset of confirmed COVID-19
cases across Missouri, linked with historical statewide hospital data.
We examined differences by race and ethnicity in raw population-based
case and mortality rates. We used patient-level regression analyses to
calculate the odds of mortality based on race and ethnicity, controlling
for comorbidities and other risk factors. As of September 10, 2020 there
were 73,635 confirmed COVID-19 cases in the State of Missouri. Among the
64,526 case records (87.7% of all cases) that merged with prior
demographic and health care utilization data, 12,946 (20.1%) were
Non-Hispanic (NH) Black, 44,550 (69.0%) were NH White, 3,822 (5.9%) were
NH Other/Unknown race, and 3,208 (5.0%) were Hispanic. Raw cumulative
case rates for NH Black individuals were 1,713 per 100,000 population,
compared with 2,095 for NH Other/Unknown, 903 for NH White, and 1,218
for Hispanic. Cumulative COVID-19-related death rates for NH Black
individuals were 58.3 per 100,000 population, compared with 38.9 for NH
Other/Unknown, 19.4 for NH White, and 14.8 for Hispanic. In a model that
included insurance source, history of a social determinant billing code
in the patient’s claims, census block travel change, population density,
Area Deprivation Index, and clinical comorbidities, NH Black race (OR
1.75, 1.51-2.04, p\<0.001) and NH Other/Unknown race (OR 1.83,
1.36-2.46, p\<0.001) remained strongly associated with mortality. In
Missouri, COVID-19 case rates and mortality rates were markedly higher
among NH Black and NH Other/Unknown race than among NH White residents,
even after accounting for social and clinical risk, population density,
and travel patterns during COVID-19. \| \|Neurology international \|In
the approximately two years since the emergence of COVID-19 (Coronavirus
Disease 2019) myriad neurological symptoms have been reported that are
seemingly unrelated to each other \[…\]. \| \|Journal of clinical
medicine research \|As the coronavirus disease 2019 (COVID-19) pandemic
is still underway, a range of clinical presentations and pathologies
continue to present themselves in unexpected ways. One such pathology is
that of epiploic appendagitis, an uncommon and underdiagnosed cause of
acute abdominal pain. We present the case of a 50-something-year-old
male who presented with left lower quadrant abdominal pain in the
setting of acute COVID-19 infection, found to have acute epiploic
appendagitis. After persistent moderate to severe abdominal pain,
epiploic appendagitis was diagnosed by computed tomography (CT) imaging
findings. The patient was managed for his COVID-19 pneumonia over the
course of his hospitalization, as well as conservatively managed with
pain control measures for his epiploic appendagitis. This is the second
reported case in the literature to the best of our knowledge that shares
the case of acute epiploic appendagitis in a patient presenting with
acute abdominal pain, who is also found to be COVID-19-positive.
Procoagulant changes in coagulation pathways are found in patients with
severe COVID-19, and contribute to venous thromboembolism in this
patient population. Diagnosing and conservatively managing epiploic
appendagitis will lead to decreasing misdiagnosis, preventing invasive
or inappropriate treatments that may increase harm to patients, and more
adequately understanding the complications associated with COVID-19.
Copyright 2021, Kaakour et al. \| \|International journal of
endocrinology \|Type 2 diabetes (T2D) as a worldwide chronic disease
combined with the COVID-19 pandemic prompts the need for improving the
management of hospitalized COVID-19 patients with preexisting T2D to
reduce complications and the risk of death. This study aimed to identify
clinical factors associated with COVID-19 outcomes specifically targeted
at T2D patients and build an individualized risk prediction nomogram for
risk stratification and early clinical intervention to reduce mortality.
In this retrospective study, the clinical characteristics of 382
confirmed COVID-19 patients, consisting of 108 with and 274 without
preexisting T2D, from January 8 to March 7, 2020, in Tianyou Hospital in
Wuhan, China, were collected and analyzed. Univariate and multivariate
Cox regression models were performed to identify specific clinical
factors associated with mortality of COVID-19 patients with T2D. An
individualized risk prediction nomogram was developed and evaluated by
discrimination and calibration. Nearly 15% (16/108) of hospitalized
COVID-19 patients with T2D died. Twelve risk factors predictive of
mortality were identified. Older age (HR = 1.076, 95% CI = 1.014-1.143,
p=0.016), elevated glucose level (HR = 1.153, 95% CI = 1.038-1.28,
p=0.0079), increased serum amyloid A (SAA) (HR = 1.007, 95% CI =
1.001-1.014, p=0.022), diabetes treatment with only oral diabetes
medication (HR = 0.152, 95%CI = 0.032-0.73, p=0.0036), and oral
medication plus insulin (HR = 0.095, 95%CI = 0.019-0.462, p=0.019) were
independent prognostic factors. A nomogram based on these prognostic
factors was built for early prediction of 7-day, 14-day, and 21-day
survival of diabetes patients. High concordance index (C-index) was
achieved, and the calibration curves showed the model had good
prediction ability within three weeks of COVID-19 onset. By
incorporating specific prognostic factors, this study provided a
user-friendly graphical risk prediction tool for clinicians to quickly
identify high-risk T2D patients hospitalized for COVID-19. Copyright ©
2022 Yuanyuan Fu et al. \| \|Hawai’i journal of health & social welfare
\|In March 2020, Hawai’i instituted public health measures to prevent
the spread of Coronavirus disease 2019 (COVID-19), including
stay-at-home orders, closure of non-essential businesses and parks, use
of facial coverings, social distancing, and a mandatory 14-day
quarantine for travelers. In response to these measures, Hawai’i Pacific
Neuroscience (HPN) modified practice processes to ensure continuity of
neurological treatment. A survey of patients was performed to assess the
impact of the COVID-19 pandemic and pandemic-related practice processes
for quality improvement. Overall, 367 patients seen at HPN between April
22, 2020, and May 18, 2020, were surveyed via telephone. Almost half
(49.6%) participated in a telemedicine appointment, with the majority
finding it easy to use (87.4%) and as valuable as face-to-face
appointments (68.7%). Many (44.5%) patients said they would have missed
a health care appointment without the availability of telemedicine, and
47.3% indicated they might prefer to use telemedicine over in-person
appointments in the future. Many reported new or worsening mental health
problems, including depression (27.6%), anxiety (38.3%), or sleep
disturbances (37.4%). A significant number reported worsening of their
condition, with 33.1% of patients who experience migraines reporting
increased symptom severity or frequency, 45.8% patients with Alzheimer’s
disease reporting worsened symptoms, 38.5% of patients with Parkinson’s
disease who had a recent fall, and 50.0% of patients with multiple
sclerosis experiencing new or worsened symptoms. Insights from this
survey applied to the practice’s pandemic-related processes include
emphasizing lifestyle modification, screening for changes in mental
health, optimizing treatment plans, and continuing the option of
telemedicine. ©Copyright 2022 by University Health Partners of Hawai‘i
(UHP Hawai‘i). \| \|Journal of biomolecular techniques : JBT
\|Controlling the course of the Coronavirus Disease 2019 (COVID-19)
pandemic will require widespread deployment of consistent and accurate
diagnostic testing of the novel Severe Acute Respiratory Syndrome
Coronavirus 2 (SARS-CoV-2). Ideally, tests should detect a minimum viral
load, be minimally invasive, and provide a rapid and simple readout.
Current Food and Drug Administration (FDA)-approved RT-qPCR-based
standard diagnostic approaches require invasive nasopharyngeal swabs and
involve laboratory-based analyses that can delay results. Recently, a
loop-mediated isothermal nucleic acid amplification (LAMP) test that
utilizes colorimetric readout received FDA approval. This approach
utilizes a pH indicator dye to detect drop in pH from nucleotide
hydrolysis during nucleic acid amplification. This method has only been
approved for use with RNA extracted from clinical specimens collected
via nasopharyngeal swabs. In this study, we developed a quantitative
LAMP-based strategy to detect SARS-CoV-2 RNA in saliva. Our detection
system distinguished positive from negative sample types using a
handheld instrument that monitors optical changes throughout the LAMP
reaction. We used this system in a streamlined LAMP testing protocol
that could be completed in less than 2 h to directly detect inactivated
SARS-CoV-2 in minimally processed saliva that bypassed RNA extraction,
with a limit of detection (LOD) of 50 genomes/reaction. The quantitative
method correctly detected virus in 100% of contrived clinical samples
spiked with inactivated SARS-CoV-2 at either 1× (50 genomes/reaction) or
2× (100 genomes/reaction) of the LOD. Importantly, the quantitative
method was based on dynamic optical changes during the reaction and was
able to correctly classify samples that were misclassified by endpoint
observation of color. © 2021 ABRF. \| \|The science of diabetes
self-management and care \|The purpose of the study was to explore
experiences of Marshallese adults related to diabetes self-care
behaviors during the COVID-19 pandemic. A qualitative descriptive design
was utilized to understand participants’ diabetes self-care behaviors
during the pandemic. Nine focus groups with 53 participants were held
via videoconference and conducted in English, Marshallese, or a mixture
of both languages. A priori codes based on diabetes self-care behaviors
provided a framework for analyzing and summarizing participant
experiences. Both increases and decreases in healthy eating and exercise
were described, with improvements in health behaviors attributed to
health education messaging via social media. Participants reported
increased stress and difficulty monitoring and managing glucose.
Difficulty obtaining medication and difficulty seeing their health care
provider regularly was reported and attributed to health care provider
availability and lack of insurance due to job loss. The study provides
significant insight into the reach of health education campaigns via
social media and provides important information about the reasons for
delays in care, which extend beyond fear of contracting COVID-19 to
structural issues. \| \|JAMA \|This study assesses hip fracture surgery
volumes among older individuals in California during the pandemic
compared with prepandemic rates. \| \|Healthcare (Basel, Switzerland)
\|Due to the unprecedented COVID-19 pandemic, there may be overuse of
telemetry monitoring compared to the pre-pandemic period. We compared
the frequency of inappropriate telemetry use in the pre-COVID-19 period
(1 November 2019 to 28 February 2020) versus the peri-COVID-19 period (1
March 2020 to 30 June 2020) at a major academic hospital in Honolulu,
Hawaii, by a retrospective chart review to assess for the
appropriateness of the telemetry orders during this period, based on the
2017 American College of Cardiology/American Heart Association
guidelines. Compared to the pre-COVID-19 period, there was a significant
increase in inappropriate telemetry use during the peri-COVID-19 period
(X2 (1, N = 11,727) = 6.59, p = 0.0103). However, there was no increase
in the proportions of respiratory failure (4.0%) or pneumonia (2.7%)
during the peri-COVID-19 period. The increase in inappropriate telemetry
use may be related to the uncertainty in clinical care and decision
making amid the pandemic of the new virus. Appropriate utilization of
telemetry monitoring is increasingly important during the pandemic due
to the limited availability of resources. Further investigation is
needed to clarify the relationship between the pandemic and trends in
telemetry ordering. \| \|JAMA network open \|This randomized clinical
trial examines the effect of digital contact tracing using smartphone
app nudges to increase downloads of Pennsylvania’s COVID Alert PA app.
\| \|PloS one \|In December 2020, the first two COVID-19 vaccines were
approved in the United States (U.S.) and recommended for distribution to
front-line personnel, including nurses. Nursing students are being
prepared to fill critical gaps in the health care workforce and have
played important supportive roles during the current pandemic. Research
has focused on vaccine intentions of current health care providers and
less is known about students’ intentions to vaccinate for COVID-19. A
national sample of undergraduate nursing students were recruited across
five nursing schools in five U.S. regions in December 2020. The survey
measured perceived risk/threat of COVID-19, COVID-19 vaccine attitudes,
perceived safety and efficacy of COVID-19 vaccines, sources for vaccine
information and level of intention to become vaccinated \[primary,
secondary (i.e., delayed), or no intention to vaccinate\]. The final
sample consisted of 772 students. The majority (83.6%) had intentions to
be vaccinated, however of those 31.1% indicated secondary intention, a
delay in intention or increased hesitancy). The strongest predictors of
primary intention were positive attitudes (OR = 6.86; CI = 4.39-10.72),
having lower safety concerns (OR = 0.26; CI = 0.18-0.36), and consulting
social media as a source of information (OR = 1.56; CI = 1.23-1.97).
Asian (OR = 0.47; CI = 0.23-0.97) and Black (OR 0.26; CI = 0.08-0.80)
students were more likely to indicate secondary intention as compared to
primary intention. Students in the Midwest were most likely to indicate
no intention as compared to secondary intention (OR = 4.6; CI =
1.32-16.11). As the first two COVID-19 vaccines were
approved/recommended in the U.S. nursing students had overall high
intentions to vaccinate. Findings can guide development of educational
interventions that reduce concerns of vaccine safety that are delivered
in a way that is supportive and affirming to minoritized populations
while being respectful of geo-political differences. \| \|Open forum
infectious diseases \|We evaluated clinical outcomes, functional burden,
and complications 1 month after coronavirus disease 2019 (COVID-19)
infection in a prospective US Military Health System (MHS) cohort of
active duty, retiree, and dependent populations using serial
patient-reported outcome surveys and electronic medical record (EMR)
review. MHS beneficiaries presenting at 9 sites across the United States
with a positive severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) test, a COVID-19-like illness, or a high-risk SARS-CoV-2
exposure were eligible for enrollment. Medical history and clinical
outcomes were collected through structured interviews and International
Classification of Diseases-based EMR review. Risk factors associated
with hospitalization were determined by multivariate logistic
regression. A total of 1202 participants were enrolled. There were 1070
laboratory-confirmed SARS-CoV-2 cases and 132 SARS-CoV-2-negative
participants. In the first month post-symptom onset among the
SARS-CoV-2-positive cases, there were 212 hospitalizations, 80%
requiring oxygen, 20 ICU admissions, and 10 deaths. Risk factors for
COVID-19-associated hospitalization included race (increased for Asian,
Black, and Hispanic compared with non-Hispanic White), age (age 45-64
and 65+ compared with \<45), and obesity (BMI≥30 compared with BMI\<30).
Over 2% of survey respondents reported the need for supplemental oxygen,
and 31% had not returned to normal daily activities at 1 month
post-symptom onset. Older age, reporting Asian, Black, or Hispanic
race/ethnicity, and obesity are associated with SARS-CoV-2
hospitalization. A proportion of acute SARS-CoV-2 infections require
long-term oxygen therapy; the impact of SARS-CoV-2 infection on
short-term functional status was substantial. A significant number of
MHS beneficiaries had not yet returned to normal activities by 1 month.
© The Author(s) 2021. Published by Oxford University Press on behalf of
Infectious Diseases Society of America. \| \|Open forum infectious
diseases \|The inFLUenza Patient-Reported Outcome Plus (FLU-PRO Plus) is
a patient-reported outcome data collection instrument assessing symptoms
of viral respiratory tract infections across 8 body systems. This study
evaluated the measurement properties of FLU-PRO Plus in a study
enrolling individuals with coronavirus disease 2019 (COVID-19). Data
from a prospective cohort study (EPICC) in US Military Health System
beneficiaries evaluated for COVID-19 was utilized. Adults with
symptomatic severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2)
infection with FLU-PRO Plus survey information within 1 week of symptom
onset were included. Reliability of FLU-PRO Plus was estimated using
intraclass correlation coefficient (ICC; 2 days’ reproducibility).
Known-groups validity was assessed using patient global assessment (PGA)
of disease severity. Patient report of return to usual health was used
to assess responsiveness (day 1-6/7). Two hundred twenty-six
SARS-CoV-2-positive participants were included in the analysis.
Reliability among those who reported no change in their symptoms from
one day to the next was high for most domains (ICC range, 0.68-0.94 for
day 1 to day 2). Construct validity was demonstrated by moderate to high
correlation between the PGA rating of disease severity and domain and
total scores (eg, total scores correlation: 0.69 \[influenza-like
illness severity\], 0.69 \[interference in daily activities\], and -0.58
\[physical health\]). In addition, FLU-PRO Plus demonstrated good
known-groups validity, with increasing domain and total scores observed
with increasing severity ratings. FLU-PRO Plus performs well in
measuring signs and symptoms in SARS-CoV-2 infection with excellent
construct validity, known-groups validity, and responsiveness to change.
Standardized data collection instruments facilitate meta-analyses,
vaccine effectiveness studies, and other COVID-19 research activities.
Published by Oxford University Press on behalf of Infectious Diseases
Society of America 2021. \| \|The Journal of rural health : official
journal of the American Rural Health Association and the National Rural
Health Care Association <ISOAbbreviation>J Rural
Health</ISOAbbreviation> </Journal> <ArticleTitle>Mental health at the
COVID-19 frontline: An assessment of distress, fear, and coping among
staff and attendees at screening clinics of rural/regional settings of
Victoria, Australia.</ArticleTitle> <Pagination>
<StartPage>773</StartPage> <EndPage>787</EndPage>
<MedlinePgn>773-787</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1111/jrh.12638</ELocationID>
<Abstract> <AbstractText Label="PURPOSE">Research examining
psychological well-being associated with COVID-19 in rural/regional
Australia is limited. This study aimed to assess the extent of
psychological distress, fear of COVID-19, and coping strategies among
the attendees in COVID-19 screening clinics at 2 rural Victorian
settings.</AbstractText> <AbstractText Label="METHODS">A cross-sectional
study was conducted during July 2020 to February 2021 inclusive.
Participants were invited to fill in an online questionnaire. Kessler
Psychological Distress Scale (K-10), Fear of COVID-19 Scale, and Brief
Resilient Coping Scale were used to assess psychological distress, fear
of COVID-19, and coping, respectively.</AbstractText>
<AbstractText Label="FINDINGS">Among 702 total participants, 69% were
females and mean age (±SD) was 49 (±15.8) years. One in 5 participants
(156, 22%) experienced high to very high psychological distress, 1 in 10
(72, 10%) experienced high fear, and more than half (397, 57%) had
medium to high resilient coping. Participants with mental health issues
had higher distress (AOR 10.4, 95% CI: 6.25-17.2) and fear (2.56,
1.41-4.66). Higher distress was also associated with having
comorbidities, increased smoking (5.71, 1.04-31.4), and alcohol drinking
(2.03, 1.21-3.40). Higher fear was associated with negative financial
impact, drinking alcohol (2.15, 1.06-4.37), and increased alcohol
drinking. Medium to high resilient coping was associated with being ≥60
years old (1.84, 1.04-3.24) and completing Bachelor and above levels of
education.</AbstractText> <AbstractText Label="CONCLUSION">People who
had pre-existing mental health issues, comorbidities, smoked, and
consumed alcohol were identified as high-risk groups for poorer
psychological well-being in rural/regional Victoria. Specific
interventions to support the mental well-being of these vulnerable
populations, along with engaging health care providers, should be
considered.</AbstractText> <CopyrightInformation>© 2021 The Authors. The
Journal of Rural Health published by Wiley Periodicals LLC on behalf of
National Rural Health Association.</CopyrightInformation> </Abstract>
<AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Rahman</LastName> <ForeName>Muhammad Aziz</ForeName>
<Initials>MA</Initials> <AffiliationInfo> <Affiliation>School of Health,
Federation University Australia, Berwick, Victoria,
Australia.</Affiliation> </AffiliationInfo> <AffiliationInfo>
<Affiliation>National Centre for Farmer Health, Deakin University,
Hamilton, Victoria, Australia.</Affiliation> </AffiliationInfo>
<AffiliationInfo> <Affiliation>Faculty of Public Health, Universitas
Airlangga, Surabaya, Indonesia.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Ford</LastName>
<ForeName>Dale</ForeName> <Initials>D</Initials> <AffiliationInfo>
<Affiliation>Western District Health Service (WDHS), Hamilton, Victoria,
Australia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Sousa</LastName>
<ForeName>Grace</ForeName> <Initials>G</Initials> <AffiliationInfo>
<Affiliation>South West Healthcare (SWH), Warrnambool, Victoria,
Australia.</Affiliation> </AffiliationInfo> <AffiliationInfo>
<Affiliation>Hawaii Emergency Physicians Associated, Honolulu, Hawaii,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Hedley</LastName> <ForeName>Lorraine</ForeName>
<Initials>L</Initials> <AffiliationInfo> <Affiliation>Western District
Health Service (WDHS), Hamilton, Victoria, Australia.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Greenstock</LastName> <ForeName>Louise</ForeName>
<Initials>L</Initials> <AffiliationInfo> <Affiliation>Western Alliance,
Warrnambool, Victoria, Australia.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Cross</LastName>
<ForeName>Wendy M</ForeName> <Initials>WM</Initials> <AffiliationInfo>
<Affiliation>School of Health, Federation University Australia, Berwick,
Victoria, Australia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Brumby</LastName>
<ForeName>Susan</ForeName> <Initials>S</Initials> <AffiliationInfo>
<Affiliation>National Centre for Farmer Health, Deakin University,
Hamilton, Victoria, Australia.</Affiliation> </AffiliationInfo>
<AffiliationInfo> <Affiliation>Western District Health Service (WDHS),
Hamilton, Victoria, Australia.</Affiliation> </AffiliationInfo>
</Author> </AuthorList> <Language>eng</Language> <PublicationTypeList>
<PublicationType UI="D016428">Journal Article</PublicationType>
<PublicationType UI="D013485">Research Support, Non-U.S.
Gov’t</PublicationType> </PublicationTypeList>
<ArticleDate DateType="Electronic"> <Year>2021</Year> <Month>12</Month>
<Day>13</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>England</Country>
      <MedlineTA>J Rural Health</MedlineTA>
      <NlmUniqueID>8508122</NlmUniqueID>
      <ISSNLinking>0890-765X</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000223" MajorTopicYN="N">Adaptation, Psychological</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000328" MajorTopicYN="N">Adult</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="Y">COVID-19</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D003430" MajorTopicYN="N">Cross-Sectional Studies</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005239" MajorTopicYN="N">Fear</DescriptorName>
        <QualifierName UI="Q000523" MajorTopicYN="N">psychology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008297" MajorTopicYN="N">Male</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008603" MajorTopicYN="N">Mental Health</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008875" MajorTopicYN="N">Middle Aged</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D014739" MajorTopicYN="N" Type="Geographic">Victoria</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">COVID-19</Keyword>
      <Keyword MajorTopicYN="N">coping</Keyword>
      <Keyword MajorTopicYN="N">mental health</Keyword>
      <Keyword MajorTopicYN="N">resilience</Keyword>
      <Keyword MajorTopicYN="N">rural</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="pubmed"> <Year>2021</Year> <Month>12</Month>
<Day>14</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="medline"> <Year>2022</Year> <Month>9</Month>
<Day>24</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="entrez"> <Year>2021</Year> <Month>12</Month>
<Day>13</Day> <Hour>13</Hour> <Minute>45</Minute> </PubMedPubDate>
</History> <PublicationStatus>ppublish</PublicationStatus>
<ArticleIdList> <ArticleId IdType="pubmed">34897806</ArticleId>
<ArticleId IdType="doi">10.1111/jrh.12638</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|Research examining psychological well-being
associated with COVID-19 in rural/regional Australia is limited. This
study aimed to assess the extent of psychological distress, fear of
COVID-19, and coping strategies among the attendees in COVID-19
screening clinics at 2 rural Victorian settings. A cross-sectional study
was conducted during July 2020 to February 2021 inclusive. Participants
were invited to fill in an online questionnaire. Kessler Psychological
Distress Scale (K-10), Fear of COVID-19 Scale, and Brief Resilient
Coping Scale were used to assess psychological distress, fear of
COVID-19, and coping, respectively. Among 702 total participants, 69%
were females and mean age (±SD) was 49 (±15.8) years. One in 5
participants (156, 22%) experienced high to very high psychological
distress, 1 in 10 (72, 10%) experienced high fear, and more than half
(397, 57%) had medium to high resilient coping. Participants with mental
health issues had higher distress (AOR 10.4, 95% CI: 6.25-17.2) and fear
(2.56, 1.41-4.66). Higher distress was also associated with having
comorbidities, increased smoking (5.71, 1.04-31.4), and alcohol drinking
(2.03, 1.21-3.40). Higher fear was associated with negative financial
impact, drinking alcohol (2.15, 1.06-4.37), and increased alcohol
drinking. Medium to high resilient coping was associated with being ≥60
years old (1.84, 1.04-3.24) and completing Bachelor and above levels of
education. People who had pre-existing mental health issues,
comorbidities, smoked, and consumed alcohol were identified as high-risk
groups for poorer psychological well-being in rural/regional Victoria.
Specific interventions to support the mental well-being of these
vulnerable populations, along with engaging health care providers,
should be considered. © 2021 The Authors. The Journal of Rural Health
published by Wiley Periodicals LLC on behalf of National Rural Health
Association. \| \|Acta tropica \|Monte Verde, a peri‑urban squatter
community near San Pedro Sula, virtually eliminated Aedes aegypti
production in all known larval habitats: wells; water storage containers
including pilas (open concrete water tanks used for laundry), 200-liter
drums, 1000-liter plastic “cisterns,” buckets; and objects collecting
rainwater. The project began in 2016 when Monte Verde was overrun with
dengue, Zika, and chikungunya. During more than a year of
experimentation, Monte Verde residents crafted an effective,
sustainable, and environmentally friendly toolkit that was inexpensive
but required full community participation. Biological control with
copepods, turtles, and tilapia was at the core of the toolkit, along
with a mix of other methods such as getting rid of unnecessary
containers, scrubbing them to remove Ae. aegypti eggs, and covering them
to exclude mosquitoes or rainwater. Environmentally friendly larvicides
also had a limited but crucial role. Key design features: (1) toolkit
components known to be nearly 100% effective at preventing Ae. aegypti
production when fitted to appropriate larval habitats; (2) using Ae.
aegypti larval habitats as a resource by transforming them into “egg
sinks” to drive Ae. aegypti population decline; (3) dedicated community
volunteers who worked with their neighbors, targeting 100% coverage of
all known Ae. aegypti larval habitats with an appropriate control
method; (4) monthly monitoring in which the volunteers visited every
house to assess progress and improve coverage as an ongoing learning
experience for both volunteers and residents. Taking pupae as an
indicator of Ae. aegypti production, from September 2018 to the end of
the record in December 2021 (except for a brief lapse during COVID
lockdown in 2020), the monthly count of pupae fluctuated between zero
and 0.6% of the 22,984 pupae counted in the baseline survey at the
beginning of the project. Adult Ae. aegypti declined to low numbers but
did not disappear completely. There were no recognizable cases of
dengue, Zika, or chikungunya after June 2018, though the study design
based on a single site did not provide a basis for rigorous confirmation
that Monte Verde’s Ae. aegypti control program was responsible.
Nonetheless, Monte Verde’s success at eliminating Ae. aegypti production
can serve as a model for extending this approach to other communities.
Key ingredients for success were outside stimulation and facilitation to
foster shared community awareness and commitment regarding the problem
and its solution, enduring commitment of local leadership, compatibility
of the toolkit with the local community, overcoming social obstacles,
rapid results with “success breeding success,” and building resilience.
Copyright © 2021. Published by Elsevier B.V. \| \|Clinical toxicology
(Philadelphia, Pa.) \|This is the 38th Annual Report of the American
Association of Poison Control Centers’ (AAPCC) National Poison Data
System (NPDS). As of 1 January, 2020, all 55 of the nation’s poison
centers (PCs) uploaded case data automatically to NPDS. The upload
interval was 6.15 \[4.60, 8.62\] (median \[25%, 75%\]) minutes,
effectuating a near real-time national exposure and information database
and surveillance system. We analyzed the case data tabulating specific
indices from NPDS. The methodology was similar to that of previous
years. Where changes were introduced, the differences are identified.
Cases with medical outcomes of death were evaluated by a team of medical
and clinical toxicologist reviewers using an ordinal scale of 1-6 to
assess the Relative Contribution to Fatality (RCF) of the exposure. In
2020, 3,316,738 closed encounters were logged by NPDS: 2,128,198 human
exposures, 66,745 animal exposures, 1,116,568 information requests, and
5,160 human confirmed nonexposures. Total encounters showed a 28.9%
increase from 2019, while health care facility (HCF) human exposure
cases decreased by 10.6%. While all information requests increased by
218.0%, medication identification (Drug ID) requests decreased by 31.5%,
and human exposure cases decreased by 0.928%. Medical Information
requests showed a 32.6-fold increase, reflecting COVID-19 pandemic calls
to PCs. Human exposures with less serious outcomes have decreased 1.90%
per year since 2008, while those with more serious outcomes (moderate,
major or death) have increased 4.59% per year since 2000.Consistent with
the previous year, the top 5 substance classes most frequently involved
in all human exposures were analgesics (10.3%), household cleaning
substances (8.37%), cosmetics/personal care products (6.53%),
antidepressants (5.30%), and sedatives/hypnotics/antipsychotics (4.92%).
As a class, antidepressant exposures increased most rapidly, by 1,793
cases/year (5.84%/year) over the past 10 years for cases with more
serious outcomes.The top 5 most common exposures in children age 5 years
or less were cosmetics/personal care products (11.8%), household
cleaning substances (11.3%), analgesics (7.57%), foreign
bodies/toys/miscellaneous (6.71%), and dietary
supplements/herbals/homeopathic (6.44%). Drug identification requests
comprised 2.89% of all information contacts. NPDS documented 4,488 human
exposures resulting in death; 3,869 (86.2%) of these were judged as
related (RCF of 1-Undoubtedly responsible, 2-Probably responsible, or
3-Contributory). These data support the continued value of PC expertise
and need for specialized medical toxicology information to manage more
serious exposures. Unintentional and intentional exposures continue to
be a significant cause of morbidity and mortality in the US. The near
real-time status of NPDS represents a national public health resource to
collect and monitor US exposure cases and information contacts. The
continuing mission of NPDS is to provide a nationwide infrastructure for
surveillance for all types of exposures (e.g., foreign body, infectious,
venomous, chemical agent, or commercial product), and the identification
and tracking of significant public health events. NPDS is a model system
for the near real-time surveillance of national and global public
health. \| \|Open forum infectious diseases \|Early recognition of
high-risk patients with coronavirus disease 2019 (COVID-19) may improve
outcomes. Although many predictive scoring systems exist, their
complexity may limit utility in COVID-19. We assessed the prognostic
performance of the National Early Warning Score (NEWS) and an age-based
modification (NEWS+age) among hospitalized COVID-19 patients enrolled in
a prospective, multicenter US Military Health System (MHS) observational
cohort study. Hospitalized adults with confirmed COVID-19 not requiring
invasive mechanical ventilation at admission and with a baseline NEWS
were included. We analyzed each scoring system’s ability to predict key
clinical outcomes, including progression to invasive ventilation or
death, stratified by baseline severity (low \[0-3\], medium \[4-6\], and
high \[≥7\]). Among 184 included participants, those with low baseline
NEWS had significantly shorter hospitalizations (P \< .01) and lower
maximum illness severity (P \< .001). Most (80.2%) of low NEWS vs 15.8%
of high NEWS participants required no or at most low-flow oxygen
supplementation. Low NEWS (≤3) had a negative predictive value of 97.2%
for progression to invasive ventilation or death; a high NEWS (≥7) had
high specificity (93.1%) but low positive predictive value (42.1%) for
such progression. NEWS+age performed similarly to NEWS at predicting
invasive ventilation or death (NEWS+age: area under the receiver
operating characteristics curve \[AUROC\], 0.69; 95% CI, 0.65-0.73;
NEWS: AUROC, 0.70; 95% CI, 0.66-0.75). NEWS and NEWS+age showed similar
test characteristics in an MHS COVID-19 cohort. Notably, low baseline
scores had an excellent negative predictive value. Given their easy
applicability, these scoring systems may be useful in resource-limited
settings to identify COVID-19 patients who are unlikely to progress to
critical illness. Published by Oxford University Press on behalf of
Infectious Diseases Society of America 2021. \| \|Academic psychiatry :
the journal of the American Association of Directors of Psychiatric
Residency Training and the Association for Academic Psychiatry \|This
report summarizes findings from a 2020 survey of US child and adolescent
psychiatry training programs that explored the impact of the COVID-19
pandemic on pediatric telepsychiatry training. The authors hypothesized
that telepsychiatry training significantly increased during the
pandemic, in part due to legal and regulatory waivers during the
COVID-19 public health emergency. In August 2020, an anonymous,
28-question online survey was emailed to all (138) accredited child
psychiatry fellowships on the Accreditation Council for Graduate Medical
Education website. Forty-nine programs responded (36%). This analysis
focuses on three of the 28 questions relevant to the hypotheses:
characteristics of the program’s training in telepsychiatry; perceived
impediments to clinical training; and perceived impediments to didactic
training pre-COVID onset vs. post-COVID onset, respectively. Total
scores were created to investigate differences in training programs and
impediments to including telepsychiatry pre- and post-COVID onset.
Paired sample t-tests were used to compare means pre- and post-COVID
onset. Results provided support for significant differences between
training components related to telepsychiatry pre- and post-COVID onset,
with participants reporting more training components post-COVID onset (M
= 5.69) than pre-COVID onset (M = 1.80); t(48) = 9.33, p \< .001.
Participants also reported significantly fewer barriers to providing
clinical experiences in pediatric telepsychiatry post-COVID onset (M =
2.65) than pre-COVID onset (M = 4.90); t(48) = - 4.20, p \< .001. During
the COVID-19 pandemic, pediatric telepsychiatry training in child
psychiatry fellowships increased significantly. Perceived barriers to
providing clinical, but not didactic, training decreased significantly.
© 2021. Academic Psychiatry. \| \|Journal of patient experience
\|COVID-19 has disproportionally burdened racial and ethnic minorities.
Minority populations report greater COVID-19 vaccine hesitancy; however,
no studies document COVID-19 vaccine willingness among Marshallese or
any Pacific Islander group, who are often underrepresented in research.
This study documents United States (US) Marshallese Pacific Islanders’:
willingness to get the COVID-19 vaccine, willingness to participate in
vaccine trials, and sociodemographic factors associated with
willingness. From July 27, 2020-November 22, 2020, a convenience sample
of US Marshallese adults were recruited through e-mail, phone calls, and
a Marshallese community Facebook page to participate in an online
survey. Of those surveyed (n = 120), 32.5% were extremely likely to get
the COVID-19 vaccine; 20.8% were somewhat likely; 14.2% were unlikely or
very unlikely; and 26.7% stated they did not know or were not sure. Only
16.7% stated they were willing to participate in a COVID-19 vaccine
trial. Vaccine willingness was positively associated with older age,
higher income, and longer US residence. Health insurance status and
having a primary care provider were positively associated with vaccine
willingness. Findings demonstrate within-group variation in COVID-19
vaccine willingness. © The Author(s) 2021. \| \|Clinical simulation in
nursing \|This paper describes the rapid conversion of a face-to-face
interprofessional (IP) disaster simulation to an online format in
response to COVID-19 campus closures. The online disaster simulation
utilized internet-based tools allowing real-time collaboration between
IP students. Team exercises involved disaster triage, disease outbreak
investigation, and disaster response. Surveys measuring self-assessment
of various IP skills and simulation learning outcomes (SLOs) were
compared with responses from previous face-to-face simulations. Results
indicated mean scores for IP skills were higher for online students when
compared with in-person simulations, and all SLOs were met. The online
disaster simulation provided an effective, innovative IP educational
opportunity. © 2021 International Nursing Association for Clinical
Simulation and Learning. Published by Elsevier Inc. All rights reserved.
\| \|Lancet regional health. Americas \|Transplant centers saw a
substantial reduction in deceased donor solid organ transplantation
since the beginning of the coronavirus 2019 (COVID-19) pandemic in the
United States. There is limited data on the impact of COVID-19 on adult
and pediatric heart transplant volume and variation in transplant
practices. We hypothesized that heart transplant activity decreased
during COVID-19 with associated increased waitlist mortality. The United
Network for Organ Sharing (UNOS) database was used to identify patients
at the time of listing for heart transplant from 2017-2020. Patients
were categorized as pediatric (\<18 years) or adult (≥18 years) and as
pre-COVID (2017-2019) or post-COVID (2020). Regional and statewide data
were taken from United States Census Bureau. CovidActNow project was
used to obtain COVID-19 mortality rates. Among pediatric patients,
average time on the waiting list decreased by 28 days. Even though the
average number of pediatric transplants (n=39 per month) did not change
significantly during 2020, there was a temporal decline in the first
quarter of 2020 followed by a sharp increase. Overall absolute pediatric
waitlist mortality decreased from 5•31 to 4•73, however female mortality
increased by 2%. Regional differences in pediatric mortality were
observed: Northeast, decreased by 7•5%; Midwest, decreased by 9%; West,
increased by 3•5%; and South, increased by 13%. North Dakota (0•55),
Oklahoma (0•21) and Hawaii (0•33) showed higher mortality than other
states per 100,000. In adults, average time on waiting list increased by
40 days and there was an increase in the number of transplants from 242
to 266. Adult waitlist mortality had a larger decrease, 18•44 to 15•70,
with an increase in female mortality of 7%. Regional differences in
adult mortality were also observed: Northeast, decreased by 3%; Midwest,
increased by 5•5%; West, increased by 4•5% and South, decreased by 5%.
Iowa (0•37), Wyoming (0•22), Arkansas (0•18) and Vermont (0•19) had the
highest mortality per 100,000 compared to the other states. Pediatric
heart transplant volume declined in early 2020 followed by a later
increase, while adult transplant volume increased all year round.
Although, overall pediatric waitlist mortality decreased, female
waitlist mortality increased for both adults and pediatrics. Regional
differences in waitlist mortality were observed for both pediatrics and
adults. Future studies are needed to understand this initial correlation
and to determine the impact of COVID-19 on heart transplant recipients.
This research received no specific grant from any funding agency in the
public, commercial or not-for-profit sectors. © 2021 The Author(s). \|
\|Vaccine: X \|The speed at which several COVID-19 vaccines went from
conception to receiving FDA and EMA approval for emergency use is an
achievement unrivaled in the history of vaccine development. Mass
vaccination efforts using the highly effective vaccines are currently
underway to generate sufficient herd immunity and reduce transmission of
the SARS-CoV-2 virus. Despite the most advanced vaccine technology,
global recipient coverage, especially in resource-poor areas remains a
challenge as genetic drift in naïve population pockets threatens overall
vaccine efficacy. In this study, we described the production of
insect-cell expressed SARS-CoV-2 spike protein ectodomain constructs and
examined their immunogenicity in mice. We demonstrated that, when
formulated with CoVaccine HTTM adjuvant, an oil-in-water nanoemulsion
compatible with lyophilization, our vaccine candidates elicit a
broad-spectrum IgG response, high neutralizing antibody (NtAb) titers
against SARS-CoV-2 prototype and variants of concern, specifically
B.1.351 (Beta) and P.1. (Gamma), and an antigen-specific IFN-γ secreting
response in outbred mice. Of note, different ectodomain constructs
yielded variations in NtAb titers against the prototype strain and some
VOC. Dose response experiments indicated that NtAb titers increased with
antigen dose, but not adjuvant dose, and may be higher with a lower
adjuvant dose. Our findings lay the immunological foundation for the
development of a dry-thermostabilized vaccine that is deployable without
refrigeration. © 2021 Published by Elsevier Ltd. \| \|Hawai’i journal of
health & social welfare \|In June 2021, over 200 stakeholders,
advocates, and visionaries gathered to launch the Healthy Hawai’i
Strategic Plan 2030 (HHSP), a 10-year strategic plan for improving the
health of Hawai’i residents by preventing and reducing chronic disease
and advancing health equity. The HHSP is a guide to enable coordination
across common risk factors, program areas, interventions, and strategies
for chronic disease prevention and control. Developed during the
COVID-19 pandemic, which revealed major areas of susceptibility in our
health system infrastructure and magnified existing disparities, the
HHSP prioritizes health equity and strives to create sustainable change
to transform communities, schools, health care and worksites to support
the health of the people of Hawai’i. The HHSP is a living document and
partners - present and future - are invited to work together to achieve
a healthier future for the people of Hawai’i. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Reducing Coronavirus disease 2019
(COVID-19) transmission relies on people quarantining after exposure to
COVID-19 or if they experience COVID-19 symptoms, and isolating from
others if COVID-19 positive. Quarantine and isolation last 10 to 14 days
and can be state-mandated; however, the level of compliance is unknown.
The University of Hawai’i Department of Family Medicine clinic called
patients instructed by our physicians to quarantine for exposure risk or
symptoms of potential COVID-19 infection between March 15, 2020, and
April 15, 2020. None of the patients tested positive for COVID-19.
Sixty-nine of 90 (77%) patients completed follow-up calls and
self-reported whether they had stayed home. Of these 69 patients, 32
(46%) broke quarantine to buy groceries (36%), work (9%), visit others
(6%), or for other reasons (12%). For patients living alone, 8 of 11
(73%) left home to buy groceries. For employed patients, 6 of 39 (15%)
returned to work during their quarantine period. Nearly half of our
patients did not quarantine for the entire period. Many persons left
home to buy food or to work. Strong public health messaging is needed to
educate communities about the requirement to quarantine. Clinicians can
help by asking patients about social and financial ability to
quarantine, schedule follow-up appointments to remind patients to stay
home, and link patients to food programs, financial assistance, and
other community resources to successfully quarantine and prevent
COVID-19 transmission. ©Copyright 2021 by University Health Partners of
Hawai‘i (UHP Hawai‘i). \| \|Epilepsy & behavior : E&B \|Telemedicine
clinic visits traditionally originated from spoke clinic sites, but
recent trends have favored home-based telemedicine, particularly in the
time of Covid-19. Our study focused on identification of barriers and
factors influencing perceptions of care with use of home-based
telemedicine in patients with seizures living in rural Hawaii. We
additionally compared characteristics of patients using telemedicine
versus in-person clinic visits prior to the Covid-19 pandemic. For the
retrospective portion of our study, we queried charts of adult
outpatients treated by the two full-time epileptologists at a Level 4
epilepsy center accredited by the National Association of Epilepsy
Centers between November 2018 and December 2019. We included patients
who live on the neighbor islands of Hawaii but not on Oahu, i.e.,
patients who would require air travel to see an epileptologist. There
had been no set protocol at the epilepsy center for telemedicine
referral; our practice had been to offer telemedicine visits to all
neighbor island patients when felt to be appropriate. We collected
demographic and clinic visit data. For the prospective portion we
surveyed neighbor island patients or their caregivers, seen via
home-based telemedicine between March 2020 and December 2020. We
obtained verbal consent for study participation. Survey questions
addressed satisfaction with clinical care, visit preferences, and
potential barriers to care. In a 14-month period prior to the Covid-19
pandemic, 75 (61%) neighbor island patients were seen exclusively
in-person in seizure clinic while 47 (39%) had at least one telemedicine
visit. 39% of patients seen only in-person were female whereas 38% of
patients seen by telemedicine were female. Patients seen in-person had
an older median age (47.2 years) compared to those seen at least once by
telemedicine (42.4 years). The no-show rate was 13% for in-person visits
versus 4% for telemedicine visits. Among patients seen in person, 17%
were Asian, 32% Native Hawaiian, and 47% White, whereas patients seen by
telemedicine were 15% Asian, 23% Native Hawaiian, and 57% White.
Patients who were seen in person lived in zip codes with median
household income of \$68,516 and patients who were seen by telemedicine
lived in zip codes with median household income of \$67,089. Patients
who were seen in person lived in zip codes in which 78% of the
population had access to broadband internet, whereas patients who were
seen by telemedicine lived in zip codes in which 79% of the population
had access to broadband internet. During the Covid-19 pandemic, we
surveyed 47 consecutive patients seen by telemedicine, 45% female with
median age of 33 years. Telemedicine connection was set up by the
patient in 74% of cases, or by the patient’s mother (15%), other family
member (9%), or other caregiver (2 %). Median patient satisfaction score
was 5 (“highly satisfied”) on a 5-point Likert scale with mean score of
4.6. Telemedicine visit was done using a smartphone by 62% of patients,
a computer by 36% of patients, and a tablet by 2% of patients. A home
WiFi connection was used in 83% of patients. Home-based telemedicine
visits provide a high-satisfaction method for seizure care delivery
despite some obstacles. Demographic disparities may be an obstacle to
telemedicine care and seem to relate to race and possibly age, rather
than to sex/gender, household income, or access to broadband internet.
Additionally, despite high satisfaction overall, more patients felt the
physical exam was superior at in-person clinic visits and more patients
expressed a preference for in-person visits. During the Covid-19
pandemic when there may be barriers to in-person clinic visits,
home-based telemedicine is a feasible alternative. Copyright © 2021
Elsevier Inc. All rights reserved. \| \|Child & adolescent social work
journal : C & A \|Child welfare work is inherently difficult, and child
welfare agencies are known to experience high rates of turnover. We
sought to expand the existing literature on intention to leave one’s
child welfare agency and commitment to child welfare work through
examining the coping mechanisms of frontline workers. Having and
utilizing healthy coping mechanisms has proved beneficial to child
welfare workers in previous research. In this paper, we examine specific
coping mechanisms identified in the Comprehensive Organizational Health
Assessment and how they were associated with child welfare workers’
intent to leave their agency and their commitment to remain in the field
of child welfare during the SARS CoV-2 (COVID-19) pandemic. We surveyed
over 250 child welfare caseworkers using the COHA instrument. Using both
bivariate analysis and linear regression, we identify specific coping
mechanisms, such as staying present with friends and family, as highly
influential and discuss ways to strengthen these areas. © The Author(s),
under exclusive licence to Springer Science+Business Media, LLC, part of
Springer Nature 2021. \| \|Epidemics \|As of 3rd June 2021, Malaysia is
experiencing a resurgence of COVID-19 cases. In response, the federal
government has implemented various non-pharmaceutical interventions
(NPIs) under a series of Movement Control Orders and, more recently, a
vaccination campaign to regain epidemic control. In this study, we
assessed the potential for the vaccination campaign to control the
epidemic in Malaysia and four high-burden regions of interest, under
various public health response scenarios. A modified
susceptible-exposed-infectious-recovered compartmental model was
developed that included two sequential incubation and infectious
periods, with stratification by clinical state. The model was further
stratified by age and incorporated population mobility to capture NPIs
and micro-distancing (behaviour changes not captured through population
mobility). Emerging variants of concern (VoC) were included as an
additional strain competing with the existing wild-type strain. Several
scenarios that included different vaccination strategies (i.e. vaccines
that reduce disease severity and/or prevent infection, vaccination
coverage) and mobility restrictions were implemented. The national model
and the regional models all fit well to notification data but
underestimated ICU occupancy and deaths in recent weeks, which may be
attributable to increased severity of VoC or saturation of case
detection. However, the true case detection proportion showed wide
credible intervals, highlighting incomplete understanding of the true
epidemic size. The scenario projections suggested that under current
vaccination rates complete relaxation of all NPIs would trigger a major
epidemic. The results emphasise the importance of micro-distancing,
maintaining mobility restrictions during vaccination roll-out and
accelerating the pace of vaccination for future control. Malaysia is
particularly susceptible to a major COVID-19 resurgence resulting from
its limited population immunity due to the country’s historical success
in maintaining control throughout much of 2020. Copyright © 2021 The
Authors. Published by Elsevier B.V. All rights reserved. \| \|Journal of
agromedicine \|The goal of this study was to conduct an exploratory
assessment of COVID-19 mitigation steps and compare workplace
experiences during the COVID-19 pandemic with Marshallese workers in
other occupations. Marshallese adults residing in the continental United
States (US) and Hawaii took part in an online survey. The sample was
divided into two categories: food processing workers and workers in all
other occupations. To examine differences between food processing
workers and workers from all other occupations, we used
Wilcoxon-Mann-Whitney U tests and Fisher’s Exact tests. Of those
employed at the time of the survey (n = 113), 31 were employed in food
processing plants, and 82 were employed in another occupation. Food
processing workers and workers in other occupations differed
significantly on level of education, length of residence in the US,
English-speaking ability, and health literacy. More food processing
workers reported that their employers installed barriers or provided
shields (45%), provided temperature screenings (71%), and tested for
COVID-19 (61%) compared with those in other occupations. A larger
proportion of food processing workers reported having no sick leave
compared with workers in other occupations, although they reported
COVID-19 testing and being insured at similar rates. This is the first
study to examine Marshallese food processing workers’ experiences during
the COVID-19 pandemic. Our findings show that while some food processing
employers implemented government-recommended guidelines to prevent the
spread of COVID-19, preventative and protective measures were not
comprehensively applied across the food processing industry, despite
efforts by public health agencies and community partners. \|
\|Computational biology and chemistry \|India, with around 15 million
COVID-19 cases, recently became the second worst-hit nation by the
SARS-CoV-2 pandemic. In this study, we analyzed the mutation and
selection landscape of 516 unique and complete genomes of SARS-CoV-2
isolates from India in a 12-month span (from Jan to Dec 2020) to
understand how the virus is evolving in this geographical region. We
identified 953 genome-wide loci displaying single nucleotide
polymorphism (SNP) and the Principal Component Analysis and mutation
plots of the datasets indicate an increase in genetic variance with
time. The 42% of the polymorphic sites display substitutions in the
third nucleotide position of codons indicating that non-synonymous
substitutions are more prevalent. These isolates displayed strong
evidence of purifying selection in ORF1ab, spike, nucleocapsid, and
membrane glycoprotein. We also find some evidence of localized positive
selections ORF1ab, spike glycoprotein, and nucleocapsid. The CDSs for
ORF3a, ORF8, nucleocapsid phosphoprotein, and spike glycoprotein were
found to evolve at rapid rate. This study will be helpful in
understanding the dynamics of rapidly evolving SARS-CoV-2. Copyright ©
2021 Elsevier Ltd. All rights reserved. \| \|Applied clinical
informatics \|We describe the design, implementation, and validation of
an online, publicly available tool to algorithmically triage patients
experiencing severe acute respiratory syndrome coronavirus
(SARS-CoV-2)-like symptoms. We conducted a chart review of patients who
completed the triage tool and subsequently contacted our institution’s
phone triage hotline to assess tool- and clinician-assigned triage
codes, patient demographics, SARS-CoV-2 (COVID-19) test data, and health
care utilization in the 30 days post-encounter. We calculated the
percentage of concordance between tool- and clinician-assigned triage
categories, down-triage (clinician assigning a less severe category than
the triage tool), and up-triage (clinician assigning a more severe
category than the triage tool) instances. From May 4, 2020 through
January 31, 2021, the triage tool was completed 30,321 times by 20,930
unique patients. Of those 30,321 triage tool completions, 51.7% were
assessed by the triage tool to be asymptomatic, 15.6% low severity,
21.7% moderate severity, and 11.0% high severity. The concordance rate,
where the triage tool and clinician assigned the same clinical severity,
was 29.2%. The down-triage rate was 70.1%. Only six patients were
up-triaged by the clinician. 72.1% received a COVID-19 test administered
by our health care system within 14 days of their encounter, with a
positivity rate of 14.7%. The design, pilot, and validation analysis in
this study show that this COVID-19 triage tool can safely triage
patients when compared with clinician triage personnel. This work may
signal opportunities for automated triage of patients for conditions
beyond COVID-19 to improve patient experience by enabling self-service,
on-demand, 24/7 triage access. Thieme. All rights reserved. \| \|Journal
of public health policy \|All-cause mortality counts allow public health
authorities to identify populations experiencing excess deaths from
pandemics, natural disasters, and other emergencies. Delays in the
completeness of mortality counts may contribute to misinformation
because death counts take weeks to become accurate. We estimate the
timeliness of all-cause mortality releases during the COVID-19 pandemic
for the dates 3 April to 5 September 2020 by estimating the number of
weekly data releases of the NCHS Fluview Mortality Surveillance System
until mortality comes within 99% of the counts in the 19 March 19 2021
provisional mortality data release. States’ mortality counts take 5
weeks at median (interquartile range 4-7 weeks) to completion. The
fastest states were Maine, New Hampshire, Vermont, New York, Utah,
Idaho, and Hawaii. States that had not adopted the electronic death
registration system (EDRS) were 4.8 weeks slower to achieve complete
mortality counts, and each weekly death per 10^8 was associated with a
0.8 week delay. Emergency planning should improve the timeliness of
mortality data by improving state vital statistics digital
infrastructure. © 2021. The Author(s), under exclusive licence to
Springer Nature Limited. \| \|Hawai’i journal of health & social welfare
\|Sugar-sweetened beverage (SSB) consumption is associated with
increased risk of obesity, diabetes, and other chronic diseases. SSB
consumption is also a health equity issue, as rates of consumption and
related chronic diseases vary by race, ethnicity, and income in Hawai’i.
The COVID-19 pandemic has highlighted the need for greater investment in
public health and the well-being of communities experiencing health
disparities because individuals with chronic diseases are more likely to
develop complications from the virus. It has also created economic
hardships for the people of Hawai’i, especially the state’s most
vulnerable populations. Amid this health and economic crisis, an
opportunity exists to implement an SSB fee in Hawai’i. An SSB fee would
impose a fee on SSB distributors that would be passed on to consumers in
the form of price increases that influence purchasing behavior.
Jurisdictions with SSB taxes or fees have seen reductions in SSB
purchases and consumption and have generated millions of dollars in
revenues to support health initiatives and reduce socioeconomic
disparities. Models predict that a \$0.02 SSB fee in Hawai’i could
generate \$60.5 million and significantly reduce healthcare costs and
chronic diseases. This commentary will present an SSB fee policy as a
viable model for Hawai’i to reduce SSB consumption, lower chronic
disease risks, and generate needed revenues to support health, reduce
inequities, and rebuild the state’s economy. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Recent studies have identified high
rates of chronic disease in Hawai’i’s adults and youth. As the state
responds to the COVID-19 pandemic and looks beyond it, the prevention
and management of chronic diseases are critical for community health and
wellbeing. Low health literacy is more common in rural populations,
Filipinos, and Pacific Islanders in Hawai’I, older adults, and many
other groups with high rates of chronic disease. Promoting health
literacy can reduce chronic disease burdens for individuals, families,
and communities. Using the framework of the social-ecological model,
which is important for visioning effective chronic disease management
and prevention, this article provides a blueprint of layers of influence
for building a health literate Hawai’I generally and around chronic
disease specifically. The article will close with a call to action
informed by the National Action Plan to Improve Health Literacy for
stakeholders and providers to address health literacy in the state of
Hawai’I in organizations, systems, and policy. These actions should
address root causes of disease and help build more equitable health
outcomes across the state now and in the future. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|The Native Hawaiian and Pacific
Islander community found itself on the front pages of national news when
the COVID-19 pandemic struck the United States. By April 2020, the
small, frequently overlooked community experienced the highest COVID-19
case rates in 5 states including Hawai’i. In response, Native Hawaiian
and Pacific Islander networks across the US were mobilized to address
the crisis. In Hawai’i, the Native Hawaiian Pacific Islander COVID-19
Response, Recovery, and Resilience Team was created. Framed by
Indigenous Pacific based cultural values, protocols, and practices, the
team consists of multiple committees that examine policy; testing,
contract tracing, and isolation; communications; social supports and
resources; and data and research. Inherent in this work are the shared
core values of pono (righteousness, goodness), aloha (love, compassion),
laulima (cooperation), and imua (moving forward with strength) as well
as an ‘ohana/aiga (family)-based, kuleana (responsibility)-centric
approach that acknowledges, honors, and values ’ike kūpuna (ancestral
knowledge). With the burden of not only COVID-19 disparities, but also
chronic diseases and socioeconomic disparities that place Native
Hawaiian and Pacific Islander communities at increased risk for adverse
impacts from COVID-19, an effective response is critical. This article,
authored by members of the Team’s Policy Committee, discusses the
development of a cultural framework that guides its advocacy efforts.
The Policy Committee’s work presents a cultural framework that grounds
and guides their efforts for effectively promoting a strong voice in
governmental and agency policies which would ultimately contribute to a
healthy and thriving Native Hawaiian and Pacific Islander community.
©Copyright 2021 by University Health Partners of Hawai‘i (UHP Hawai‘i).
\| \|Hawai’i journal of health & social welfare \|Community health
workers play an instrumental role in the health care system and are
critical partners in pandemic response. In Hawai’i, community health
workers are working to reduce the burden of chronic disease among
Pacific Islander, Filipino, and Native Hawaiian populations in
partnership with government agencies and health care organizations. This
commentary reviews the role community health workers in Hawai’i are
playing in assisting with the COVID-19 response. Utilizing their skills
and the community’s trust, they are optimally positioned to reach
marginalized and vulnerable populations hit hardest by COVID-19;
community health workers educate, screen, and provide social service
referrals to community members. ©Copyright 2021 by University Health
Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i journal of health &
social welfare \|Early evidence of disproportionate COVID-19 infection
and death rates in Native Hawaiian and Pacific Islander communities in
the continental US raised concerns for similar disparities in Hawai’i,
where these communities make up 25% of the state’s population.
Representatives from more than 40 different government, academic,
institutional and community-based organizations partnered to form the
Hawai’i Native Hawaiian and Pacific Islander COVID-19 Response,
Recovery, and Resilience Team. The team consists of 5 committees
including the Data & Research Committee. This committee is tasked with
examining issues regarding the acquisition, quality, public reporting,
and utilization of race/ethnicity-related health data used to inform
priorities and guide resource allocation. Problems addressed by this
committee include: inconsistency across agencies in the use of race
identifiers, defaulting to the Office of Management and Budget standards
which aggregated Native Hawaiian and Pacific Islanders, and methods of
data collection and reporting by the Department of Health. Outcomes
include: 2 forms with race categories that reflect the population of
Hawai’i; the reporting of disaggregated data by the Department of
Health; and conversations with testing sites, laboratories, and health
institutions urging a standardized form for race/ethnicity data
collection. The collection and reporting of disaggregated race/ethnicity
data is critical to guiding organizations in addressing underlying
inequities in chronic disease and social determinants of health that can
exacerbate the adverse effects of COVID-19. The Data and Research
Committee’s network offers a community-based model for collaborative
work that honors culture and ensures Native Hawaiian, Pacific Islander,
and other minority populations are recognized and counted. ©Copyright
2021 by University Health Partners of Hawai‘i (UHP Hawai‘i). \|
\|Hawai’i journal of health & social welfare \|Micronesian communities
in Hawai’i have a long history of mobilizing to address challenges they
encounter as the most recent and fastest growing Pacific Islander
immigrant population in the state. In particular, community leaders
navigate a slew of obstacles specific to systemic racism and health care
access. These hurdles have become exacerbated by the COVID-19 pandemic,
prompting a range of Micronesian-led responses to the health crisis
including strategic adaptations to existing networks and roles to
address essential public health functions. These community responses
have filled many critical gaps left by the state’s delayed response to
addressing the disparate impact of COVID-19 on Micronesian communities.
This article highlights and encourages engagement with diverse models of
collaboration and elevation of Micronesian leadership that has resulted
in more productive cooperation with government leaders, agencies, and
policymakers. This work offers insight into pathways forward toward
healthier Micronesian families and communities. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Increasing exclusive breastfeeding
rates is an established public health strategy to reduce chronic disease
and protect infants from illness. The role of breastfeeding in
addressing health disparities takes on new significance as the COVID-19
pandemic has disproportionately impacted some communities in Hawai’i,
and those with chronic conditions face increased risk of hospitalization
and death. However, there are myriad policy, systemic, and environmental
barriers that make it difficult for parents to breastfeed, some of which
have been exacerbated by the COVID-19 pandemic. This editorial discusses
the importance of breastfeeding in reducing chronic disease, reviews the
status of breastfeeding in Hawai’i, explores the challenges parents face
in breastfeeding their infants, especially in the time of COVID-19, and
presents opportunities for improved access to lactation care to reduce
health disparities. ©Copyright 2021 by University Health Partners of
Hawai‘i (UHP Hawai‘i). \| \|Hawai’i journal of health & social welfare
\|The precarious financial status of the majority of Hawai’i residents
coupled with the state’s heavy reliance on tourism suggests that
residents are particularly vulnerable to increased economic hardship
resulting from the COVID-19 pandemic, which temporarily shut down the
tourism industry and continues to erect barriers for resuming
operations. Understanding how Hawai’i residents prioritize access to
health care, food economics, care of ’āina, and culturally informed
community in light of the current and future economic situation can
inform policy actions that will support public health. To that end, this
paper analyzes: (1) Hawai’i residents’ views on health, specifically
food security and healthcare, and their priorities for the future of
these areas; (2) the differences between Native Hawaiian and non-Native
Hawaiian views and priorities; and (3) the differences in views and
priorities between families with higher and lower levels of economic
stability. The authors close with policy recommendations that can be
seen as medicine, or ways to heal Hawai’i, as the state shifts towards a
more equitable and sustainable future. ©Copyright 2021 by University
Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i journal of health
& social welfare \|Utilizing 11 waves of data from the Household Pulse
Survey collected between April and November 2020, this study examines
disparities in psychological distress (defined as having symptoms of
anxiety/depression) among adult residents of Hawai’i during the COVID-19
pandemic. Results showed that 36.4% of the respondents reported symptoms
of distress. Younger age, female, and lower household income were
associated with higher levels of psychological distress than older age,
male, and higher household income. The prevalence ratios of distress for
those aged 18-24, 25-34, 35-44 and females were 43.1%, 47.3%, 44.1%, and
39.3% respectively. Asians experienced lower prevalence compared to
other racial/ethnic groups. Two practical implications are offered.
First, the economic sequelae of COVID-19 impact psychological distress
even when the community infection rate is stable. Second, disparities in
psychosocial distress demonstrate that social and economic resources are
needed by social groups such as young adults, females, and racial/ethnic
minorities that have experienced the highest impact. Strategies need to
be developed to mitigate the unavoidable local consequences of a
pandemic. ©Copyright 2021 by University Health Partners of Hawai‘i (UHP
Hawai‘i). \| \|Hawai’i journal of health & social welfare \|NA \|
\|European journal of nutrition \|In less than one and a half year, the
COVID-19 pandemic has nearly brought to a collapse our health care and
economic systems. The scientific research community has concentrated all
possible efforts to understand the pathogenesis of this complex disease,
and several groups have recently emphasized recommendations for
nutritional support in COVID-19 patients. In this scoping review, we aim
at encouraging a deeper appreciation of magnesium in clinical nutrition,
in view of the vital role of magnesium and the numerous links between
the pathophysiology of SARS-CoV-2 infection and magnesium-dependent
functions. By searching PubMed and Google Scholar from 1990 to date, we
review existing evidence from experimental and clinical studies on the
role of magnesium in chronic non-communicable diseases and infectious
diseases, and we focus on recent reports of alterations of magnesium
homeostasis in COVID-19 patients and their association with disease
outcomes. Importantly, we conduct a census on ongoing clinical trials
specifically dedicated to disclosing the role of magnesium in COVID-19.
Despite many methodological limitations, existing data seem to
corroborate an association between deranged magnesium homeostasis and
COVID-19, and call for further and better studies to explore the
prophylactic or therapeutic potential of magnesium supplementation. We
propose to reconsider the relevance of magnesium, frequently overlooked
in clinical practice. Therefore, magnesemia should be monitored and, in
case of imbalanced magnesium homeostasis, an appropriate nutritional
regimen or supplementation might contribute to protect against
SARS-CoV-2 infection, reduce severity of COVID-19 symptoms and
facilitate the recovery after the acute phase. © 2021. The Author(s). \|
\|PLoS medicine \|UNAIDS has established new program targets for 2025 to
achieve the goal of eliminating AIDS as a public health threat by 2030.
This study reports on efforts to use mathematical models to estimate the
impact of achieving those targets. We simulated the impact of achieving
the targets at country level using the Goals model, a mathematical
simulation model of HIV epidemic dynamics that includes the impact of
prevention and treatment interventions. For 77 high-burden countries, we
fit the model to surveillance and survey data for 1970 to 2020 and then
projected the impact of achieving the targets for the period 2019 to
2030. Results from these 77 countries were extrapolated to produce
estimates for 96 others. Goals model results were checked by comparing
against projections done with the Optima HIV model and the AIDS Epidemic
Model (AEM) for selected countries. We included estimates of the impact
of societal enablers (access to justice and law reform, stigma and
discrimination elimination, and gender equality) and the impact of
Coronavirus Disease 2019 (COVID-19). Results show that achieving the
2025 targets would reduce new annual infections by 83% (71% to 86%
across regions) and AIDS-related deaths by 78% (67% to 81% across
regions) by 2025 compared to 2010. Lack of progress on societal enablers
could endanger these achievements and result in as many as 2.6 million
(44%) cumulative additional new HIV infections and 440,000 (54%) more
AIDS-related deaths between 2020 and 2030 compared to full achievement
of all targets. COVID-19-related disruptions could increase new HIV
infections and AIDS-related deaths by 10% in the next 2 years, but
targets could still be achieved by 2025. Study limitations include the
reliance on self-reports for most data on behaviors, the use of
intervention effect sizes from published studies that may overstate
intervention impacts outside of controlled study settings, and the use
of proxy countries to estimate the impact in countries with fewer than
4,000 annual HIV infections. The new targets for 2025 build on the
progress made since 2010 and represent ambitious short-term goals.
Achieving these targets would bring us close to the goals of reducing
new HIV infections and AIDS-related deaths by 90% between 2010 and 2030.
By 2025, global new infections and AIDS deaths would drop to 4.4 and 3.9
per 100,000 population, and the number of people living with HIV (PLHIV)
would be declining. There would be 32 million people on treatment, and
they would need continuing support for their lifetime. Incidence for the
total global population would be below 0.15% everywhere. The number of
PLHIV would start declining by 2023. \| \|Hawai’i journal of health &
social welfare \|The Republic of the Marshall Islands, American Samoa,
the Federated States of Micronesia, and the Republic of Palau have been
without any COVID-19 community transmission since the beginning of the
global pandemic. The Commonwealth of the Northern Mariana Islands has
experienced modest community transmission, and Guam has had significant
COVID-19 community transmission and morbidity. Although several of these
United States Affiliated Pacific Island jurisdictions made difficult
strategic choices to prevent the spread of COVID-19 which have been
largely successful, the built environment and the population density in
the urban areas of the Pacific remain inherently conducive to rapid
COVID-19 transmission. Rapid transmission could result in devastating
health and economic consequences in the absence of continued vigilance
and long-term strategic measures. The unique COVID-19 vulnerability of
islands in the Pacific can be modeled through examination of recent
outbreaks onboard several United States Naval ships and other marine
vessels. The environmental characteristics that pose challenges to
infection control on an isolated naval ship are analogous to the
environmental characteristics of these Pacific island communities.
Considering a collection of case studies of COVID-19 transmission on
ships and applying to Pacific Island environments, provides a heuristic,
easily accessible epidemiologic framework to identify methods for
interventions that are practical and reliable towards COVID-19
containment, prevention, and control. Using accessible evidence based
public health policies, infection risk can be decreased with the
objective of maintaining in-country health and social stability. These
case studies have also been examined for their relevance to current
discussions of health care infrastructure and policy in the Pacific
Islands, especially that of vaccination and repatriation of citizens
marooned in other countries. The need for aggressive preparation on the
parts of territories and nations not yet heavily exposed to the virus is
critical to avoid a rapid “burn-through” of disease across the islands,
which would likely result in catastrophic consequences. ©Copyright 2021
by University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Hawai’i’s Pacific Islander (PI)
population has suffered a higher burden of coronavirus disease 2019
(COVID-19) infections, hospitalizations, and deaths compared to other
groups in the state. The Hawai’i Emergency Management Agency Community
Care Outreach Unit conducted an assessment across the state to gain an
understanding of the impact of the COVID-19 pandemic on the health and
social welfare of households. Survey data was collected from individuals
across the state during a period of 3 weeks (August 12-September 5,
2020). The following are resulting recommendations from the Pacific
Island community to mitigate the impact and disparities of the pandemic
as immediate and medium-term structural requests: (1) ensure that
Pacific Island communities are proactively represented in state and
county committees that develop health interventions to ensure that
relevant language and culturally tailored communications and strategies
are included, (2) provide consistent funding and community centered
support to ensure consistent COVID-19 impact services for the Pacific
Island families, (3) enhance the capacity of PI health care navigators
and interpreters through increased funding and program support, and (4)
engage state policy makers immediately to understand and address the
systemic structural barriers to health care and social services for
Pacific Islanders in Hawai’i. These recommendations were developed to
address the generational inequities and disparities that exist for
Pacific islanders in Hawai’i which were exacerbated by the COVID-19
pandemic. ©Copyright 2021 by University Health Partners of Hawai‘i (UHP
Hawai‘i). \| \|Hawai’i journal of health & social welfare \|Hawai’i’s
Pacific Islander (PI) population has suffered a higher burden of
coronavirus disease 2019 (COVID-19) infections, hospitalizations, and
deaths compared to other groups in the state. The Hawai’i Emergency
Management Agency Community Care Outreach Unit conducted an assessment
across the state to gain an understanding of the impact of the COVID-19
pandemic on the health and social welfare of households. Survey data was
collected from individuals across the state during a period of 3 weeks
(August 12-September 5, 2020). The following are resulting
recommendations from the Pacific Island community to mitigate the impact
and disparities of the pandemic as immediate and medium-term structural
requests: (1) ensure that Pacific Island communities are proactively
represented in state and county committees that develop health
interventions to ensure that relevant language and culturally tailored
communications and strategies are included, (2) provide consistent
funding and community centered support to ensure consistent COVID-19
impact services for the Pacific Island families, (3) enhance the
capacity of PI health care navigators and interpreters through increased
funding and program support, and (4) engage state policy makers
immediately to understand and address the systemic structural barriers
to health care and social services for Pacific Islanders in Hawai’i.
These recommendations were developed to address the generational
inequities and disparities that exist for Pacific islanders in Hawai’i
which were exacerbated by the COVID-19 pandemic. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Hawai’i’s Filipino community has
been deeply impacted by coronavirus disease 2019 (COVID-19). This
article reports the findings for the Filipino population from the
Hawai’i Emergency Management Agency (HI-EMA) Community Care Outreach
Unit (CCO) Unit evaluation assessment of the impact of COVID-19 on the
health and social welfare of individuals across the state. The survey
was conducted from August-September 2020. We propose recommendations to
mitigate the impact of the pandemic on this community, including the
following actions: (1) developing linguistically and culturally
appropriate support for all COVID-19 related services, especially for
the high number of older Filipinos with limited English proficiency, (2)
providing support and resource information in locations that are
accessible to Filipino communities, and (3) supporting those already
doing work to address the deep and diverse needs in the Filipino
community with funding. Building partnerships between existing Filipino
organizations, health and social service providers, and state agencies
will contribute to sustainability over time. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|Native Hawaiians (NHs) are among
the most vulnerable groups at greater risk for coronavirus disease 2019
(COVID-19). To understand the impact of COVID-19 on the state’s
population, a 35-question cross-sectional survey was administered across
the state of Hawai’i. NH data from the larger report are provided here.
The findings indicate that the impact of COVID-19 is disproportionately
affecting NH households in areas of income and housing stability,
chronic disease prevalence, emotional wellness, and COVID-19 prevention.
Short-, medium-, and long-term recommendations are presented as next
steps to addressing the health inequities among NHs. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i
journal of health & social welfare \|The Community Care Outreach Unit of
the Hawai’i Emergency Management Agency (HI-EMA) Medical/Public Heath
Branch conducted a survey to gauge the impact, needs, and threats to the
health and social welfare of individuals and their families pertaining
to coronavirus disease 2019 (COVID-19). This article presents key
findings for the County of Maui (MC) in the state. A mixed-methods
framework was utilized for survey distribution and recruitment of
participants from across the state. Recruitment strategies included
snowball sampling via website and social media, and paper surveys.
Descriptive analysis of the data is presented to give a basic overview
of the impact of COVID-19 in MC. A total of 883 participants in MC
responded to the survey. Approximately one-third reported that they or
family members experienced reduced work hours or lost their job because
of COVID-19. In all questions related to paying for essential living
needs, the percentage of participants who expected to have future
problems was higher than the percentage who reported having current
problems. Of those preparing for the fall 2020 school semester, expected
challenges included lack of funds to purchase school supplies, lack of
face coverings, and language barriers. Most participants in MC perceived
the severity of COVID-19 to be moderate to very high, and there was a
moderate level of knowledge about which groups are more at risk for
contracting severe COVID-19. Less than half would know how to provide
care for someone in their family with COVID-19. Several resource
barriers for caring for a family member with COVID-19 were identified.
The COVID-19 pandemic has had a more severe impact on Native Hawaiian
and Pacific Islander groups compared to others in the county. The
results may provide a baseline for understanding the impact, needs, and
threats to the health and social welfare of individuals and their
families in MC. Local stakeholders can utilize this information to
develop priorities, strategies, and programs to address the COVID-19
pandemic response in MC. ©Copyright 2021 by University Health Partners
of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i journal of health & social
welfare \|The Hawai’i Emergency Management Agency Community Care
Outreach Unit (CCO) conducted a survey to gauge the impact of
coronavirus disease 2019 (COVID-19) on the health and social welfare of
individuals and their families across the state of Hawai’i. A
mixed-methods framework was utilized for survey distribution. This
article presents a descriptive analysis of the data to provide a basic
overview of the impact of COVID-19 in Kaua’i County (KC), as assessed in
August/September 2020. A total of 420 participants in KC responded to
the statewide survey. Approximately one-third reported that they or
their family members experienced reduced work hours or lost their job
because of COVID-19. Many reported difficulties paying for many types of
living essentials and expected these difficulties to increase in the
near future. Prevalent challenges for the fall school semester included
access to funds for school supplies and face-coverings. About one-third
reported feeling nervous more than half the time or nearly every day in
the past 2 weeks, and one-fourth reported feeling worried more than half
the time or nearly every day in the past 2 weeks. The majority perceived
the severity of COVID-19 to be moderate/very high and most had at least
a moderate level of knowledge about risks for contracting severe
COVID-19. Less than half said they would know how to provide care for
someone in their family with COVID-19. Half of the respondents in KC
reported maintaining social distancing usually/all of the time, the
majority reported wearing a face-covering usually/always when needed.
The results provide a baseline for understanding the impact, needs, and
threats to the health and social welfare of households and their
families in KC as a result of COVID-19. Local stakeholders can utilize
this information for developing priorities, strategies, and programs to
address the pandemic where needed and also to assess progress in areas
of need. ©Copyright 2021 by University Health Partners of Hawai‘i (UHP
Hawai‘i). \| \|Hawai’i journal of health & social welfare \|The
Community Care Outreach Unit (CCO) of the Hawai’i Emergency Management
Medical/Public Health Services Branch conducted a survey to gauge the
impact of coronavirus disease 2019 (COVID-19) on the health and social
welfare of individuals and families in the state of Hawai’i. A
mixed-methods framework was utilized for survey distribution; 7927
respondents participated in the survey. This article presents key
findings for the state’s Hawai’i County (HC). It presents a descriptive
analysis of the data to provide a basic overview of the impact of
COVID-19 in HC, as assessed in August-September 2020. A total of 936
participants from HC responded to the survey. Approximately one-third
reported that they or their family members experienced reduced work
hours, and one-fifth lost their jobs because of COVID-19. Many reported
difficulties paying for many types of living essentials and expected
these difficulties to increase in the near future. Challenges for the
fall school semester included lack of access to funds for school
supplies and face-coverings. The majority perceived the severity of
COVID-19 to be moderate/very high and most had at least a moderate level
of knowledge about risks for developing severe COVID-19. Approximately
half reported maintaining social distancing usually/all of the time, and
about two-thirds reported wearing a face-covering usually/always when
needed. Other barriers for COVID disease prevention and response
included a lack of space for quarantine/isolation of family members, not
having enough cleaning supplies, low knowledge of how to care for a
household member with COVID disease and not having someone available to
care for them if they contracted the virus. The results provide a
baseline for understanding the impact, needs, and threats to the health
and social welfare of individuals and their families as a result of
COVID-19 in HC. Local stakeholders can utilize this information when
developing priorities, strategies, and programs to address the pandemic
where needed. ©Copyright 2021 by University Health Partners of Hawai‘i
(UHP Hawai‘i). \| \|Hawai’i journal of health & social welfare \|To
address the impact of COVID-19 in the state of Hawai’i, the Hawai’i
Emergency Management Agency Medical Public Health Branch activated its’
Community Care Outreach Unit (CCO Unit). A team from this unit developed
a survey to assess the impact, needs, and threats to the health and
social welfare of individuals and their families as they pertain to
COVID-19. This article presents key findings for the City and County of
Honolulu (CCH). A total of 5598 CCH residents responded. Approximately
half of these respondents reported they or their household members
experienced reduced work hours or lost their job as a result of
COVID-19. In all questions related to paying for essential living costs,
at the time of the survey, the percentage of participants who expected
to have future problems nearly doubled. Those preparing for school in
the fall school semester expected challenges centered on insufficient
funds to purchase school supplies, lack of available face-coverings, and
language barriers. Financial assistance, rental assistance, and food
assistance seemed to be more difficult to apply for compared to health
care services. The most common reasons for difficulty with applications
noted by residents included that they could not figure out how to
complete the form, did not have all the documents, or could not get
through on the telephone. About one-half of CCH participants reported
feeling nervous more than half of the days or nearly every day in the
past 2 weeks. Most perceived the severity of COVID-19 to be moderate to
very high. Less than half reported knowing how to provide care for
someone in their family with COVID-19. Half of the CCH participants
reported that they practice social distancing usually or all of the
time, and the majority reported wearing a face-covering usually or
always when outside of the home. A significant portion of respondents
reported barriers for providing care for a household member exposed or
infected with COVID-19. Such barriers included a lack of space in their
home for isolation; not having enough cleaning supplies; no working
thermometer in the home, or no family member available to care for them.
The results presented may provide a baseline for understanding the
impact, needs, and threats to the health and social welfare of
individuals and their families in CCH and across the state of Hawai’i.
Local stakeholders can utilize this information in developing
priorities, strategies, and programs to address the pandemic as it
continues to unfold and learn lessons for future pandemics. ©Copyright
2021 by University Health Partners of Hawai‘i (UHP Hawai‘i). \|
\|Hawai’i journal of health & social welfare \|The coronavirus disease
2019 (COVID-19) pandemic has had a profound impact on the world. To
address the impact of COVID-19 in the state of Hawai’i, the Hawai’i
Emergency Management Agency (HI-EMA) Community Care Outreach Unit
conducted an assessment survey to determine the impact of COVID-19 on
the health and social welfare of individuals and their families across
the state. This article presents key statewide findings from this
assessment, including areas of need and community-based recommendations
to help mitigate the impact of the pandemic, particularly for vulnerable
groups. A total of 7927 participants responded to the assessment survey
from across the state’s counties. In all questions related to paying for
essentials, the percentage of participants that expect to have problems
in the future, as compared to now, almost doubled. Slightly higher than
one-third reported that they would know how to care for a family member
in the home with COVID-19, and half of the respondents reported a lack
of space for isolation in their home. About half reported that if they
got COVID-19, they would have someone available to care for them.
Overall, Native Hawaiian, Pacific Islander, and Filipino groups reported
greater burden in almost all areas surveyed. The results presented
provide a baseline in understanding the impact, needs, and threats to
the health and social welfare of individuals and their families across
the state of Hawai’i. Local stakeholders can utilize this information
when developing priorities, strategies, and programs to address current
and future pandemics in the state. ©Copyright 2021 by University Health
Partners of Hawai‘i (UHP Hawai‘i). \| \|Hawai’i journal of health &
social welfare \|Health and social service organizations across Hawai’i
were surveyed between April 29 and May 11, 2020 by the Community Care
Outreach Unit of the Hawai’i Emergency Management Agency. This article
contextualizes and describes some of the major findings of that survey
that reveal the impact of coronavirus disease 2019 (COVID-19) on Hawai’i
community agencies, service organizations, and the individuals they
serve. Major issues for individuals served by the responding
organizations included securing basic needs such as food and housing as
well as access to health services, mental health needs, and COVID-19
concerns (such as inadequate personal protective equipment, cleaning
supplies, quarantine, and testing issues). Respondents reported that job
loss and the resulting financial problems were a root cause of personal
strain among clients served. Community-level stress was related to the
distressed economy and store closures. Fulfilling immediate and future
needs of health and social service agencies and the individuals they
serve, as articulated in this report, could dampen the effect of
COVID-19, promote population wellbeing, and support community
resilience. ©Copyright 2021 by University Health Partners of Hawai‘i
(UHP Hawai‘i). \| \|Hawai’i journal of health & social welfare \|NA \|
\|Hawai’i journal of health & social welfare \|NA \| \|Alzheimer’s &
dementia (New York, N. Y.) \|Recent clinical trials are considering
inclusion of more than just apolipoprotein E (APOE) ε4 genotype as a way
of reducing variability in analysis of outcomes. Case-control data were
used to compare the capacity of age, sex, and 58 Alzheimer’s disease
(AD)-associated single nucleotide polymorphisms (SNPs) to predict AD
status using several statistical models. Model performance was assessed
with Brier scores and tenfold cross-validation. Genotype and sex × age
estimates from the best performing model were combined with age and
intercept estimates from the general population to develop a
personalized genetic risk score, termed age, and sex-adjusted GenoRisk.
The elastic net model that included age, age x sex interaction, allelic
APOE terms, and 29 additional SNPs performed the best. This model
explained an additional 19% of the heritable risk compared to APOE
genotype alone and achieved an area under the curve of 0.747. GenoRisk
could improve the risk assessment of individuals identified for
prevention studies. © 2021 The Authors. Alzheimer’s & Dementia:
Diagnosis, Assessment & Disease Monitoring published by Wiley
Periodicals, LLC on behalf of Alzheimer’s Association. \| \|Infectious
disease reports \|Given that the success of vaccines against coronavirus
disease 2019 (COVID-19) relies on herd immunity, identifying patients at
risk for vaccine hesitancy is imperative-particularly for those at high
risk for severe COVID-19 (i.e., minorities and patients with
neurological disorders). Among patients from a large neuroscience
institute in Hawaii, vaccine hesitancy was investigated in relation to
over 30 sociodemographic variables and medical comorbidities, via a
telephone quality improvement survey conducted between 23 January 2021
and 13 February 2021. Vaccine willingness (n = 363) was 81.3%.
Univariate analysis identified that the odds of vaccine acceptance
reduced for patients who do not regard COVID-19 as a severe illness, are
of younger age, have a lower Charlson Comorbidity Index, use illicit
drugs, or carry Medicaid insurance. Multivariable logistic regression
identified the best predictors of vaccine hesitancy to be: social media
use to obtain COVID-19 information, concerns regarding vaccine safety,
self-perception of a preexisting medical condition contraindicated with
vaccination, not having received the annual influenza vaccine, having
some high school education only, being a current smoker, and not having
a prior cerebrovascular accident. Unique amongst males, a conservative
political view strongly predicted vaccine hesitancy. Specifically for
Asians, a higher body mass index, while for Native Hawaiians and other
Pacific Islanders (NHPI), a positive depression screen, both reduced the
odds of vaccine acceptance. Upon identifying the variables associated
with vaccine hesitancy amongst patients with neurological disorders, our
clinic is now able to efficiently provide ancillary COVID-19 education
to sub-populations at risk for vaccine hesitancy. While our results may
be limited to the sub-population of patients with neurological
disorders, the findings nonetheless provide valuable insight to
understanding vaccine hesitancy. \| \|The primary care companion for CNS
disorders \|NA \| \|CA: a cancer journal for clinicians
<ISOAbbreviation>CA Cancer J Clin</ISOAbbreviation> </Journal>
<ArticleTitle>Cancer statistics for the US Hispanic/Latino population,
2021.</ArticleTitle> <Pagination> <StartPage>466</StartPage>
<EndPage>487</EndPage> <MedlinePgn>466-487</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.3322/caac.21695</ELocationID>
<Abstract> <AbstractText>The Hispanic/Latino population is the second
largest racial/ethnic group in the continental United States and Hawaii,
accounting for 18% (60.6 million) of the total population. An additional
3 million Hispanic Americans live in Puerto Rico. Every 3 years, the
American Cancer Society reports on cancer occurrence, risk factors, and
screening for Hispanic individuals in the United States using the most
recent population-based data. An estimated 176,600 new cancer cases and
46,500 cancer deaths will occur among Hispanic individuals in the
continental United States and Hawaii in 2021. Compared to non-Hispanic
Whites (NHWs), Hispanic men and women had 25%-30% lower incidence
(2014-2018) and mortality (2015-2019) rates for all cancers combined and
lower rates for the most common cancers, although this gap is
diminishing. For example, the colorectal cancer (CRC) incidence rate
ratio for Hispanic compared with NHW individuals narrowed from 0.75 (95%
CI, 0.73-0.78) in 1995 to 0.91 (95% CI, 0.89-0.93) in 2018, reflecting
delayed declines in CRC rates among Hispanic individuals in part because
of slower uptake of screening. In contrast, Hispanic individuals have
higher rates of infection-related cancers, including approximately
two-fold higher incidence of liver and stomach cancer. Cervical cancer
incidence is 32% higher among Hispanic women in the continental US and
Hawaii and 78% higher among women in Puerto Rico compared to NHW women,
yet is largely preventable through screening. Less access to care may be
similarly reflected in the low prevalence of localized-stage breast
cancer among Hispanic women, 59% versus 67% among NHW women.
Evidence-based strategies for decreasing the cancer burden among the
Hispanic population include the use of culturally appropriate lay health
advisors and patient navigators and targeted, community-based
intervention programs to facilitate access to screening and promote
healthy behaviors. In addition, the impact of the COVID-19 pandemic on
cancer trends and disparities in the Hispanic population should be
closely monitored.</AbstractText> <CopyrightInformation>© 2021 The
Authors. CA: A Cancer Journal for Clinicians published by Wiley
Periodicals LLC on behalf of American Cancer
Society.</CopyrightInformation> </Abstract> <AuthorList CompleteYN="Y">
<Author ValidYN="Y"> <LastName>Miller</LastName> <ForeName>Kimberly
D</ForeName> <Initials>KD</Initials>
<Identifier Source="ORCID">0000-0002-2609-2260</Identifier>
<AffiliationInfo> <Affiliation>Surveillance and Health Services
Research, American Cancer Society, Atlanta, Georgia.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Ortiz</LastName> <ForeName>Ana P</ForeName>
<Initials>AP</Initials> <AffiliationInfo> <Affiliation>Cancer Control
and Population Sciences, University of Puerto Rico Comprehensive Cancer
Center, San Juan, Puerto Rico.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Pinheiro</LastName>
<ForeName>Paulo S</ForeName> <Initials>PS</Initials> <AffiliationInfo>
<Affiliation>Sylvester Comprehensive Cancer Center, University of Miami
Health System, Miami, Florida.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Bandi</LastName>
<ForeName>Priti</ForeName> <Initials>P</Initials> <AffiliationInfo>
<Affiliation>Surveillance and Health Services Research, American Cancer
Society, Atlanta, Georgia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Minihan</LastName>
<ForeName>Adair</ForeName> <Initials>A</Initials> <AffiliationInfo>
<Affiliation>Surveillance and Health Services Research, American Cancer
Society, Atlanta, Georgia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Fuchs</LastName> <ForeName>Hannah
E</ForeName> <Initials>HE</Initials> <AffiliationInfo>
<Affiliation>Surveillance and Health Services Research, American Cancer
Society, Atlanta, Georgia.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Martinez Tyson</LastName>
<ForeName>Dinorah</ForeName> <Initials>D</Initials> <AffiliationInfo>
<Affiliation>College of Public Health, University of South Florida,
Tampa, Florida.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Tortolero-Luna</LastName>
<ForeName>Guillermo</ForeName> <Initials>G</Initials> <AffiliationInfo>
<Affiliation>Cancer Control and Population Sciences, University of
Puerto Rico Comprehensive Cancer Center, San Juan, Puerto
Rico.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Fedewa</LastName> <ForeName>Stacey A</ForeName>
<Initials>SA</Initials> <AffiliationInfo> <Affiliation>Surveillance and
Health Services Research, American Cancer Society, Atlanta,
Georgia.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Jemal</LastName> <ForeName>Ahmedin M</ForeName>
<Initials>AM</Initials> <AffiliationInfo> <Affiliation>Surveillance and
Health Services Research, American Cancer Society, Atlanta,
Georgia.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Siegel</LastName> <ForeName>Rebecca L</ForeName>
<Initials>RL</Initials>
<Identifier Source="ORCID">0000-0001-5247-8522</Identifier>
<AffiliationInfo> <Affiliation>Surveillance and Health Services
Research, American Cancer Society, Atlanta, Georgia.</Affiliation>
</AffiliationInfo> </Author> </AuthorList> <Language>eng</Language>
<PublicationTypeList> <PublicationType UI="D016428">Journal
Article</PublicationType> <PublicationType UI="D052061">Research
Support, N.I.H., Extramural</PublicationType> </PublicationTypeList>
<ArticleDate DateType="Electronic"> <Year>2021</Year> <Month>09</Month>
<Day>21</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>United States</Country>
      <MedlineTA>CA Cancer J Clin</MedlineTA>
      <NlmUniqueID>0370647</NlmUniqueID>
      <ISSNLinking>0007-9235</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000293" MajorTopicYN="N">Adolescent</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000328" MajorTopicYN="N">Adult</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000368" MajorTopicYN="N">Aged</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D055088" MajorTopicYN="N">Early Detection of Cancer</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="Y">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006297" MajorTopicYN="N">Health Services Accessibility</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="Y">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006630" MajorTopicYN="N">Hispanic or Latino</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="Y">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D015994" MajorTopicYN="N">Incidence</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008297" MajorTopicYN="N">Male</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008875" MajorTopicYN="N">Middle Aged</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D009369" MajorTopicYN="N">Neoplasms</DescriptorName>
        <QualifierName UI="Q000208" MajorTopicYN="Y">ethnology</QualifierName>
        <QualifierName UI="Q000401" MajorTopicYN="N">mortality</QualifierName>
        <QualifierName UI="Q000517" MajorTopicYN="N">prevention &amp; control</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011647" MajorTopicYN="N" Type="Geographic">Puerto Rico</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D012307" MajorTopicYN="N">Risk Factors</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D015996" MajorTopicYN="N">Survival Rate</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D014481" MajorTopicYN="N" Type="Geographic">United States</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D044465" MajorTopicYN="N">White People</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="N">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D055815" MajorTopicYN="N">Young Adult</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">Hispanics</Keyword>
      <Keyword MajorTopicYN="N">Latinos</Keyword>
      <Keyword MajorTopicYN="N">statistics</Keyword>
      <Keyword MajorTopicYN="N">surveillance</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="received"> <Year>2021</Year> <Month>8</Month>
<Day>4</Day> </PubMedPubDate> <PubMedPubDate PubStatus="accepted">
<Year>2021</Year> <Month>8</Month> <Day>4</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="pubmed"> <Year>2021</Year> <Month>9</Month>
<Day>22</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="medline"> <Year>2021</Year> <Month>11</Month>
<Day>16</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="entrez"> <Year>2021</Year> <Month>9</Month>
<Day>21</Day> <Hour>8</Hour> <Minute>46</Minute> </PubMedPubDate>
</History> <PublicationStatus>ppublish</PublicationStatus>
<ArticleIdList> <ArticleId IdType="pubmed">34545941</ArticleId>
<ArticleId IdType="doi">10.3322/caac.21695</ArticleId> </ArticleIdList>
<ReferenceList> References \|The Hispanic/Latino population is the
second largest racial/ethnic group in the continental United States and
Hawaii, accounting for 18% (60.6 million) of the total population. An
additional 3 million Hispanic Americans live in Puerto Rico. Every 3
years, the American Cancer Society reports on cancer occurrence, risk
factors, and screening for Hispanic individuals in the United States
using the most recent population-based data. An estimated 176,600 new
cancer cases and 46,500 cancer deaths will occur among Hispanic
individuals in the continental United States and Hawaii in 2021.
Compared to non-Hispanic Whites (NHWs), Hispanic men and women had
25%-30% lower incidence (2014-2018) and mortality (2015-2019) rates for
all cancers combined and lower rates for the most common cancers,
although this gap is diminishing. For example, the colorectal cancer
(CRC) incidence rate ratio for Hispanic compared with NHW individuals
narrowed from 0.75 (95% CI, 0.73-0.78) in 1995 to 0.91 (95% CI,
0.89-0.93) in 2018, reflecting delayed declines in CRC rates among
Hispanic individuals in part because of slower uptake of screening. In
contrast, Hispanic individuals have higher rates of infection-related
cancers, including approximately two-fold higher incidence of liver and
stomach cancer. Cervical cancer incidence is 32% higher among Hispanic
women in the continental US and Hawaii and 78% higher among women in
Puerto Rico compared to NHW women, yet is largely preventable through
screening. Less access to care may be similarly reflected in the low
prevalence of localized-stage breast cancer among Hispanic women, 59%
versus 67% among NHW women. Evidence-based strategies for decreasing the
cancer burden among the Hispanic population include the use of
culturally appropriate lay health advisors and patient navigators and
targeted, community-based intervention programs to facilitate access to
screening and promote healthy behaviors. In addition, the impact of the
COVID-19 pandemic on cancer trends and disparities in the Hispanic
population should be closely monitored. © 2021 The Authors. CA: A Cancer
Journal for Clinicians published by Wiley Periodicals LLC on behalf of
American Cancer Society. \| \|The Lancet. Infectious diseases \|NA \|
\|NAM perspectives \|NA \| \|MMWR. Morbidity and mortality weekly report
\|Native Hawaiian and Pacific Islander populations have been
disproportionately affected by COVID-19 (1-3). Native Hawaiian, Pacific
Islander, and Asian populations vary in language; cultural practices;
and social, economic, and environmental experiences,† which can affect
health outcomes (4).§ However, data from these populations are often
aggregated in analyses. Although data aggregation is often used as an
approach to increase sample size and statistical power when analyzing
data from smaller population groups, it can limit the understanding of
disparities among diverse Native Hawaiian, Pacific Islander, and Asian
subpopulations¶ (4-7). To assess disparities in COVID-19 outcomes among
Native Hawaiian, Pacific Islander, and Asian populations, a
disaggregated, descriptive analysis, informed by recommendations from
these communities,\*\* was performed using race data from 21,005
COVID-19 cases and 449 COVID-19-associated deaths reported to the Hawaii
State Department of Health (HDOH) during March 1, 2020-February 28,
2021.†† In Hawaii, COVID-19 incidence and mortality rates per 100,000
population were 1,477 and 32, respectively during this period. In
analyses with race categories that were not mutually exclusive,
including persons of one race alone or in combination with one or more
races, Pacific Islander persons, who account for 5% of Hawaii’s
population, represented 22% of COVID-19 cases and deaths (COVID-19
incidence of 7,070 and mortality rate of 150). Native Hawaiian persons
experienced an incidence of 1,181 and a mortality rate of 15. Among
subcategories of Asian populations, the highest incidences were
experienced by Filipino persons (1,247) and Vietnamese persons (1,200).
Disaggregating Native Hawaiian, Pacific Islander, and Asian race data
can aid in identifying racial disparities among specific subpopulations
and highlights the importance of partnering with communities to develop
culturally responsive outreach teams§§ and tailored public health
interventions and vaccination campaigns to more effectively address
health disparities. \| \|PLoS computational biology \|Accurate estimates
of infection prevalence and seroprevalence are essential for evaluating
and informing public health responses and vaccination coverage needed to
address the ongoing spread of COVID-19 in each United States (U.S.)
state. However, reliable, timely data based on representative population
sampling are unavailable, and reported case and test positivity rates
are highly biased. A simple data-driven Bayesian semi-empirical modeling
framework was developed and used to evaluate state-level prevalence and
seroprevalence of COVID-19 using daily reported cases and test
positivity ratios. The model was calibrated to and validated using
published state-wide seroprevalence data, and further compared against
two independent data-driven mathematical models. The prevalence of
undiagnosed COVID-19 infections is found to be well-approximated by a
geometrically weighted average of the positivity rate and the reported
case rate. Our model accurately fits state-level seroprevalence data
from across the U.S. Prevalence estimates of our semi-empirical model
compare favorably to those from two data-driven epidemiological models.
As of December 31, 2020, we estimate nation-wide a prevalence of 1.4%
\[Credible Interval (CrI): 1.0%-1.9%\] and a seroprevalence of 13.2%
\[CrI: 12.3%-14.2%\], with state-level prevalence ranging from 0.2%
\[CrI: 0.1%-0.3%\] in Hawaii to 2.8% \[CrI: 1.8%-4.1%\] in Tennessee,
and seroprevalence from 1.5% \[CrI: 1.2%-2.0%\] in Vermont to 23% \[CrI:
20%-28%\] in New York. Cumulatively, reported cases correspond to only
one third of actual infections. The use of this simple and
easy-to-communicate approach to estimating COVID-19 prevalence and
seroprevalence will improve the ability to make public health decisions
that effectively respond to the ongoing COVID-19 pandemic. \| \|Public
health \|During the COVID-19 pandemic, the prevalence of psychological
distress rose from 11% in 2019 to more than 40% in 2020. This study aims
to examine the disparities among US adult men and women. We used 21
waves of cross-sectional data from the Household Pulse Survey that were
collected between April and December 2020 for the study. The Household
Pulse Survey was developed by the U.S. Census Bureau to document the
social and economic impact of COVID-19. The study population included
four groups of adults: emerging adults (18-24 years); young adults
(25-44 years); middle-aged adults (45-64 years); and older adults (65-88
years). Psychological distress was measured by their Generalized Anxiety
Disorder score and the Patient Health Questionnaire. The prevalence of
psychological stress was calculated using logistic models adjusted for
socio-demographic variables including race/ethnicity, education,
household income, and household structure. All descriptive and
regression analysis considered survey weights. Younger age groups
experienced higher prevalence of psychological distress than older age
groups. Among emerging adults, the prevalence of anxiety (42.6%) and
depression (39.5%) was more than twice as high as older adults who
experienced prevalence of anxiety at 20% and depression at 16.6%. Gender
differences were also more apparent in emerging adults. Women between 18
and 24 years reported higher differential rates of anxiety and
depression than those with men (anxiety: 43.9% vs. 28.3%; depression:
33.3% vs. 24.9%). Understanding the complex dynamics between COVID-19
and psychological distress has emerged as a public health priority.
Mitigating the negative mental health consequences associated with the
COVID-19 pandemic, for younger generations and females in particular,
will require local efforts to rebuild capacity for social integration
and social connection. Copyright © 2021 The Royal Society for Public
Health. Published by Elsevier Ltd. All rights reserved. \| \|JAMA
\|People who have been infected with or vaccinated against SARS-CoV-2
have reduced risk of subsequent infection, but the proportion of people
in the US with SARS-CoV-2 antibodies from infection or vaccination is
uncertain. To estimate trends in SARS-CoV-2 seroprevalence related to
infection and vaccination in the US population. In a repeated
cross-sectional study conducted each month during July 2020 through May
2021, 17 blood collection organizations with blood donations from all 50
US states; Washington, DC; and Puerto Rico were organized into 66
study-specific regions, representing a catchment of 74% of the US
population. For each study region, specimens from a median of
approximately 2000 blood donors were selected and tested each month; a
total of 1 594 363 specimens were initially selected and tested. The
final date of blood donation collection was May 31, 2021. Calendar time.
Proportion of persons with detectable SARS-CoV-2 spike and nucleocapsid
antibodies. Seroprevalence was weighted for demographic differences
between the blood donor sample and general population. Infection-induced
seroprevalence was defined as the prevalence of the population with both
spike and nucleocapsid antibodies. Combined infection- and
vaccination-induced seroprevalence was defined as the prevalence of the
population with spike antibodies. The seroprevalence estimates were
compared with cumulative COVID-19 case report incidence rates. Among 1
443 519 specimens included, 733 052 (50.8%) were from women, 174 842
(12.1%) were from persons aged 16 to 29 years, 292 258 (20.2%) were from
persons aged 65 years and older, 36 654 (2.5%) were from non-Hispanic
Black persons, and 88 773 (6.1%) were from Hispanic persons. The overall
infection-induced SARS-CoV-2 seroprevalence estimate increased from 3.5%
(95% CI, 3.2%-3.8%) in July 2020 to 20.2% (95% CI, 19.9%-20.6%) in May
2021; the combined infection- and vaccination-induced seroprevalence
estimate in May 2021 was 83.3% (95% CI, 82.9%-83.7%). By May 2021, 2.1
SARS-CoV-2 infections (95% CI, 2.0-2.1) per reported COVID-19 case were
estimated to have occurred. Based on a sample of blood donations in the
US from July 2020 through May 2021, vaccine- and infection-induced
SARS-CoV-2 seroprevalence increased over time and varied by age, race
and ethnicity, and geographic region. Despite weighting to adjust for
demographic differences, these findings from a national sample of blood
donors may not be representative of the entire US population. \|
\|Clinical toxicology (Philadelphia, Pa.) \|Our six goals are: 1)
describe the relationship between the National Strategy for the COVID-19
Response and Pandemic Preparedness and the 55 US poison centers (PCs);
2) detail FDA emergency Use Authorization (EUA) COVID-19 vaccine-related
regulatory procedures and associated acronyms; 3) list availability of
specific vaccine clinical information to support PC staff COVID-19
vaccination and adverse event (AE) data collection; 4) describe required
health care practitioner COVID-19 vaccine AE reporting to the Vaccine AE
Reporting System (VAERS) and PC reporting options; 5) document public
and health care professionals’ use of PCs for COVID-19 vaccine
information; and 6) propose strategy to maximize PCs contribution to the
pandemic solution. We reviewed 13-Feb-2020 through 15-Apr-2021 National
Poison Data System (NPDS) COVID-19 records for changes over time. We
examined NPDS cases and VAERS COVID-19 vaccine reports 1-Nov-2020
through 2-Apr-2021 for vaccine manufacturer, patient characteristics,
state, and clinical effects. PCs reported 1,052,174 COVID-19 contacts;
maximum (peak) contacts/day (12,163) on 16-Mar-2020. As of 5-Apr-2021
the US reported \>167 million administrations of COVID-19 vaccines
(Pfizer-BioNTech, Moderna or Janssen). US PCs reported 162,052 COVID-19
vaccine contacts. Most (61.1%) were medical information calls, 34.9%
were drug information, and 2.58% were exposures. Over the same period
VAERS reported 49,078 COVID-19 vaccine cases reporting 226,205
symptoms - headache most frequent, ranging from 20% to 40% across the 3
vaccines. Although differences exist between the intent and content of
the 2 data sets, NPDS volume is compelling. The PC nationwide 800 number
facilitates data collection and suggests comingling the 2 data streams
has merit. PC professionals received tens of thousands of calls and
can: 1) support fact-based vaccine information; 2) contribute vaccine AE
follow-up information: 3) advocate for best-case coding and reporting,
especially for vaccine adverse experiences. \| \|Paediatric respiratory
reviews \|Mathematical modelling has played a pivotal role in
understanding the epidemiology of and guiding public health responses to
the ongoing coronavirus disease of 2019 (COVID-19) pandemic. Here, we
review the role of epidemiological models in understanding evolving
epidemic characteristics, including the effects of vaccination and
Variants of Concern (VoC). We highlight ways in which models continue to
provide important insights, including (1) calculating the herd immunity
threshold and evaluating its limitations; (2) verifying that nascent
vaccines can prevent severe disease, infection, and transmission but may
be less efficacious against VoC; (3) determining optimal vaccine
allocation strategies under efficacy and supply constraints; and (4)
determining that VoC are more transmissible and lethal than previously
circulating strains, and that immune escape may jeopardize
vaccine-induced herd immunity. Finally, we explore how models can help
us anticipate and prepare for future stages of COVID-19 epidemiology
(and that of other diseases) through forecasts and scenario projections,
given current uncertainties and data limitations. Copyright © 2021.
Published by Elsevier Ltd. \| \|Health affairs (Project Hope) \|COVID-19
vaccination campaigns continue in the United States, with the
expectation that vaccines will slow transmission of the virus, save
lives, and enable a return to normal life in due course. However, the
extent to which faster vaccine administration has affected
COVID-19-related deaths is unknown. We assessed the association between
US state-level vaccination rates and COVID-19 deaths during the first
five months of vaccine availability. We estimated that by May 9, 2021,
the US vaccination campaign was associated with a reduction of 139,393
COVID-19 deaths. The association varied in different states. In New
York, for example, vaccinations led to an estimated 11.7 fewer COVID-19
deaths per 10,000, whereas Hawaii observed the smallest reduction, with
an estimated 1.1 fewer deaths per 10,000. Overall, our analysis suggests
that the early COVID-19 vaccination campaign was associated with
reductions in COVID-19 deaths. As of May 9, 2021, reductions in COVID-19
deaths associated with vaccines had translated to value of statistical
life benefit ranging between \$625 billion and \$1.4 trillion. \|
\|International journal of infectious diseases : IJID : official
publication of the International Society for Infectious Diseases \|To
evaluate the impact of the World Antimicrobial Awareness Week (WAAW) on
public awareness of antimicrobial resistance using Google Trends
analysis. The impact of WAAW on public awareness of ‘antimicrobial
resistance’ (AMR), ‘antibacterial’, and ‘antibiotics’ in Japan, the UK,
the United States, and worldwide from 2015 to 2020 was analyzed, using
the relative search volume (RSV) of Google Trends as a surrogate. A
joinpoint regression analysis was performed to identify a statistically
significant time point of a change in trend. No joinpoints around WAAW
were identified in Japan, the United Kingdom, or the United States from
2015 to 2020 with RSVs of ‘AMR’, whereas increasing RSVs were noted
worldwide in 2017 and 2020. Further, there were decreasing RSVs of
‘antibiotics’ in the first half of 2020, which could be due to the
COVID-19 pandemic. The study results suggest that WAAW did little to
improve public awareness of AMR in the selected countries despite its
contribution worldwide. This study implies that we need to develop a
more effective method to improve public awareness to fight against AMR.
Copyright © 2021 The Author(s). Published by Elsevier Ltd.. All rights
reserved. \| \|Clinical & experimental pharmacology \|The impact of
COVID-19 disease on health and economy has been global, and the
magnitude of devastation is unparalleled in modern history. Any
potential course of action to manage this complex disease requires the
systematic and efficient analysis of data that can delineate the
underlying pathogenesis. We have developed a mathematical model of
disease progression to predict the clinical outcome, utilizing a set of
causal factors known to contribute to COVID-19 pathology such as age,
comorbidities, and certain viral and immunological parameters. Viral
load and selected indicators of a dysfunctional immune response, such as
cytokines IL-6 and IFNα which contribute to the cytokine storm and
fever, parameters of inflammation D-Dimer and Ferritin, aberrations in
lymphocyte number, lymphopenia, and neutralizing antibodies were
included for the analysis. The model provides a framework to unravel the
multi-factorial complexities of the immune response manifested in
SARS-CoV-2 infected individuals. Further, this model can be valuable to
predict clinical outcome at an individual level, and to develop
strategies for allocating appropriate resources to manage severe cases
at a population level. \| \|Hawai’i journal of health & social welfare
\|Native Hawaiian and Pacific Islander (NHPI) populations suffer from
disproportionately higher rates of chronic conditions, such as type 2
diabetes, that arises from metabolic dysfunction and are often
associated with obesity and inflammation. In addition, the global
coronavirus disease 2019 pandemic has further compounded the effect of
health inequities observed in Indigenous populations, including NHPI
communities. Reversible lifestyle habits, such as diet, may either be
protective of or contribute to the increasing prevalence of health
inequities in these populations via the immunoepigenetic-microbiome
axis. This axis offers insight into the connection between diet,
epigenetics, the microbiome composition, immune function, and response
to viral infection. Epigenetic mechanisms that regulate inflammatory
states associated with metabolic diseases, including diabetes, are
impacted by diet. Furthermore, diet may modulate the gut microbiome by
influencing microbial diversity and richness; dysbiosis of the
microbiome is associated with chronic disease. A high fiber diet
facilitates a favorable microbiome composition and in turn increases
production of intermediate metabolites named short-chain fatty acids
(SCFAs) that act on metabolic and immune pathways. In contrast, low
fiber diets typically associated with a westernized lifestyle decreases
the abundance of microbial derived SCFAs. This decreased abundance is
characteristic of metabolic syndromes and activation of chronic
inflammatory states, having larger implications in disease pathogenesis
of both communicable and non-communicable diseases. Native Hawaiians and
Pacific Islanders that once thrived on healthy traditional diets may be
more sensitive than non-indigenous peoples to the metabolic perturbation
of westernized diets that impinge on the immunoepigenetic-gut microbiome
axis. Recent studies conducted in the Maunakea lab at the University of
Hawai’i at Mānoa John A. Burns School of Medicine have helped elucidate
the connections between diet, microbiome composition, metabolic
syndrome, and epigenetic regulation of immune function to better
understand disease pathogenesis. Potentially, this research could point
to ways to prevent pre-disease conditions through novel biomarker
discovery using community-based approaches. ©Copyright 2021 by
University Health Partners of Hawai‘i (UHP Hawai‘i). \| \|Social science
& medicine (1982) \|The United States experienced three surges of
COVID-19 community infection since the World Health Organization
declared the pandemic on March 11, 2020. The prevalence of psychological
distress among U.S. adults increased from 11 % in 2019 to 35.9 % in
April 2020 when New York City become the epicenter of the COVID-19
outbreak. Analyzing 21 waves of the Household Pulse Survey data
collected between April 2020 and December 2020, this study aimed to
examine the distress level in the 15 most populated metropolitan areas
in the U.S. Our study found that, as the pandemic swept from East to
South and soared in the West, 39.9%-52.3 % U.S. adults living in these
15 metropolitan areas reported symptoms of psychological distress. The
highest distress levels were found within the Western areas including
Riverside-San Bernardino-Ontario (52.3 % in July 2020, 95 % CI:
44.9%-59.6 %) and Los Angeles-Long Beach-Anaheim (49.9 % in December
2020, 95 % CI: 44.5%-55.4 %). The lowest distress level was observed in
Washington-Arlington-Alexandria ranging from 29.1 % in May 2020 to 39.9
% in November 2020. COVID-19 and its complex ecology of social and
economic stressors have engaged high levels of sustained psychological
distress. Our findings will support the efforts of local, state and
national leadership to plan interventions by addressing not only the
medical, but also the economic and social conditions associated with the
pandemic. Copyright © 2021 Elsevier Ltd. All rights reserved. \|
\|Seminars in arthroplasty \|Although the COVID-19 pandemic has
disrupted elective shoulder arthroplasty throughput, traumatic shoulder
arthroplasty procedures are less apt to be postponed. We sought to
evaluate shoulder arthroplasty utilization for fracture during the
COVID-19 pandemic and California’s associated shelter-in-place order
compared to historical controls. We conducted a cohort study with
historical controls, identifying patients who underwent shoulder
arthroplasty for proximal humerus fracture in California using our
integrated electronic health record. The time period of interest was
following the implementation of the statewide shelter-in-place order:
March 19, 2020-May 31, 2020. This was compared to three historical
periods: January 1, 2020-March 18, 2020, March 18, 2019-May 31, 2019,
and January 1, 2019-March 18, 2019. Procedure volume, patient
characteristics, in-hospital length of stay, and 30-day events
(emergency department visit, readmission, infection, pneumonia, and
death) were reported. Changes over time were analyzed using linear
regression adjusted for usual seasonal and yearly changes and age, sex,
comorbidities, and postadmission factors. Surgical volume dropped from
an average of 4.4, 5.2, and 2.6 surgeries per week in the historical
time periods, respectively, to 2.4 surgeries per week after
shelter-in-place. While no more than 30% of all shoulder arthroplasty
procedures performed during any given week were for fracture during the
historical time periods, arthroplasties performed for fracture was the
overwhelming primary indication immediately after the shelter-in-place
order. More patients were discharged the day of surgery (+33.2%, P =
.019) after the shelter-in-place order, but we did not observe a change
in any of the corresponding 30-day events. The volume of shoulder
arthroplasty for fracture dropped during the time of COVID-19. The
reduction in volume could be due to less shoulder trauma due to
shelter-in-place or a change in the indications for arthroplasty given
the perceived higher risks associated with intubation and surgical care.
We noted more patients undergoing shoulder arthroplasty for fracture
were safely discharged on the day of surgery, suggesting this may be a
safe practice that can be adopted moving forward. Level III;
Retrospective Case-control Comparative Study. © 2021 American Shoulder
and Elbow Surgeons. Published by Elsevier Inc. All rights reserved. \|
\|The Journal of infectious diseases \|The mechanisms underlying the
association between obesity and coronavirus disease 2019 (COVID-19)
severity remain unclear. After verifying that obesity was a correlate of
severe COVID-19 in US Military Health System (MHS) beneficiaries, we
compared immunological and virological phenotypes of severe acute
respiratory syndrome coronavirus 2 (SARS-CoV-2) infection in both obese
and nonobese participants. COVID-19-infected MHS beneficiaries were
enrolled, and anthropometric, clinical, and demographic data were
collected. We compared the SARS-CoV-2 peak IgG humoral response and
reverse-transcription polymerase chain reaction viral load in obese and
nonobese patients, stratified by hospitalization, utilizing logistic
regression models. Data from 511 COVID-19 patients were analyzed, among
whom 24% were obese and 14% severely obese. Obesity was independently
associated with hospitalization (adjusted odds ratio \[aOR\], 1.91; 95%
confidence interval \[CI\], 1.15-3.18) and need for oxygen therapy (aOR,
3.39; 95% CI, 1.61-7.11). In outpatients, severely obese had a log10
(1.89) higher nucleocapsid (N1) genome equivalents (GE)/reaction and
log10 (2.62) higher N2 GE/reaction than nonobese (P = 0.03 and P \<
.001, respectively). We noted a correlation between body mass index and
peak anti-spike protein IgG in inpatients and outpatients (coefficient =
5.48, P \< .001). Obesity is a strong correlate of COVID-19 severity in
MHS beneficiaries. These findings offer new pathophysiological insights
into the relationship between obesity and COVID-19 severity. © The
Author(s) 2021. Published by Oxford University Press for the Infectious
Diseases Society of America. All rights reserved. For permissions,
e-mail: <journals.permissions@oup.com>. \| \|American journal of public
health \|As of March 2021, Native Hawaiians and Pacific Islanders
(NHPIs) in the United States have lost more than 800 lives to
COVID-19-the highest per capita death rate in 18 of 20 US states
reporting NHPI deaths. However, NHPI risks are overlooked in policy
discussions. We discuss the NHPI COVID-19 Data Policy Lab and dashboard,
featuring the disproportionate COVID-19 mortality burden for NHPIs. The
Lab democratized NHPI data, developed community infrastructure and
resources, and informed testing site and outreach policies related to
health equity. \| \|The Lancet regional health. Western Pacific
\|COVID-19 initially caused less severe outbreaks in many low- and
middle-income countries (LMIC) compared with many high-income countries,
possibly because of differing demographics, socioeconomics,
surveillance, and policy responses. Here, we investigate the role of
multiple factors on COVID-19 dynamics in the Philippines, a LMIC that
has had a relatively severe COVID-19 outbreak. We applied an
age-structured compartmental model that incorporated time-varying
mobility, testing, and personal protective behaviors (through a “Minimum
Health Standards” policy, MHS) to represent the first wave of the
Philippines COVID-19 epidemic nationally and for three highly affected
regions (Calabarzon, Central Visayas, and the National Capital Region).
We estimated effects of control measures, key epidemiological
parameters, and interventions. Population age structure, contact rates,
mobility, testing, and MHS were sufficient to explain the Philippines
epidemic based on the good fit between modelled and reported cases,
hospitalisations, and deaths. The model indicated that MHS reduced the
probability of transmission per contact by 13-27%. The February 2021
case detection rate was estimated at \~8%, population recovered at \~9%,
and scenario projections indicated high sensitivity to MHS adherence.
COVID-19 dynamics in the Philippines are driven by age, contact
structure, mobility, and MHS adherence. Continued compliance with
low-cost MHS should help the Philippines control the epidemic until
vaccines are widely distributed, but disease resurgence may be occurring
due to a combination of low population immunity and detection rates and
new variants of concern. © 2021 Published by Elsevier Ltd. \|
\|EClinicalMedicine \|There is limited prior investigation of the
combined influence of personal and community-level socioeconomic factors
on racial/ethnic disparities in individual risk of coronavirus disease
2019 (COVID-19). We performed a cross-sectional analysis nested within a
prospective cohort of 2,102,364 participants from March 29, 2020 in the
United States (US) and March 24, 2020 in the United Kingdom (UK) through
December 02, 2020 via the COVID Symptom Study smartphone application. We
examined the contribution of community-level deprivation using the
Neighborhood Deprivation Index (NDI) and the Index of Multiple
Deprivation (IMD) to observe racial/ethnic disparities in COVID-19
incidence. ClinicalTrials.gov registration: NCT04331509. Compared with
non-Hispanic White participants, the risk for a positive COVID-19 test
was increased in the US for non-Hispanic Black (multivariable-adjusted
odds ratio \[OR\], 1.32; 95% confidence interval \[CI\], 1.18-1.47) and
Hispanic participants (OR, 1.42; 95% CI, 1.33-1.52) and in the UK for
Black (OR, 1.17; 95% CI, 1.02-1.34), South Asian (OR, 1.39; 95% CI,
1.30-1.49), and Middle Eastern participants (OR, 1.38; 95% CI,
1.18-1.61). This elevated risk was associated with living in more
deprived communities according to the NDI/IMD. After accounting for
downstream mediators of COVID-19 risk, community-level deprivation still
mediated 16.6% and 7.7% of the excess risk in Black compared to White
participants in the US and the UK, respectively. Our results illustrate
the critical role of social determinants of health in the
disproportionate COVID-19 risk experienced by racial and ethnic
minorities. © 2021 The Author(s). \| \|Journal of traumatic stress \|The
onset of the COVID-19 pandemic disrupted many aspects of daily life and
required a rapid and unprecedented shift in psychotherapy delivery from
in-person to telemental health. In the present study, we explored the
impact of the pandemic on individuals’ ability to participate in
posttraumatic stress disorder (PTSD) psychotherapy and the association
between the impact of COVID-19 impact on health and financial well-being
and psychotherapy participation. Participants (N = 161, 63.2% male, Mage
= 42.7 years) were United States military veterans (n = 108), active
duty military personnel (n = 12), and civilians (n = 6), who were
participating in one of nine PTSD treatment trials. The results indicate
a predominately negative COVID-19 impact on therapy participation,
although some participants (26.1%) found attending therapy sessions
through telehealth to be easier than in-person therapy. Most
participants (66.7%) reported that completing in vivo exposure homework
became harder during the pandemic. Moreover, the impact of the pandemic
on PTSD symptom severity and daily stress were each associated with
increased difficulty with aspects of therapy participation. The findings
highlight the unique challenges to engaging in PTSD treatment during the
pandemic as well as a negative impact on daily stress and PTSD severity,
both of which were related to treatment engagement difficulties. © 2021
International Society for Traumatic Stress Studies. \| \|Journal of
travel medicine \|NA \| \|Journal of the American Medical Directors
Association \|NA \| \|Journal of child and adolescent psychopharmacology
\|Objectives: Our goal was to develop an open access nationally
disseminated online curriculum for use in graduate and continuing
medical education on the topic of pediatric telepsychiatry to enhance
the uptake of telepsychiatry among child psychiatry training programs
and improve access to mental health care for youth and families.
Methods: Following Kern’s 6-stage model of curriculum development, we
identified a core problem, conducted a needs assessment, developed broad
goals and measurable objectives in a competency-based model, and
developed educational content and methods. The curriculum was reviewed
by experts and feedback incorporated. Given the urgent need for such a
curriculum due to the COVID-19 pandemic, the curriculum was immediately
posted on the American Academy of Child and Adolescent Psychiatry and
American Association of Directors of Psychiatric Residency Training
websites. Further evaluation will be conducted over the next year.
Results: The curriculum covers the six areas of core competence adapted
for pediatric telepsychiatry and includes teaching content and
resources, evaluation tools, and information about other resources.
Conclusion: This online curriculum is available online and provides an
important resource and set of standards for pediatric telepsychiatry
training. Its online format allows for ongoing revision as the
telepsychiatry landscape changes. \| \|Human vaccines &
immunotherapeutics \|Nurses are the largest single occupation of health
care providers and at greatest risk for exposure to and acquisition of
Coronavirus Disease 2019 (COVID-19). In December 2020, nurses in Hawaii
were recruited for an online survey that measured perceived risk/threat
of COVID-19, vaccine attitudes, and perceived safety of COVID-19
vaccines, as well as level of intention: primary, secondary (i.e.,
delayed), or no intention to vaccinate. The final sample consisted of
423 nurses. Participants were primarily Asian (27.9%) and White (45.2%).
The majority were 18-50 years (65.5%) and female (87.0%), held an RN
license (91.7%), and identified as a staff nurse (57.7%) in the hospital
setting (56.7%). Among participants, 52.3% indicated primary intention,
27.9% secondary intention, and 19.9% no intention to vaccinate. The
strongest predictors of any level of intention were greater positive
attitudes toward COVID-19 vaccination and lower concerns related to
COVID-19 vaccine safety. Findings can guide interventions to support
vaccine acceptance for those who initially decline vaccination. \|
\|JAMA \|Clinical trials assessing the efficacy of IL-6 antagonists in
patients hospitalized for COVID-19 have variously reported benefit, no
effect, and harm. To estimate the association between administration of
IL-6 antagonists compared with usual care or placebo and 28-day
all-cause mortality and other outcomes. Trials were identified through
systematic searches of electronic databases between October 2020 and
January 2021. Searches were not restricted by trial status or language.
Additional trials were identified through contact with experts. Eligible
trials randomly assigned patients hospitalized for COVID-19 to a group
in whom IL-6 antagonists were administered and to a group in whom
neither IL-6 antagonists nor any other immunomodulators except
corticosteroids were administered. Among 72 potentially eligible trials,
27 (37.5%) met study selection criteria. In this prospective
meta-analysis, risk of bias was assessed using the Cochrane Risk of Bias
Assessment Tool. Inconsistency among trial results was assessed using
the I2 statistic. The primary analysis was an inverse variance-weighted
fixed-effects meta-analysis of odds ratios (ORs) for 28-day all-cause
mortality. The primary outcome measure was all-cause mortality at 28
days after randomization. There were 9 secondary outcomes including
progression to invasive mechanical ventilation or death and risk of
secondary infection by 28 days. A total of 10 930 patients (median age,
61 years \[range of medians, 52-68 years\]; 3560 \[33%\] were women)
participating in 27 trials were included. By 28 days, there were 1407
deaths among 6449 patients randomized to IL-6 antagonists and 1158
deaths among 4481 patients randomized to usual care or placebo (summary
OR, 0.86 \[95% CI, 0.79-0.95\]; P = .003 based on a fixed-effects
meta-analysis). This corresponds to an absolute mortality risk of 22%
for IL-6 antagonists compared with an assumed mortality risk of 25% for
usual care or placebo. The corresponding summary ORs were 0.83 (95% CI,
0.74-0.92; P \< .001) for tocilizumab and 1.08 (95% CI, 0.86-1.36; P =
.52) for sarilumab. The summary ORs for the association with mortality
compared with usual care or placebo in those receiving corticosteroids
were 0.77 (95% CI, 0.68-0.87) for tocilizumab and 0.92 (95% CI,
0.61-1.38) for sarilumab. The ORs for the association with progression
to invasive mechanical ventilation or death, compared with usual care or
placebo, were 0.77 (95% CI, 0.70-0.85) for all IL-6 antagonists, 0.74
(95% CI, 0.66-0.82) for tocilizumab, and 1.00 (95% CI, 0.74-1.34) for
sarilumab. Secondary infections by 28 days occurred in 21.9% of patients
treated with IL-6 antagonists vs 17.6% of patients treated with usual
care or placebo (OR accounting for trial sample sizes, 0.99; 95% CI,
0.85-1.16). In this prospective meta-analysis of clinical trials of
patients hospitalized for COVID-19, administration of IL-6 antagonists,
compared with usual care or placebo, was associated with lower 28-day
all-cause mortality. PROSPERO Identifier: CRD42021230155. \| \|Optimal
control applications & methods \|In this work, we develop and analyze a
mathematical model for the dynamics of COVID-19 with re-infection in
order to assess the impact of prior comorbidity (specifically, diabetes
mellitus) on COVID-19 complications. The model is simulated using data
relevant to the dynamics of the diseases in Lagos, Nigeria, making
predictions for the attainment of peak periods in the presence or
absence of comorbidity. The model is shown to undergo the phenomenon of
backward bifurcation caused by the parameter accounting for increased
susceptibility to COVID-19 infection by comorbid susceptibles as well as
the rate of reinfection by those who have recovered from a previous
COVID-19 infection. Simulations of the cumulative number of active cases
(including those with comorbidity), at different reinfection rates, show
infection peaks reducing with decreasing reinfection of those who have
recovered from a previous COVID-19 infection. In addition, optimal
control and cost-effectiveness analysis of the model reveal that the
strategy that prevents COVID-19 infection by comorbid susceptibles is
the most cost-effective of all the control strategies for the prevention
of COVID-19. © 2021 John Wiley & Sons Ltd. \| \|Epidemiology and
infection \|We estimate the delay-adjusted all-cause excess deaths
across 53 US jurisdictions. Using provisional data collected from
September through December 2020, we first identify a common mean
reporting delay of 2.8 weeks, whereas four jurisdictions have prolonged
reporting delays compared to the others: Connecticut (mean 5.8 weeks),
North Carolina (mean 10.4 weeks), Puerto Rico (mean 4.7 weeks) and West
Virginia (mean 5.5 weeks). After adjusting for reporting delays, we
estimate the percent change in all-cause excess mortality from March to
December 2020 with range from 0.2 to 3.6 in Hawaii to 58.4 to 62.4 in
New York City. Comparing the March-December with September-December 2020
periods, the highest increases in excess mortality are observed in South
Dakota (36.9-54.0), North Dakota (33.9-50.7) and Missouri (27.8-33.9).
Our findings indicate that analysis of provisional data requires caution
in interpreting the death counts in recent weeks, while one needs also
to account for heterogeneity in reporting delays of excess deaths among
US jurisdictions. \| \|Hawai’i journal of health & social welfare \|This
report describes the rapid implementation of a statewide observational
surveillance program to monitor the public’s wearing of face masks in
public spaces during community spread of Coronavirus disease 2019
(COVID-19). It describes how the Hawai’i State Department of Health
partnered with University of Hawai’i faculty to develop and implement
the surveillance program. The surveillance program involved organizing
volunteers to conduct weekly direct observations in designated
locations. A smartphone application (app) was created to record
real-time observational surveillance data. From September 5, 2020, to
March 13, 2021, a total of 84 577 observations were conducted across the
state. Eighty-three percent of those observed were correctly wearing a
face mask, 7% were wearing a face mask incorrectly, and 10% were not
wearing a mask. Following the 2-week pilot phase of the project,
volunteers were surveyed regarding facilitators and barriers for
conducting observations and motivations for volunteering. Feedback was
used to refine project procedures. With few states having implemented
such a surveillance program, the information reported in this article
may inform communities interested in tracking mask-wearing behaviors in
the context of the COVID-19 pandemic. ©Copyright 2021 by University
Health Partners of Hawai‘i (UHP Hawai‘i). \| \|International journal of
disaster risk reduction : IJDRR \|COVID-19 pandemic is devastating the
health, social, and economic well-being of citizens worldwide. The high
rates of morbidity and mortality and the absence of vaccines cause fear
among the people regardless of age, gender, or social status. People’s
fear is heightened by misinformation spread across all media types,
especially on social media. Filipino college students are one of the top
Internet users worldwide and are very active in social media. Hence they
are very prone to misinformation. This paper aims to ascertain the
levels of knowledge, precaution, and fear of COVID-19 of the college
students in Iloilo, Philippines, and determine the effects of their
information-seeking behavior on the variables above. This paper is a
cross-sectional survey that used a qualitative-quantitative method and
snowball sampling technique. Data were gathered among 228 college
students using an online survey instrument a few months after the
pandemic began. College students were knowledgeable of the basic facts
about the highly infectious COVID-19. However, the majority were
inclined to believe the myths and misinformation regarding the pandemic.
Television was the primary, most believable, and preferred source when
seeking information. The Internet as a preferred source of information
was significantly associated with a high level of knowledge. In
contrast, the information sourced from interpersonal channels were found
to make college students very cautious. The local presence of COVID-19
cases had caused college students to fear, likely exacerbated by the
plethora of information about the pandemic, mostly from Facebook. This
is the first study conducted on the effects of the information-seeking
behavior on the levels of knowledge, precaution, and fear of COVID-19 of
the college students in Iloilo, Philippines. © 2021 Elsevier Ltd. All
rights reserved. \| \|Genetics in medicine : official journal of the
American College of Medical Genetics \|It is critical to identify
putative causal targets for SARS coronavirus 2, which may guide drug
repurposing options to reduce the public health burden of COVID-19. We
applied complementary methods and multiphased design to pinpoint the
most likely causal genes for COVID-19 severity. First, we applied
cross-methylome omnibus (CMO) test and leveraged data from the COVID-19
Host Genetics Initiative (HGI) comparing 9,986 hospitalized COVID-19
patients and 1,877,672 population controls. Second, we evaluated
associations using the complementary S-PrediXcan method and leveraging
blood and lung tissue gene expression prediction models. Third, we
assessed associations of the identified genes with another COVID-19
phenotype, comparing very severe respiratory confirmed COVID versus
population controls. Finally, we applied a fine-mapping method,
fine-mapping of gene sets (FOGS), to prioritize putative causal genes.
Through analyses of the COVID-19 HGI using complementary CMO and
S-PrediXcan methods along with fine-mapping, XCR1, CCR2, SACM1L, OAS3,
NSF, WNT3, NAPSA, and IFNAR2 are identified as putative causal genes for
COVID-19 severity. We identified eight genes at five genomic loci as
putative causal genes for COVID-19 severity. © 2021. The Author(s),
under exclusive licence to the American College of Medical Genetics and
Genomics. \| \|Proceedings of the National Academy of Sciences of the
United States of America \|In this perspective, we draw on recent
scientific research on the coffee leaf rust (CLR) epidemic that severely
impacted several countries across Latin America and the Caribbean over
the last decade, to explore how the socioeconomic impacts from COVID-19
could lead to the reemergence of another rust epidemic. We describe how
past CLR outbreaks have been linked to reduced crop care and investment
in coffee farms, as evidenced in the years following the 2008 global
financial crisis. We discuss relationships between CLR incidence,
farmer-scale agricultural practices, and economic signals transferred
through global and local effects. We contextualize how current COVID-19
impacts on labor, unemployment, stay-at-home orders, and international
border policies could affect farmer investments in coffee plants and in
turn create conditions favorable for future shocks. We conclude by
arguing that COVID-19’s socioeconomic disruptions are likely to drive
the coffee industry into another severe production crisis. While this
argument illustrates the vulnerabilities that come from a globalized
coffee system, it also highlights the necessity of ensuring the
well-being of all. By increasing investments in coffee institutions and
paying smallholders more, we can create a fairer and healthier system
that is more resilient to future social-ecological shocks. \| \|Nursing
outlook \|In 2020, nursing educational programs were abruptly
interrupted and largely moved online due to the COVID-19 pandemic. To
explore nursing students’ perspectives about the effects of the pandemic
on their education and intention to join the nursing workforce.
Undergraduate nursing students from 5 universities across 5 United
States regions were invited to participate in an online survey to elicit
both quantitative and qualitative data. The final sample included
quantitative data on 772 students and qualitative data on 540 students.
Largely (65.1%), students reported that the pandemic strengthened their
desire to become a nurse; only 11% had considered withdrawing from
school. Qualitatively, students described the effect of the pandemic on
their psychosocial wellbeing, adjustment to online learning, and
challenges to clinical experiences. Findings highlighted the need to
develop emergency education preparedness plans that address student
wellbeing and novel collaborative partnerships between schools and
clinical partners. Copyright © 2021 Elsevier Inc. All rights reserved.
\| \|Cardiology in the young \|Two adolescent males presented within 3
days after the first and second dose of the BNT162b2 vaccine with chest
pain. Elevated troponin levels, ST segment elevation, and enhancement of
the myocardium in cardiac MRI suggested myocarditis. Left ventricular
function remained normal, symptoms resolved, and patients were
discharged in 4 days. BNT162b2 vaccine may be associated with
self-limited myocarditis in youth. \| \|BMC public health \|Influenza
immunization is a highly effective method of reducing illness,
hospitalization and mortality from this disease. However, influenza
vaccination rates in the U.S. remain below public health targets and
persistent structural inequities reduce the likelihood that Black,
American Indian and Alaska Native, Latina/o, Asian groups, and
populations of low socioeconomic status will receive the influenza
vaccine. We analyzed correlates of influenza vaccination rates using the
2019 Behavioral Risk Factor Surveillance System (BRFSS) in the year
2020. Our analysis compared influenza vaccination as the outcome of
interest with the variables age, sex, race, education, income,
geographic location, health insurance status, access to primary care,
history of delaying care due to cost, and comorbidities such as: asthma,
cardiovascular disease, hypertension, body mass index, cancer and
diabetes. Non-Hispanic White (46.5%) and Asian (44.1%) participants are
more likely to receive the influenza vaccine compared to Non-Hispanic
Black (36.7%), Hispanic (33.9%), American Indian/Alaskan Native (36.6%),
and Native Hawaiian/Other Pacific Islander (37.9%) participants. We
found persistent structural inequities that predict influenza
vaccination, within and across racial and ethnic groups, including not
having health insurance \[OR: 0.51 (0.47-0.55)\], not having regular
access to primary care \[OR: 0.50 (0.48-0.52)\], and the need to delay
medical care due to cost \[OR: 0.75 (0.71-0.79)\]. As COVID-19
vaccination efforts evolve, it is important for physicians and
policymakers to identify the structural impediments to equitable U.S.
influenza vaccination so that future vaccination campaigns are not
impeded by these barriers to immunization. \| \|Emerging microbes &
infections \|The SARS-CoV-2 B.1.1.7 lineage is highly infectious and as
of April 2021 accounted for 92% of COVID-19 cases in Europe and 59% of
COVID-19 cases in the U.S. It is defined by the N501Y mutation in the
receptor-binding domain (RBD) of the Spike (S) protein, and a few other
mutations. These include two mutations in the N terminal domain (NTD) of
the S protein, HV69-70del and Y144del (also known as Y145del due to the
presence of tyrosine at both positions). We recently identified several
emerging SARS-CoV-2 variants of concerns, characterized by Membrane (M)
protein mutations, including I82T and V70L. We now identify a
sub-lineage of B.1.1.7 that emerged through sequential acquisitions of
M:V70L in November 2020 followed by a novel S:D178H mutation first
observed in early February 2021. The percentage of B.1.1.7 isolates in
the US that belong to this sub-lineage increased from 0.15% in February
2021 to 1.8% in April 2021. To date, this sub-lineage appears to be
U.S.-specific with reported cases in 31 states, including Hawaii. As of
April 2021, it constituted 36.8% of all B.1.1.7 isolates in Washington.
Phylogenetic analysis and transmission inference with Nextstrain suggest
this sub-lineage likely originated in either California or Washington.
Structural analysis revealed that the S:D178H mutation is in the NTD of
the S protein and close to two other signature mutations of B.1.1.7,
HV69-70del and Y144del. It is surface exposed and may alter NTD tertiary
configuration or accessibility, and thus has the potential to affect
neutralization by NTD directed antibodies. \| \|Clinical infectious
diseases : an official publication of the Infectious Diseases Society of
America \|Little is known about severe acute respiratory syndrome
coronavirus 2 “vaccine-breakthrough” infections (VBIs). Here we
characterize 24 VBIs in predominantly young healthy persons. While none
required hospitalization, a proportion endorsed severe symptoms and shed
live virus as high as 4.13 × 103 plaque-forming units/mL. Infecting
genotypes included both variant-of-concern (VOC) and non-VOC strains. ©
The Author(s) 2021. Published by Oxford University Press for the
Infectious Diseases Society of America. \| \|Frontiers in psychiatry
\|Background: Social distancing and school suspension due to the
coronavirus pandemic (COVID-19) may have a negative impact on children’s
behavior and well-being. Problematic smartphone use (PSU), problematic
social media use (PSMU) and perceived weight stigma (PWS) are
particularly important issues for children, yet we have a poor
understanding of how these may have been affected by lockdowns and
physical isolation resulting from COVID-19. This research aimed to
understand how these psychosocial and behavioral variables may be
associated with psychological distress, and how these associations may
have changed during the COVID-19 pandemic. Methods: A total of 489
children completed a three-wave longitudinal study from January 2020 to
June 2020. The first wave was conducted before the COVID-19 outbreak.
The second wave was conducted during the outbreak. The third wave was
conducted during post-COVID-19 lockdown. Questionnaires measured
psychological distress, PSU, PSMU, and PWS. Results: PSU, PSMU, PWS and
psychological distress were all significantly associated with each
other. PSU was significantly higher during outbreak. PWS was
significantly higher before outbreak. We found an increased association
between PSMU and PWS across three waves in all three models. The
association between PSU and depression/anxiety decreased across three
waves; however, association between PSMU and depression/anxiety
increased across three waves. Conclusions: COVID-19 initiated school
suspension and associated lockdowns appear to have exacerbated PSU and
depression among children. However, PWS was reduced during this period.
Children should use smartphones and social media safely and cautiously,
and be aware of the potential exposure to weight stigmatization.
Copyright © 2021 Fung, Siu, Potenza, O’Brien, Latner, Chen, Chen and
Lin. \| \|Future oncology (London, England) \|Aim: To assess the
perception of telehealth visits among a multiracial cancer population
during the coronavirus disease 2019 pandemic. Methods: This
cross-sectional study was conducted at outpatient cancer clinics in
Hawaii between March and August 2020. Patients were invited to
participate in the survey either by phone or email. Results: Of the 212
survey respondents, 61.3% were Asian, 23.6% were White and 15.1% were
Native Hawaiians or Pacific Islanders. Asians, Native Hawaiians and
Pacific Islanders were less likely to desire future telehealth visits
compared with Whites. Predictors with regard to preferring future
telehealth visits included lower income and hematopoietic cancers.
Conclusion: The authors found racial differences in preference for
telehealth. Future studies aimed at overcoming these racial disparities
are needed to provide equitable oncology care. \| \|Journal of
occupational and environmental medicine \|To investigate associations
between adverse changes in employment status and physical and mental
health among US adults (aged 18 years or older) during the COVID-19
pandemic. Data from participants (N = 2565) of a national Internet panel
(June 2020) were assessed using path analyses to test associations
between changes in self-reported employment status and hours worked and
physical and mental health outcomes. Respondents who lost a job after
March 1, 2020 (vs those who did not) reported more than twice the number
of mentally unhealthy days. Females and those lacking social support had
significantly worse physical and mental health outcomes. Participants in
the lowest, pre-pandemic household income groups reported experiencing
worse mental health. Results demonstrate the importance of providing
economic and social support services to US adults experiencing poor
mental and physical health during the COVID-19 pandemic. Copyright ©
2021 American College of Occupational and Environmental Medicine. \|
\|International journal of environmental research and public health
\|Studies documenting coronavirus disease 2019 (COVID-19) racial/ethnic
disparities in the United States were limited to data from the initial
few months of the pandemic, did not account for changes over time, and
focused primarily on Black and Hispanic minority groups. To fill these
gaps, we examined time trends in racial/ethnic disparities in COVID-19
infection and mortality. We used the Veteran Health Administration’s
(VHA) national database of veteran COVID-19 infections over three time
periods: 3/1/2020-5/31/2020 (spring); 6/1/2020-8/31/2020 (summer); and
9/1/2020-11/25/2020 (fall). We calculated COVID-19 infection and
mortality predicted probabilities from logistic regression models that
included time period-by-race/ethnicity interaction terms, and controlled
for age, gender, and prior diagnosis of CDC risk factors. Racial/ethnic
groups at higher risk for COVID-19 infection and mortality changed over
time. American Indian/Alaskan Natives (AI/AN), Blacks, Hispanics, and
Native Hawaiians/Other Pacific Islanders experienced higher COVID-19
infections compared to Whites during the summertime. There were
mortality disparities for Blacks in springtime, and AI/ANs, Asians, and
Hispanics in summertime. Policy makers should consider the dynamic
nature of racial/ethnic disparities as the pandemic evolves, and
potential effects of risk mitigation and other (e.g., economic) policies
on these disparities. Researchers should consider how trends in
disparities change over time in other samples. \| \|Nature
communications \|NA \| \|Hawai’i journal of health & social welfare \|NA
\| \|BMJ case reports \|A novel coronaravirus, identified as SARS-CoV-2,
spread throughout the world in 2020. The COVID-19 pandemic has led to
many discoveries and clinical manifestations. A young patient is
presented with new, self-resolving neutropenia presenting weeks after a
prolonged hospital stay for COVID-19 pneumonia. Workup included analysis
for underlying infection, nutritional abnormalities, malignancy,
medication and toxin exposure, all of which were negative. From 2020 to
the present, few reports have described neutropenia associated with a
recent COVID-19 infection. In particular, no reports have described a
delayed presentation of neutropenia. The authors would like to propose
that the significant inflammatory response associated with COVID-19 is
likely what led to this patient’s postviral neutropenia. Furthermore, in
young healthy patients, bone marrow biopsy may be deferred and a
watchful-waiting approach may be taken to assess for neutropenia
resolution. © BMJ Publishing Group Limited 2021. No commercial re-use.
See rights and permissions. Published by BMJ. \| \|The Journal of school
health <ISOAbbreviation>J Sch Health</ISOAbbreviation> </Journal>
<ArticleTitle>Five Years and Moving Forward: A Successful Joint
Academic-Practice Public Partnership to Improve the Health of Hawaii’s
Schoolchildren.</ArticleTitle> <Pagination> <StartPage>584</StartPage>
<EndPage>591</EndPage> <MedlinePgn>584-591</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1111/josh.13034</ELocationID>
<Abstract> <AbstractText Label="BACKGROUND">In 2014, the Hawaii
Department of Education (DOE), the only statewide school system in the
United States, predominately enrolled children (keiki) from underserved
communities and lacked school nurses or a school health program. Chronic
absenteeism due to health concerns was identified as a barrier to
academic success.</AbstractText> <AbstractText Label="METHODS">The DOE
and a public university created Hawaii Keiki: Healthy and Ready to Learn
(HK), a program to provide school-based services for 170 Title 1 schools
in urban and rural settings and build momentum for statewide collective
action. HK has maintained support from public and private entities to
address student health.</AbstractText>
<AbstractText Label="RESULTS">This paper describes 5 years of program
development, implementation, and continuing challenges. Most recently in
2020-2021, HK pivoted in the face of school campus closings due to
COVID-19 with strategic plans, including telehealth, to move forward in
this changed school environment.</AbstractText>
<AbstractText Label="CONCLUSIONS">The HK program has increased awareness
of students’ needs and is addressing the imperative to build health
services within public schools. The multipronged approach of building
awareness of need, providing direct services, educating future care
providers, and supporting sound policy development, has an impact that
goes beyond any one individual area.</AbstractText>
<CopyrightInformation>© 2021 American School Health
Association.</CopyrightInformation> </Abstract>
<AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Davis</LastName> <ForeName>Katherine Finn</ForeName>
<Initials>KF</Initials>
<Identifier Source="ORCID">0000-0002-9203-4990</Identifier>
<AffiliationInfo> <Affiliation>School of Nursing and Dental Hygiene,
University of Hawaii at Manoa, 2528 McCarthy Mall, Webster Hall 410,
Honolulu, HI, 96822, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Loos</LastName> <ForeName>Joanne
R</ForeName> <Initials>JR</Initials> <AffiliationInfo>
<Affiliation>School of Nursing and Dental Hygiene, University of Hawaii
at Manoa, 2528 McCarthy Mall, Webster Hall 440, Honolulu, HI, 96822,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Boland</LastName> <ForeName>Mary G</ForeName>
<Initials>MG</Initials> <AffiliationInfo> <Affiliation>School of Nursing
and Dental Hygiene, University of Hawaii at Manoa, 2528 McCarthy Mall,
Webster Hall 402, Honolulu, HI, 96822, USA.</Affiliation>
</AffiliationInfo> </Author> </AuthorList> <Language>eng</Language>
<PublicationTypeList> <PublicationType UI="D016428">Journal
Article</PublicationType> <PublicationType UI="D013485">Research
Support, Non-U.S. Gov’t</PublicationType> </PublicationTypeList>
<ArticleDate DateType="Electronic"> <Year>2021</Year> <Month>05</Month>
<Day>10</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>United States</Country>
      <MedlineTA>J Sch Health</MedlineTA>
      <NlmUniqueID>0376370</NlmUniqueID>
      <ISSNLinking>0022-4391</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000293" MajorTopicYN="N">Adolescent</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="N">COVID-19</DescriptorName>
        <QualifierName UI="Q000517" MajorTopicYN="N">prevention &amp; control</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002648" MajorTopicYN="N">Child</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000067576" MajorTopicYN="N">Child Health</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="Y">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002669" MajorTopicYN="N">Child Welfare</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="Y">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D019058" MajorTopicYN="N">Community Networks</DescriptorName>
        <QualifierName UI="Q000458" MajorTopicYN="Y">organization &amp; administration</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D003299" MajorTopicYN="N">Cooperative Behavior</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006254" MajorTopicYN="N" Type="Geographic">Hawaii</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006293" MajorTopicYN="N">Health Promotion</DescriptorName>
        <QualifierName UI="Q000458" MajorTopicYN="Y">organization &amp; administration</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D015397" MajorTopicYN="N">Program Evaluation</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D012572" MajorTopicYN="N">School Health Services</DescriptorName>
        <QualifierName UI="Q000458" MajorTopicYN="Y">organization &amp; administration</QualifierName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">child and adolescent health</Keyword>
      <Keyword MajorTopicYN="N">chronic absenteeism</Keyword>
      <Keyword MajorTopicYN="N">health data</Keyword>
      <Keyword MajorTopicYN="N">school health program implementation</Keyword>
      <Keyword MajorTopicYN="N">school health services</Keyword>
      <Keyword MajorTopicYN="N">school nursing services</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="revised"> <Year>2020</Year> <Month>11</Month>
<Day>23</Day> </PubMedPubDate> <PubMedPubDate PubStatus="received">
<Year>2020</Year> <Month>6</Month> <Day>16</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="accepted"> <Year>2020</Year> <Month>11</Month>
<Day>24</Day> </PubMedPubDate> <PubMedPubDate PubStatus="pubmed">
<Year>2021</Year> <Month>5</Month> <Day>12</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="medline">
<Year>2021</Year> <Month>6</Month> <Day>22</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="entrez">
<Year>2021</Year> <Month>5</Month> <Day>11</Day> <Hour>6</Hour>
<Minute>45</Minute> </PubMedPubDate> </History>
<PublicationStatus>ppublish</PublicationStatus> <ArticleIdList>
<ArticleId IdType="pubmed">33973241</ArticleId>
<ArticleId IdType="doi">10.1111/josh.13034</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|In 2014, the Hawaii Department of Education
(DOE), the only statewide school system in the United States,
predominately enrolled children (keiki) from underserved communities and
lacked school nurses or a school health program. Chronic absenteeism due
to health concerns was identified as a barrier to academic success. The
DOE and a public university created Hawaii Keiki: Healthy and Ready to
Learn (HK), a program to provide school-based services for 170 Title 1
schools in urban and rural settings and build momentum for statewide
collective action. HK has maintained support from public and private
entities to address student health. This paper describes 5 years of
program development, implementation, and continuing challenges. Most
recently in 2020-2021, HK pivoted in the face of school campus closings
due to COVID-19 with strategic plans, including telehealth, to move
forward in this changed school environment. The HK program has increased
awareness of students’ needs and is addressing the imperative to build
health services within public schools. The multipronged approach of
building awareness of need, providing direct services, educating future
care providers, and supporting sound policy development, has an impact
that goes beyond any one individual area. © 2021 American School Health
Association. \| \|Advances in colloid and interface science \|Severe
acute respiratory syndrome coronavirus 2 (SARS-CoV-2), the virus
responsible for the novel coronavirus disease 2019 (COVID-19), has
caused a global pandemic on a scale not seen for over a century.
Increasing evidence suggests that respiratory droplets and aerosols are
likely the most common route of transmission for SARS-CoV-2. Since the
virus can be spread by presymptomatic and asymptomatic individuals,
universal face masking has been recommended as a straightforward and
low-cost strategy to mitigate virus transmission. Numerous governments
and public health agencies around the world have advocated for or
mandated the wearing of masks in public settings, especially in
situations where social distancing is not possible. However, the
efficacy of wearing a mask remains controversial. This interdisciplinary
review summarizes the current, state-of-the-art understanding of mask
usage against COVID-19. It covers three main aspects of mask usage amid
the pandemic: quality standards for various face masks and their
fundamental filtration mechanisms, empirical methods for quantitatively
determining mask integrity and particle filtration efficiency, and
decontamination methods that allow for the reuse of traditionally
disposable N95 and surgical masks. The focus is given to the fundamental
physicochemical and engineering sciences behind each aspect covered in
this review, providing novel insights into the current understanding of
mask usage to curb COVID-19 spread. Copyright © 2021 Elsevier B.V. All
rights reserved. \| \|The Journal of surgical research \|NA \|
\|Asia-Pacific journal of public health \|NA \| \|Emerging microbes &
infections \|Neutralizing antibodies to SARS-CoV-2 have been shown to
correlate with protection in animals and humans, disease severity,
survival, and vaccine efficacy. With the ongoing large-scale vaccination
in different countries and continuous surge of new variants of global
concerns, a convenient, cost-effective and high-throughput
neutralization test is urgently needed. Conventional SARS-CoV-2
neutralization test is tedious, time-consuming and requires a biosafety
level 3 laboratory. Despite recent reports of neutralizations using
different pseudoviruses with a luciferase or green fluorescent protein
reporter, the laborious steps, inter-assay variability or high
background limit their high-throughput potential. In this study we
generated lentivirus-based pseudoviruses containing a monomeric infrared
fluorescent protein reporter to develop neutralization assays. Similar
tropism, infection kinetics and mechanism of entry through
receptor-mediated endocytosis were found in the three pseudoviruses
generated. Compared with pseudovirus D614, pseudovirus with D614G
mutation had decreased shedding and higher density of S1 protein present
on particles. The 50% neutralization titers to pseudoviruses D614 or
D614G correlated with the plaque reduction neutralization titers to live
SARS-CoV-2. The turn-around time of 48-72 h, minimal autofluorescence,
one-step image quantification, expandable to 384-well, sequential
readouts and dual quantifications by flow cytometry support its
high-throughput and versatile applications at a non-reference and
biosafety level 2 laboratory, in particular for assessing the
neutralization sensitivity of new variants by sera from natural
infection or different vaccinations during our fight against the
pandemic. \| \|Patient safety in surgery \|At the time of writing of
this article, there have been over 110 million cases and 2.4 million
deaths worldwide since the start of the Coronavirus Disease 2019
(COVID-19) pandemic, postponing millions of non-urgent surgeries.
Existing literature explores the complexities of rationing medical care.
However, implications of non-urgent surgery postponement during the
COVID-19 pandemic have not yet been analyzed within the context of the
four pillars of medical ethics. The objective of this review is to
discuss the ethics of elective surgery cancellation during the COVID-19
pandemic in relation to beneficence, non-maleficence, justice, and
autonomy. This review hypothesizes that a more equitable decision-making
algorithm can be formulated by analyzing the ethical dilemmas of
elective surgical care during the pandemic through the lens of these
four pillars. This paper’s analysis shows that non-urgent surgeries
treat conditions that can become urgent if left untreated. Postponement
of these surgeries can cause cumulative harm downstream. An improved
algorithm can address these issues of beneficence by weighing local
pandemic stressors within predictive algorithms to appropriately
increase surgeries. Additionally, the potential harms of performing
non-urgent surgeries extend beyond the patient. Non-maleficence is
maintained through using enhanced screening protocols and modifying
surgical techniques to reduce risks to patients and clinicians. This
model proposes a system to transfer patients from areas of high to low
burden, addressing the challenge of justice by considering facility
burden rather than value judgments concerning the nature of a particular
surgery, such as cosmetic surgeries. Autonomy can be respected by giving
patients the option to cancel or postpone non-urgent surgeries. However,
in the context of limited resources in a global pandemic, autonomy is
not absolute. Non-urgent surgeries can ethically be postponed in
opposition to the patient’s preference. The proposed algorithm attempts
to uphold the four principles of medical ethics in rationing non-urgent
surgical care by building upon existing decision models, using
additional measures of resource burden and surgical safety to increase
health care access and decrease long-term harm as much as possible. The
next global health crisis will undoubtedly present its own unique
challenges. This model may serve as a comprehensive starting point in
determining future guidelines for non-urgent surgical care. \|
\|Emerging infectious diseases \|We examined disparities in cumulative
incidence of severe acute respiratory syndrome coronavirus 2 by
race/ethnicity, age, and sex in the United States during January
1-October 1, 2020. Hispanic/Latino and non-Hispanic Black, American
Indian/Alaskan Native, and Native Hawaiian/other Pacific Islander
persons had a substantially higher incidence of infection than
non-Hispanic White persons. \| \|Sustainability science \|Because of
ethnic and cultural violence in Myanmar, approximately a million
Rohingya fled to neighboring Bangladesh starting from August 2017, in
what the UN has called a “textbook example of ethnic cleansing”. Those
arriving in Bangladesh were able to escape decade-long ethnic violence
in Myanmar, but the Rohingya’s immediate destination, Cox’s Bazar
district is one of the most climate-vulnerable and disaster-prone areas
in Bangladesh. Currently, they have been subjected to extreme rainfalls,
landslides, and flashfloods. With the COVID-19 pandemic, they continue
to face fear and further marginalization in resource-constrained
Bangladesh, as well as increased vulnerability due to tropical cyclones,
flashfloods, and landslides. The Rohingya in southeast Bangladesh are
now at the epicenter of a humanitarian and sustainability crisis.
However, their situation is not entirely unique. Millions of displaced,
stateless or refugees around the world are facing multi-dimensional
crises in various complex geopolitical, and climatic situations. Using
the theoretical lens of political ecology and critical development
studies, this paper analyzes the sustainability-peace nexus for millions
of Rohingya in Myanmar and in Bangladesh. This paper is based on
information from various sources, including three ethnographic field
visits in recent years, which helped to get local insights into the
current sustainability challenges in this humanitarian context. The core
arguments of this paper suggest that sustainability-peace nexus will
especially be compromised in climate-vulnerable resource-constrained
conditions. To overcome this challenge, decolonizing Rohingya solutions
would be critical, by engaging the Rohingya in the process of
development and meaningful change, which can affect their lives,
livelihoods, and wellbeing. Even though this paper has a specific
geographical focus, the insights are relevant in parts of the world
facing similar social, economic, political, and environmental
challenges. © The Author(s), under exclusive licence to Springer Japan
KK, part of Springer Nature 2021. \| \|Annals of epidemiology \|There is
a growing concern about the COVID-19 epidemic intensifying in rural
areas in the United States (U.S.). In this study, we described the
dynamics of COVID-19 cases and deaths in rural and urban counties in the
U.S. Using data from April 1 to November 12, 2020, from Johns Hopkins
University, we estimated COVID-19 incidence and mortality rates and
conducted comparisons between urban and rural areas in three time
periods at the national level, and in states with higher and lower
COVID-19 incidence rates. Results at the national level showed greater
COVID-19 incidence rates in urban compared to rural counties in the
Northeast and Mid-Atlantic regions of the U.S. at the beginning of the
epidemic. However, the intensity of the epidemic has shifted to a rapid
surge in rural areas. In particular, high incidence states located in
the Mid-west of the country had more than 3,400 COVID-19 cases per
100,000 people compared to 1,284 cases per 100,000 people in urban
counties nationwide during the third period (August 30 to November 12).
Overall, the current epicenter of the epidemic is located in states with
higher infection rates and mortality in rural areas. Infection
prevention and control efforts including healthcare capacity should be
scaled up in these vulnerable rural areas. Copyright © 2021. Published
by Elsevier Inc. \| \|Hawai’i journal of health & social welfare \|NA \|
\|Hawai’i journal of health & social welfare \|NA \| \|Nature
communications \|Substantial COVID-19 research investment has been
allocated to randomized clinical trials (RCTs) on
hydroxychloroquine/chloroquine, which currently face recruitment
challenges or early discontinuation. We aim to estimate the effects of
hydroxychloroquine and chloroquine on survival in COVID-19 from all
currently available RCT evidence, published and unpublished. We present
a rapid meta-analysis of ongoing, completed, or discontinued RCTs on
hydroxychloroquine or chloroquine treatment for any COVID-19 patients
(protocol: <https://osf.io/QESV4/> ). We systematically identified
unpublished RCTs (ClinicalTrials.gov, WHO International Clinical Trials
Registry Platform, Cochrane COVID-registry up to June 11, 2020), and
published RCTs (PubMed, medRxiv and bioRxiv up to October 16, 2020).
All-cause mortality has been extracted (publications/preprints) or
requested from investigators and combined in random-effects
meta-analyses, calculating odds ratios (ORs) with 95% confidence
intervals (CIs), separately for hydroxychloroquine and chloroquine.
Prespecified subgroup analyses include patient setting, diagnostic
confirmation, control type, and publication status. Sixty-three trials
were potentially eligible. We included 14 unpublished trials (1308
patients) and 14 publications/preprints (9011 patients). Results for
hydroxychloroquine are dominated by RECOVERY and WHO SOLIDARITY, two
highly pragmatic trials, which employed relatively high doses and
included 4716 and 1853 patients, respectively (67% of the total sample
size). The combined OR on all-cause mortality for hydroxychloroquine is
1.11 (95% CI: 1.02, 1.20; I² = 0%; 26 trials; 10,012 patients) and for
chloroquine 1.77 (95%CI: 0.15, 21.13, I² = 0%; 4 trials; 307 patients).
We identified no subgroup effects. We found that treatment with
hydroxychloroquine is associated with increased mortality in COVID-19
patients, and there is no benefit of chloroquine. Findings have unclear
generalizability to outpatients, children, pregnant women, and people
with comorbidities. \| \|American journal of public health \|The complex
and evolving picture of COVID-19-related mortality highlights the need
for data to guide the response. Yet many countries are struggling to
maintain their data systems, including the civil registration system,
which is the foundation for detailed and continuously available
mortality statistics. We conducted a search of country and development
agency Web sites and partner and media reports describing disruptions to
the civil registration of births and deaths associated with COVID-19
related restrictions.We found considerable intercountry variation and
grouped countries according to the level of disruption to birth and
particularly death registration. Only a minority of the 66 countries
were able to maintain service continuity during the COVID-19
restrictions. In the majority, a combination of legal and operational
challenges resulted in declines in birth and death registration. Few
countries established business continuity plans or developed strategies
to deal with the backlog when restrictions are lifted.Civil registration
systems and the vital statistics they generate must be strengthened as
essential services during health emergencies and as core components of
the response to COVID-19. \| \|medRxiv : the preprint server for health
sciences \|The impact of COVID-19 disease on health and economy has been
global, and the magnitude of devastation is unparalleled in modern
history. Any potential course of action to manage this complex disease
requires the systematic and efficient analysis of data that can
delineate the underlying pathogenesis. We have developed a mathematical
model of disease progression to predict the clinical outcome, utilizing
a set of causal factors known to contribute to COVID-19 pathology such
as age, comorbidities, and certain viral and immunological parameters.
Viral load and selected indicators of a dysfunctional immune response,
such as cytokines IL-6 and IFNα, which contribute to the cytokine storm
and fever, parameters of inflammation d-dimer and ferritin, aberrations
in lymphocyte number, lymphopenia, and neutralizing antibodies were
included for the analysis. The model provides a framework to unravel the
multi-factorial complexities of the immune response manifested in
SARS-CoV-2 infected individuals. Further, this model can be valuable to
predict clinical outcome at an individual level, and to develop
strategies for allocating appropriate resources to mitigate severe cases
at a population level. \| \|Water research \|Wastewater is a pooled
sampling instrument that may provide rapid and even early disease
signals in the surveillance of COVID-19 disease at the community level,
yet the fine-scale temporal dynamics of SARS-CoV-2 RNA in wastewater
remains poorly understood. This study tracked the daily dynamics of
SARS-CoV-2 RNA in the wastewater from two wastewater treatment plants
(WWTPs) in Honolulu during a rapidly expanding COVID-19 outbreak and a
responding four-week lockdown that resulted in a rapid decrease of daily
clinical COVID-19 new cases. The wastewater SARS-CoV-2 RNA concentration
from both WWTPs, as measured by three quantification assays (N1, N2, and
E), exhibited both significant inter-day fluctuations (101.2-105.1 gene
copies or GC/L in wastewater liquid fractions, or 101.4-106.2 GC/g in
solid fractions) and an overall downward trend over the lockdown period.
Strong and significant correlation was observed in measured SARS-CoV-2
RNA concentrations between the solid and liquid wastewater fractions,
with the solid fraction containing majority (82.5%-92.5%) of the
SARS-CoV-2 RNA mass and the solid-liquid SARS-CoV-2 RNA concentration
ratios ranging from 103.6 to 104.3 mL/g. The measured wastewater
SARS-CoV-2 RNA concentration was normalized by three endogenous fecal
RNA viruses (F+ RNA coliphages Group II and III, and pepper mild mottle
virus) to account for variations that may occur during the multi-step
wastewater processing and molecular quantification, and the normalized
abundance also exhibited similar daily fluctuations and overall downward
trend over the sampling period. Copyright © 2021 Elsevier Ltd. All
rights reserved. \| \|Research square \|Using genomics, bioinformatics
and statistics, herein we demonstrate the effect of statewide and
nationwide quarantine on the introduction of SARS-CoV-2 variants of
concern (VOC) in Hawai’i. To define the origins of introduced VOC, we
analyzed 260 VOC sequences from Hawai’i, and 301,646 VOC sequences
worldwide, deposited in the GenBank and global initiative on sharing all
influenza data (GISAID), and constructed phylogenetic trees. The trees
define the most recent common ancestor as the origin. Further, the
multiple sequence alignment used to generate the phylogenetic trees
identified the consensus single nucleotide polymorphisms in the VOC
genomes. These consensus sequences allow for VOC comparison and
identification of mutations of interest in relation to viral immune
evasion and host immune activation. Of note is the P71L substitution
within the E protein, the protein sensed by TLR2 to produce cytokines,
found in the B.1.351 VOC may diminish the efficacy of some vaccines.
Based on the phylogenetic trees, the B.1.1.7, B.1.351, B.1.427, and
B.1.429 VOC have been introduced in Hawai’i multiple times since
December 2020 from several definable geographic regions. From the first
worldwide report of VOC in GenBank and GISAID, to the first arrival of
VOC in Hawai’i, averages 320 days with quarantine, and 132 days without
quarantine. As such, the effect of quarantine is shown to significantly
affect the time to arrival of VOC in Hawai’i, both during and following
quarantine. Further, the collective 2020 quarantine of 43-states in the
United States demonstrates a profound impact in delaying the arrival of
VOC in states that did not practice quarantine, such as Utah. Our data
demonstrates that at least 76% of all definable SARS-CoV-2 VOC have
entered Hawai’i from California, with the B.1.351 variant in Hawai’i
originating exclusively from the United Kingdom. These data provide a
foundation for policy-makers and public-health officials to apply
precision public health genomics to real-world policies such as
mandatory screening and quarantine. \| \|Contraception \|To demonstrate
the effectiveness of medication abortion with the implementation of
telemedicine and a no-test protocol in response to the COVID-19
pandemic. This is a retrospective cohort study of patients who had a
medication abortion up to 77 days gestation at the University of Hawai’i
between April and November 2020. Patients had the option of traditional
in clinic care or telemedicine with either in clinic pickup or mailing
of medications. During this time, a no-test protocol for medication
abortion without prior labs or ultrasound was in place for eligible
patients. The primary outcome was the rate of successful medication
abortion without surgical intervention. Secondary outcomes included
abortion-related complications. A total of 334 patients were dispensed
mifepristone and misoprostol, 149 (44.6%) with telemedicine with
in-person pickup of medications, 75 (22.5%) via telemedicine with
medications mailed, and 110 (32.9%) via traditional in person visits.
The overall rate of complete medication abortion without surgical
intervention was 95.8%, with success rates of 96.8, 97.1, and 93.6% for
the clinic pickup, mail, and clinic visit groups, respectively. Success
for those without an ultrasound performed prior to the procedure was
96.6%, compared to 95.5% for those with ultrasound. We obtained
follow-up data for 87.8% of participants. Medication abortion was safe
and effective while offering multiple modes of care delivery including
telemedicine visits without an ultrasound performed prior to dispensing
medications. Incorporating telemedicine and a no-test protocol for
medication abortion is safe and has the potential to expand access to
abortion care. All care models had low rates of adverse events, which
contradicts the idea that the Risk Evaluation and Mitigation
Strategyincreases the safety of medication abortion. Copyright © 2021
Elsevier Inc. All rights reserved. \| \|Contraception \|To present
updated evidence on the safety, efficacy and acceptability of a
direct-to-patient telemedicine abortion service and describe how the
service functioned during the COVID-19 pandemic. We offered the study at
10 sites that provided the service in 13 states and Washington DC.
Interested individuals obtained any needed preabortion tests locally and
had a videoconference with a study clinician. Sites sent study packages
containing mifepristone and misoprostol by mail and had remote follow-up
consultations within one month by telephone (or by online survey, if the
participant could not be reached) to evaluate abortion completeness. The
analysis was descriptive. We mailed 1390 packages between May 2016 and
September 2020. Of the 83% (1157/1390) of abortions for which we
obtained outcome information, 95% (1103/1157) were completed without a
procedure. Participants made 70 unplanned visits to emergency rooms or
urgent care centers for reasons related to the abortion (6%), and 10
serious adverse events occurred, including 5 transfusions (0.4%).
Enrollment increased substantially with the onset of COVID-19. Although
a screening ultrasound was required, sites determined in 52% (346/669)
of abortions that occurred during COVID that those participants should
not get the test to protect their health. Use of urine pregnancy test to
confirm abortion completion increased from 67% (144/214) in the 6 months
prior to COVID to 90% (602/669) in the 6 months during COVID. Nearly all
satisfaction questionnaires (99%, 1013/1022) recorded that participants
were satisfied with the service. This direct-to-patient telemedicine
service was safe, effective, and acceptable, and supports the claim that
there is no medical reason for mifepristone to be dispensed in clinics
as required by the Food and Drug Administration. In some cases,
participants did not need to visit any facilities to obtain the service,
which was critical to protecting patient safety during the COVID-19
pandemic. Medical abortion using telemedicine and mail is effective and
can be safely provided without a pretreatment ultrasound. This method of
service delivery has the potential to greatly improve access to abortion
care in the United States. Copyright © 2021 Elsevier Inc. All rights
reserved. \| \|Surgery \|NA \| \|Global health promotion \|The current
COVID-19 pandemic has exposed missing links between health promotion and
national/global health emergency policies. In response, health promotion
initiatives were urgently developed and applied around the world. A
selection of case studies from five countries, based on the
Socio-Ecological Model of Health Promotion, exemplify ‘real-world’
action and challenges for health promotion intervention, research, and
policy during the COVID-19 pandemic. Interventions range from a focus on
individuals/families, organizations, communities and in healthcare,
public health, education and media systems, health-promoting settings,
and policy. Lessons learned highlight the need for emphasizing equity,
trust, systems approach, and sustained action in future health promotion
preparedness strategies. Challenges and opportunities are highlighted
regarding the need for rapid response, clear communication based on
health literacy, and collaboration across countries, disciplines, and
health and education systems for meaningful solutions to global health
crises. \| \|Journal of general internal medicine \|NA \| \|Journal of
addiction medicine \|The outbreak of the coronavirus disease 2019
(COVID-19) pandemic has generated negative effects on psychological
well-being worldwide, including in schoolchildren. Government
requirements to stay at home and avoid social and school settings may
impact psychological well-being by modifying various behaviors such as
problematic phone and Internet use, yet there is a paucity of research
on this issue. This study examined whether the COVID-19 outbreak may
have impacted problematic smartphone use (PSU), problematic gaming (PG),
and psychological distress, specifically the pattern of relationships
between PSU, PG, and psychological distress in schoolchildren.
Longitudinal data on psychological distress, PSU, and PG were collected
from 575 children in primary schools in 3 waves: Waves 1 and 2 were
conducted before the COVID-19 outbreak and Wave 3 during the outbreak.
Cross-lagged panel models were used to examine relationships between
factors across the 3 waves. Cross-lagged models found that higher levels
of PSU were not significantly related prospectively to greater
psychological distress before the COVID-19 outbreak, but this
prospective relationship became significant during the COVID-19
outbreak. Whereas PG was associated prospectively with psychological
distress before the COVID-19 outbreak (ie, between Waves 1 and 2), this
association became nonsignificant during the COVID-19 lockdown (ie,
between Waves 2 and 3). The COVID-19 outbreak has seemed to change
prospective relationships between PSU and psychological distress and PG
and psychological distress in schoolchildren. Future research should
examine whether restrictions on or information provided to
schoolchildren may exacerbate PSUs effects on psychological distress.
Copyright © 2021 American Society of Addiction Medicine. \|
\|Translational behavioral medicine \|Widespread uptake of the COVID-19
vaccine is critical to halt the pandemic. At present, little is known
about factors that will affect vaccine uptake, especially among diverse
racial/ethnic communities that have experienced the highest burden of
COVID. We administered an online survey to a Qualtrics respondent panel
of women ages 27-45 years (N = 396) to assess vaccine intentions and
attitudes, and trusted vaccine information sources. 56.8% intended to be
vaccinated and 25.5% were unsure. In bivariate analyses, a greater
percentage of non-Latina White (NLW) and Chinese women reported that
they would be vaccinated, compared with Latina and non-Latina Black
(NLB) women (p \< 0.001). Those who were uninsured, unemployed and those
with lower incomes were less likely to say that they would be
vaccinated. In analyses stratified by race/ethnicity, NLB women remained
significantly less likely to report that they would be vaccinated
compared with NLW women (adjusted odds ratio: 0.47; 95% confidence
interval: 0.23, 0.94), controlling for age, marital status, income,
education, employment, and insurance status. When analyses were
additionally controlled for beliefs in vaccine safety and efficacy,
racial/ethnic differences were no longer significant (adjusted odds
ratio: 0.64; 95% confidence interval: 0.31, 1.34). Given that NLB women
were less likely to report the intention to be vaccinated, targeted
efforts will be needed to promote vaccine uptake. It will be critical to
emphasize that the vaccine is safe and effective; this message may be
best delivered by trusted community members. Published by Oxford
University Press on behalf of the Society of Behavioral Medicine 2021.
\| \|Annals of oncology : official journal of the European Society for
Medical Oncology \|Patients with cancer may be at high risk of adverse
outcomes from severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) infection. We analyzed a cohort of patients with cancer and
coronavirus 2019 (COVID-19) reported to the COVID-19 and Cancer
Consortium (CCC19) to identify prognostic clinical factors, including
laboratory measurements and anticancer therapies. Patients with active
or historical cancer and a laboratory-confirmed SARS-CoV-2 diagnosis
recorded between 17 March and 18 November 2020 were included. The
primary outcome was COVID-19 severity measured on an ordinal scale
(uncomplicated, hospitalized, admitted to intensive care unit,
mechanically ventilated, died within 30 days). Multivariable regression
models included demographics, cancer status, anticancer therapy and
timing, COVID-19-directed therapies, and laboratory measurements (among
hospitalized patients). A total of 4966 patients were included (median
age 66 years, 51% female, 50% non-Hispanic white); 2872 (58%) were
hospitalized and 695 (14%) died; 61% had cancer that was present,
diagnosed, or treated within the year prior to COVID-19 diagnosis. Older
age, male sex, obesity, cardiovascular and pulmonary comorbidities,
renal disease, diabetes mellitus, non-Hispanic black race, Hispanic
ethnicity, worse Eastern Cooperative Oncology Group performance status,
recent cytotoxic chemotherapy, and hematologic malignancy were
associated with higher COVID-19 severity. Among hospitalized patients,
low or high absolute lymphocyte count; high absolute neutrophil count;
low platelet count; abnormal creatinine; troponin; lactate
dehydrogenase; and C-reactive protein were associated with higher
COVID-19 severity. Patients diagnosed early in the COVID-19 pandemic
(January-April 2020) had worse outcomes than those diagnosed later.
Specific anticancer therapies (e.g. R-CHOP, platinum combined with
etoposide, and DNA methyltransferase inhibitors) were associated with
high 30-day all-cause mortality. Clinical factors (e.g. older age,
hematological malignancy, recent chemotherapy) and laboratory
measurements were associated with poor outcomes among patients with
cancer and COVID-19. Although further studies are needed, caution may be
required in utilizing particular anticancer therapies. NCT04354701.
Copyright © 2021 The Author(s). Published by Elsevier Ltd.. All rights
reserved. \| \|JPEN. Journal of parenteral and enteral nutrition
\|Severe acute respiratory syndrome coronavirus 2 is a respiratory virus
that poses risks to the nutrition status and survival of infected
patients, yet there is paucity of data to inform evidence-based quality
care. We collected data on the nutrition care provided to patients with
coronavirus disease 2019 (COVID-19) by registered dietitian
nutritionists (RDNs). Hospitalized COVID-19 patients (N = 101) in this
cohort were older adults and had elevated body mass index. The most
frequent nutrition problems were inadequate oral intake (46.7%),
inadequate energy intake (18.9%), and malnutrition (18.4%). These
problems were managed predominantly with enteral nutrition, food
supplements, and multivitamin-multimineral supplement therapy. Over 90%
of documented problems required a follow-up. This data set is the first
of its kind to report on the types of nutrition diagnoses and
interventions for COVID-19 cases used by RDNs and highlights the need
for increased and continued nutrition care. © 2021 American Society for
Parenteral and Enteral Nutrition. \| \|Hawai’i journal of health &
social welfare \|The COVID-19 pandemic has ravaged the world, caused
over 1.8 million deaths in its first year, and severely affected the
global economy. Hawai’i has not been spared from the transmission of
SARS-CoV-2 in the local population, including high infection rates in
racial and ethnic minorities. Early in the pandemic, we described in
this journal various technologies used for the detection of SARS-CoV-2.
Herein we characterize a 969-bp SARS-CoV-2 segment of the S gene
downstream of the receptor-binding domain. At the John A. Burns School
of Medicine Biocontainment Facility, RNA was extracted from an
oropharyngeal swab and a nasal swab from 2 patients from Hawai’i who
were infected with SARS-CoV-2 in August 2020. Following PCR, the 2 viral
strains were sequenced using Sanger sequencing, and phylogenetic trees
were generated using MEGAX. Phylogenetic tree results indicate that the
virus has been introduced to Hawai’i from multiple sources. Further, we
decoded 13 single nucleotide polymorphisms across 13 unique SARS-CoV-2
genomes within this region of the S gene, with 1 non-synonymous mutation
(P681H) found in the 2 Hawai’i strains. The P681H mutation has unique
and emerging characteristics with a significant exponential increase in
worldwide frequency when compared to the plateauing of the now universal
D614G mutation. The P681H mutation is also characteristic of the new
SARS-CoV-2 variants from the United Kingdom and Nigeria. Additionally,
several mutations resulting in cysteine residues were detected,
potentially resulting in disruption of the disulfide bridges in and
around the receptor-binding domain. Targeted sequence characterization
is warranted to determine the origin of multiple introductions of
SARS-CoV-2 circulating in Hawai’i. ©Copyright 2021 by University Health
Partners of Hawai‘i (UHP Hawai‘i). \| \|Journal of general and family
medicine \|We welcome their additional suggestion that the government
should publish potential causes for and implications of the additional
outbreak beyond the quarantine to the international scientific community
so that similar outbreaks may be swiftly prevented. However, given the
absence of government-driven publications, we published this report
based on our independent investigation, which may be more reliable
considering the inherently sensitive and political nature of the events.
© 2020 The Authors. Journal of General and Family Medicine published by
John Wiley & Sons Australia, Ltd on behalf of Japan Primary Care
Association. \| \|Health & place \|The global Coronavirus Disease 2019
(COVID-19) pandemic has led to the implementation of social distancing
measures such as work-from-home orders that have drastically changed
people’s travel-related behavior. As countries are easing up these
measures and people are resuming their pre-pandemic activities, the
second wave of COVID-19 is observed in many countries. This study
proposes a Community Activity Score (CAS) based on inter-community
traffic characteristics (in and out of community traffic volume and
travel distance) to capture the current travel-related activity level
compared to the pre-pandemic baseline and study its relationship with
confirmed COVID-19 cases. Fourteen other travel-related factors
belonging to five categories (Social Distancing Index, residents staying
at home, travel frequency and distance, mobility trend, and
out-of-county visitors) and three social distancing measures
(stay-at-home order, face-covering order, and self-quarantine for
out-of-county travels) are also considered to reflect the likelihood of
exposure to the COVID-19. Considering that it usually takes days from
exposure to confirming the infection, the exposure-to-confirm temporal
delay between the time-varying travel-related factors and their impacts
on the number of confirmed COVID-19 cases is considered in this study.
Honolulu County in the State of Hawaii is used as a case study to
evaluate the proposed CAS and other factors on confirmed COVID-19 cases
with various temporal delays at a county-level. Negative Binomial models
were chosen to study the impacts of travel-related factors and social
distancing measures on COVID-19 cases. The case study results show that
CAS and other factors are correlated with COVID-19 spread, and models
that factor in the exposure-to-confirm temporal delay perform better in
forecasting COVID-19 cases later. Policymakers can use the study’s
various findings and insights to evaluate the impacts of social
distancing policies on travel and effectively allocate resources for the
possible increase in confirmed COVID-19 cases. Copyright © 2021 Elsevier
Ltd. All rights reserved. \| \|MMWR. Morbidity and mortality weekly
report \|NA \| \|Preventing chronic disease \|Communication networks
among professionals can be pathways for accelerating the diffusion of
innovations if some local health departments (LHDs) drive the spread of
knowledge. Such a network could prove valuable during public health
emergencies such as the novel coronavirus disease 2019 (COVID-19)
pandemic. Our objective was to determine whether LHDs in the United
States were tied together in an informal network to share information
and advice about innovative community health practices, programs, and
policies. In January and February 2020, we conducted an online survey of
2,303 senior LHD leaders to ask several questions about their sources of
advice. We asked respondents to rank up to 3 other LHDs whose practices
informed their work on new public health programs, evidence-based
practices, and policies intended to improve community health. We used a
social network analysis program to assess answers. A total of 329 LHDs
responded. An emergent network appeared to operate nationally among 740
LHDs. Eleven LHDs were repeatedly nominated by peers as sources of
advice or examples (ie, opinion leaders), and 24 acted as relational
bridges to hold these emergent networks together (ie, boundary
spanners). Although 2 LHDs played both roles, most LHDs we surveyed
performed neither of these roles. Opinion leading and boundary spanning
health departments can be accessed to increase the likelihood of
affecting the rate of interest in and adoption of innovations. Decision
makers involved in disseminating new public health practices, programs,
or policies may find our results useful both for emergencies and for
practice-as-usual. \| \|PloS one \|A newly identified coronavirus,
designated as severe acute respiratory syndrome coronavirus 2 (SARS
CoV-2), has spread rapidly from its epicenter in China to more than 150
countries across six continents. In this study, we have designed three
reverse-transcription loop-mediated isothermal amplification (RT-LAMP)
primer sets to detect the RNA-dependent RNA polymerase (RdRP), Envelope
(E) and Nucleocapsid protein (N) genes of SARS CoV-2. For one tube
reaction, the detection limits for five combination SARS CoV-2 LAMP
primer sets (RdRP/E, RdRP/N, E/N, RdRP/E/N and RdRP/N/Internal control
(actin beta)) were evaluated with a clinical nasopharyngeal swab sample.
Among the five combination, the RdRP/E and RdRP/N/IC multiplex LAMP
assays showed low detection limits. The sensitivity and specificity of
the RT-LAMP assay were evaluated and compared to that of the widely used
Allplex™ 2019-nCoV Assay (Seegene, Inc., Seoul, South Korea) and
PowerChek™ 2019-nCoV Real-time PCR kit (Kogenebiotech, Seoul, South
Korea) for 130 clinical samples from 91 SARS CoV-2 patients and 162 NP
specimens from individuals with (72) and without (90) viral respiratory
infections. The multiplex RdRP (FAM)/N (CY5)/IC (Hex) RT-LAMP assay
showed comparable sensitivities (RdRP: 93.85%, N: 94.62% and RdRP/N:
96.92%) to that of the Allplex™ 2019-nCoV Assay (100%) and superior to
those of PowerChek™ 2019-nCoV Real-time PCR kit (RdRP: 92.31%, E: 93.85%
and RdRP/E: 95.38%). \| \|The Lancet. Microbe \|NA \| \|The Brazilian
journal of infectious diseases : an official publication of the
Brazilian Society of Infectious Diseases \|In the pandemic, rapid and
accurate detection of SARS-CoV-2 is crucial in controlling the outbreak.
Recent studies have shown a high detection rate using saliva/oral fluids
as specimens for laboratory detection of the virus. We intended to
evaluate the test performance of the Xpert Xpress SARS-CoV-2 cartridge
assay in comparison to a conventional qRT-PCR testing, using saliva as
biological specimen. Forty saliva samples from symptomatic participants
were collected. Conventional qRT-PCR was performed for amplification of
E and RdRp genes and the Xpert Xpress SARS-CoV-2 assay amplified E and
N2 genes. In the conventional assay, the median cycle threshold value of
the E gene was 34.9, and of the RdRp gene was 38.3. In the Xpert Xpress
assay, the median cycle threshold value of the E gene was 29.7, and of
the N2 gene was 31.6. These results can allow a broaden use of molecular
tests for management of COVID-19 pandemic, especially in
resources-limited settings. Copyright © 2021 Sociedade Brasileira de
Infectologia. Published by Elsevier España, S.L.U. All rights reserved.
\| \|The American journal of tropical medicine and hygiene \|An outbreak
of SARS-CoV-2 has led to a global pandemic affecting virtually every
country. As of August 31, 2020, globally, there have been approximately
25,500,000 confirmed cases and 850,000 deaths; in the United States (50
states plus District of Columbia), there have been more than 6,000,000
confirmed cases and 183,000 deaths. We propose a Bayesian mixture model
to predict and monitor COVID-19 mortality across the United States. The
model captures skewed unimodal (prolonged recovery) or multimodal
(multiple surges) curves. The results show that across all states, the
first peak dates of mortality varied between April 4, 2020 for Alaska
and June 18, 2020 for Arkansas. As of August 31, 2020, 31 states had a
clear bimodal curve showing a strong second surge. The peak date for a
second surge ranged from July 1, 2020 for Virginia to September 12, 2020
for Hawaii. The first peak for the United States occurred about April
16, 2020-dominated by New York and New Jersey-and a second peak on
August 6, 2020-dominated by California, Texas, and Florida. Reliable
models for predicting the COVID-19 pandemic are essential to informing
resource allocation and intervention strategies. A Bayesian mixture
model was able to more accurately predict the shape of the mortality
curves across the United States than other models, including the timing
of multiple peaks. However, given the dynamic nature of the pandemic, it
is important that the results be updated regularly to identify and
better monitor future waves, and characterize the epidemiology of the
pandemic. \| \|Journal of medical Internet research \|The COVID-19
pandemic has significantly impacted mental health and well-being. Mobile
mental health apps can be scalable and useful tools in large-scale
disaster responses and are particularly promising for reaching
vulnerable populations. COVID Coach is a free, evidence-informed mobile
app designed specifically to provide tools and resources for addressing
COVID-19-related stress. The purpose of this study was to characterize
the overall usage of COVID Coach, explore retention and return usage,
and assess whether the app was reaching individuals who may benefit from
mental health resources. Anonymous usage data collected from COVID Coach
between May 1, 2020, through October 31, 2020, were extracted and
analyzed for this study. The sample included 49,287 unique user codes
and 3,368,931 in-app events. Usage of interactive tools for coping and
stress management comprised the majority of key app events (n=325,691,
70.4%), and the majority of app users tried a tool for managing stress
(n=28,009, 58.8%). COVID Coach was utilized for ≤3 days by 80.9%
(n=34,611) of the sample whose first day of app use occurred within the
6-month observation window. Usage of the key content in COVID Coach
predicted returning to the app for a second day. Among those who tried
at least one coping tool on their first day of app use, 57.2% (n=11,444)
returned for a second visit; whereas only 46.3% (n=10,546) of those who
did not try a tool returned (P\<.001). Symptoms of anxiety, depression,
and posttraumatic stress disorder (PTSD) were prevalent among app users.
For example, among app users who completed an anxiety assessment on
their first day of app use (n=4870, 11.4% of users), 55.1% (n=2680)
reported levels of anxiety that were moderate to severe, and 29.9%
(n=1455) of scores fell into the severe symptom range. On average, those
with moderate levels of depression on their first day of app use
returned to the app for a greater number of days (mean 3.72 days) than
those with minimal symptoms (mean 3.08 days; t1=3.01, P=.003).
Individuals with significant PTSD symptoms on their first day of app use
utilized the app for a significantly greater number of days (mean 3.79
days) than those with fewer symptoms (mean 3.13 days; t1=2.29, P=.02).
As the mental health impacts of the pandemic continue to be widespread
and increasing, digital health resources, such as apps like COVID Coach,
are a scalable way to provide evidence-informed tools and resources.
Future research is needed to better understand for whom and under what
conditions the app is most helpful and how to increase and sustain
engagement. ©Beth K Jaworski, Katherine Taylor, Kelly M Ramsey, Adrienne
Heinz, Sarah Steinmetz, Ian Pagano, Giovanni Moraja, Jason E Owen.
Originally published in the Journal of Medical Internet Research
(<http://www.jmir.org>), 01.03.2021. \| \|Annals of internal medicine
\|NA \| \|International journal of obesity (2005) \|The novel
coronavirus disease 2019 (COVID-19) pandemic, and its resulting social
policy changes may result in psychological distress among schoolchildren
with overweight. This study thus aimed to (1) compare psychological
distress (including fear of COVID-19 infection, stress, anxiety, and
depression), perceived weight stigma, and problematic internet-related
behaviors between schoolchildren with and without overweight; (2) assess
whether perceived weight stigma and problematic internet-related
behaviors explained psychological distress. Schoolchildren (n = 1357;
mean age = 10.7 years) with overweight (n = 236) and without overweight
(n = 1121) completed an online survey assessing their fear of COVID-19
infection, stress, anxiety, depression, perceived weight stigma,
problematic smartphone application use, problematic social media use,
and problematic gaming. Schoolchildren with overweight had significantly
higher levels of COVID-19 infection fear, stress, depression, perceived
weight stigma, and problematic social media use than those without
overweight. Regression models showed that perceived weight stigma and
problematic internet-related behaviors were significant predictors of
psychological distress among schoolchildren with overweight. Strategies
to manage perceived weight stigma and problematic internet-related
behaviors may have a positive influence on mental health among
schoolchildren with overweight under health-threatening circumstances,
such as the current COVID-19 pandemic. \| \|Global health promotion
\|Shortly after a healthy default beverage (HDB) law took effect in
Hawai’i, requiring restaurants that serve children’s meals to offer
healthy beverages with the meals, the COVID-19 pandemic struck. Efforts
to contain the virus resulted in changes to restaurants’ operations and
disrupted HDB implementation efforts. Economic repercussions from
containment efforts have exacerbated food insecurity, limited access to
healthy foods, and created obstacles to chronic disease management.
Promoting healthy default options is critical at a time when engaging in
healthy behaviors is difficult, but important, to both prevent and
manage chronic disease and decrease COVID-19 risk. This commentary
discusses COVID-19’s impact on restaurant operations and healthy eating,
and the resulting challenges and opportunities for this promising health
promotion intervention. \| \|JMIR public health and surveillance \|The
COVID-19 pandemic has greatly limited patients’ access to care for
spine-related symptoms and disorders. However, physical distancing
between clinicians and patients with spine-related symptoms is not
solely limited to restrictions imposed by pandemic-related lockdowns. In
most low- and middle-income countries, as well as many underserved
marginalized communities in high-income countries, there is little to no
access to clinicians trained in evidence-based care for people
experiencing spinal pain. The aim of this study is to describe the
development and present the components of evidence-based patient and
clinician guides for the management of spinal disorders where in-person
care is not available. Ultimately, two sets of guides were developed
(one for patients and one for clinicians) by extracting information from
the published Global Spine Care Initiative (GSCI) papers. An
international, interprofessional team of 29 participants from 10
countries on 4 continents participated. The team included practitioners
in family medicine, neurology, physiatry, rheumatology, psychology,
chiropractic, physical therapy, and yoga, as well as epidemiologists,
research methodologists, and laypeople. The participants were invited to
review, edit, and comment on the guides in an open iterative consensus
process. The Patient Guide is a simple 2-step process. The first step
describes the nature of the symptoms or concerns. The second step
provides information that a patient can use when considering self-care,
determining whether to contact a clinician, or considering seeking
emergency care. The Clinician Guide is a 5-step process: (1) Obtain and
document patient demographics, location of primary clinical symptoms,
and psychosocial information. (2) Review the symptoms noted in the
patient guide. (3) Determine the GSCI classification of the patient’s
spine-related complaints. (4) Ask additional questions to determine the
GSCI subclassification of the symptom pattern. (5) Consider appropriate
treatment interventions. The Patient and Clinician Guides are designed
to be sufficiently clear to be useful to all patients and clinicians,
irrespective of their location, education, professional qualifications,
and experience. However, they are comprehensive enough to provide
guidance on the management of all spine-related symptoms or disorders,
including triage for serious and specific diseases. They are consistent
with widely accepted evidence-based clinical practice guidelines. They
also allow for adequate documentation and medical record keeping. These
guides should be of value during periods of government-mandated physical
or social distancing due to infectious diseases, such as during the
COVID-19 pandemic. They should also be of value in underserved
communities in high-, middle-, and low-income countries where there is a
dearth of accessible trained spine care clinicians. These guides have
the potential to reduce the overutilization of unnecessary and expensive
interventions while empowering patients to self-manage uncomplicated
spinal pain with the assistance of their clinician, either through
direct in-person consultation or via telehealth communication. ©Scott
Haldeman, Margareta Nordin, Patricia Tavares, Rajani Mullerpatan,
Deborah Kopansky-Giles, Vincent Setlhare, Roger Chou, Eric Hurwitz,
Caroline Treanor, Jan Hartvigsen, Michael Schneider, Ralph Gay, Jean
Moss, Joan Haldeman, David Gryfe, Adam Wilkey, Richard Brown, Geoff
Outerbridge, Stefan Eberspaecher, Linda Carroll, Reginald Engelbrecht,
Kait Graham, Nathan Cashion, Stefanie Ince, Erin Moon. Originally
published in JMIR Public Health and Surveillance
(<http://publichealth.jmir.org>), 17.02.2021. \| \|Journal of leukocyte
biology \|The global pandemic caused by the severe acute respiratory
syndrome coronavirus 2 (SARS-CoV-2) is a highly pathogenic RNA virus
causing coronavirus disease 2019 (COVID-19) in humans. Although most
patients with COVID-19 have mild illness and may be asymptomatic, some
will develop severe pneumonia, acute respiratory distress syndrome,
multi-organ failure, and death. RNA viruses such as SARS-CoV-2 are
capable of hijacking the epigenetic landscape of host immune cells to
evade antiviral defense. Yet, there remain considerable gaps in our
understanding of immune cell epigenetic changes associated with severe
SARS-CoV-2 infection pathology. Here, we examined genome-wide DNA
methylation (DNAm) profiles of peripheral blood mononuclear cells from 9
terminally-ill, critical COVID-19 patients with confirmed SARS-CoV-2
plasma viremia compared with uninfected, hospitalized influenza,
untreated primary HIV infection, and mild/moderate COVID-19 HIV
coinfected individuals. Cell-type deconvolution analyses confirmed
lymphopenia in severe COVID-19 and revealed a high percentage of
estimated neutrophils suggesting perturbations to DNAm associated with
granulopoiesis. We observed a distinct DNAm signature of severe COVID-19
characterized by hypermethylation of IFN-related genes and
hypomethylation of inflammatory genes, reinforcing observations in
infection models and single-cell transcriptional studies of severe
COVID-19. Epigenetic clock analyses revealed severe COVID-19 was
associated with an increased DNAm age and elevated mortality risk
according to GrimAge, further validating the epigenetic clock as a
predictor of disease and mortality risk. Our epigenetic results reveal a
discovery DNAm signature of severe COVID-19 in blood potentially useful
for corroborating clinical assessments, informing pathogenic mechanisms,
and revealing new therapeutic targets against SARS-CoV-2. ©2021 Society
for Leukocyte Biology. \| \|bioRxiv : the preprint server for biology
\|COVID-19 pandemic has ravaged the world, caused over 1.8 million
deaths in the first year, and severely affected the global economy.
Hawaii is not spared from the transmission of SARS-CoV-2 in the local
population, including high infection rates in racial and ethnic
minorities. Early in the pandemic, we described in this journal various
technologies used for the detection of SARS-CoV-2. Herein we
characterize a 969-bp SARS-CoV-2 segment of the S gene downstream of the
receptor-binding domain. At the John A. Burns School of Medicine
Biocontainment Facility, RNA was extracted from an oropharyngeal swab
and a nasal swab from two patients from Hawaii who were infected with
the SARS-CoV-2 in August 2020. Following PCR, the two viral strains were
sequenced using Sanger sequencing, and phylogenetic trees were generated
using MEGAX. Phylogenetic tree results indicate that the virus has been
introduced to Hawaii from multiple sources. Further, we decoded 13
single nucleotide polymorphisms across 13 unique SARS-CoV-2 genomes
within this region of the S gene, with one non-synonymous mutation
(P681H) found in the two Hawaii strains. The P681H mutation has unique
and emerging characteristics with a significant exponential increase in
worldwide frequency when compared to the plateauing of the now universal
D614G mutation. The P681H mutation is also characteristic of the new
SARS-CoV-2 variants from the United Kingdom and Nigeria. Additionally,
several mutations resulting in cysteine residues were detected,
potentially resulting in disruption of the disulfide bridges in and
around the receptor-binding domain. Targeted sequence characterization
is warranted to determine the origin of multiple introductions of
SARS-CoV-2 circulating in Hawaii. \| \|Journal of thoracic oncology :
official publication of the International Association for the Study of
Lung Cancer \|Severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) spreads mainly by means of aerosols (microdroplets) in
enclosed environments, especially those in which temperature and
humidity are regulated by means of air-conditioning. About 30% of
individuals infected with SARS-CoV-2 develop coronavirus disease 2019
(COVID-19) disease. Among them, approximately 25% require
hospitalization. In medicine, cases are identified as those who become
ill. During this pandemic, cases have been identified as those with a
positive SARS-CoV-2 polymerase chain reaction test, including
approximately 70% who were asymptomatic-this has caused unnecessary
anxiety. Individuals more than 65 years old, those affected by obesity,
diabetes, asthma, or are immune-depressed owing to cancer and other
conditions, are at a higher risk of hospitalization and of dying of
COVID-19. Healthy individuals younger than 40 years very rarely die of
COVID-19. Estimates of the COVID-19 mortality rate vary because the
definition of COVID-19-related deaths varies. Belgium has the highest
death rate at 154.9 per 100,000 persons, because it includes anyone who
died with symptoms compatible with COVID-19, even those never tested for
SARS-CoV-2. The United States includes all patients who died with a
positive test, whether they died because of, or with, SARS-CoV-2.
Countries that include only patients in which COVID-19 was the main
cause of death, rather than a cofactor, have lower death rates. Numerous
therapies are being developed, and rapid improvements are anticipated.
Because of disinformation, only approximately 50% of the U.S. population
plans to receive a COVID-19 vaccine. By sharing accurate information,
physicians, health professionals, and scientists play a key role in
addressing myths and anxiety, help public health officials enact
measures to decrease infections, and provide the best care for those who
become sick. In this article, we discuss these issues. Copyright © 2021
International Association for the Study of Lung Cancer. Published by
Elsevier Inc. All rights reserved. \| \|Journal of the American Academy
of Dermatology \|To update guidance regarding the management of
psoriatic disease during the COVID-19 pandemic. The task force (TF)
includes 18 physician voting members with expertise in dermatology,
rheumatology, epidemiology, infectious diseases, and critical care. The
TF was supplemented by nonvoting members, which included fellows and
National Psoriasis Foundation staff. Clinical questions relevant to the
psoriatic disease community were informed by inquiries received by the
National Psoriasis Foundation. A Delphi process was conducted. The TF
updated evidence for the original 22 statements and added 5 new
recommendations. The average of the votes was within the category of
agreement for all statements, 13 with high consensus and 14 with
moderate consensus. The evidence behind many guidance statements is
variable in quality and/or quantity. These statements provide guidance
for the treatment of patients with psoriatic disease on topics including
how the disease and its treatments affect COVID-19 risk, how medical
care can be optimized during the pandemic, what patients should do to
lower their risk of getting infected with severe acute respiratory
syndrome coronavirus 2 (including novel vaccination), and what they
should do if they develop COVID-19. The guidance is a living document
that is continuously updated by the TF as data emerge. Copyright © 2021
American Academy of Dermatology, Inc. Published by Elsevier Inc. All
rights reserved. \| \|ASAIO journal (American Society for Artificial
Internal Organs : 1992) \|NA \| \|Cardiology in the young \|A
12-year-old girl presented with fever and signs of systemic
inflammation, and was found to have junctional tachycardia. She was
subsequently diagnosed with Multisystem Inflammatory Syndrome in
Children and treated with intravenous immunoglobulin and steroids, which
led to resolution of the arrhythmia. \| \|Journal of emergency
management (Weston, Mass.) \|During the COVID-19 pandemic, some
nonprofit organizations (NPOs) have been struggling to maintain their
operations, while others are able to coordinate with partners to provide
programs and services locally and globally. This study explores how NPOs
are able to survive and actively engage in local and global COVID-19
responses by investigating the organizational capacities of the Tzu Chi
Foundation, a Taiwan-based international NPO. This study employs
interview data and secondary data from a variety of sources to answer
the research questions. Through this case study, we find that Tzu Chi
Foundation’s capacity to coordinate local and global COVID-19 issues
quickly, broadly, and effectively can be attributed to three main
factors: (1) clear mission and charismatic leadership, (2) rich
experience of disaster relief and recovery strategies, and (3) committed
and active volunteers. Moreover, we find that financial management
capacity and adaptive capacity are two crucial kinds of capacity for
enabling the Tzu Chi Foundation to survive and continuously engage in
emergency responses during the pandemic. We conclude with implications
for future nonprofit capacity and emergency management research. \|
\|Expert review of respiratory medicine \|The dramatic impact of
COVID-19 on humans worldwide has initiated an extraordinary search for
effective treatment approaches. One of these is the administration of
exogenous surfactant, which is being tested in ongoing clinical trials.
Exogenous surfactant is a life-saving treatment for premature infants
with neonatal respiratory distress syndrome. This treatment has also
been tested for acute respiratory distress syndrome (ARDS) with limited
success possibly due to the complexity of that syndrome. The 60-year
history of successes and failures associated with surfactant therapy
distinguishes it from many other treatments currently being tested for
COVID-19 and provides the opportunity to discuss the factors that may
influence the success of this therapy. Clinical data provide a strong
rationale for using exogenous surfactant in COVID-19 patients. Success
of this therapy may be influenced by the mechanical ventilation
strategy, the timing of treatment, the doses delivered, the method of
delivery and the preparations utilized. In addition, future development
of enhanced preparations may improve this treatment approach. Overall,
results from ongoing trials may not only provide data to indicate if
this therapy is effective for COVID-19 patients, but also lead to
further scientific understanding and improved treatment strategies. \|
\|Headache <ISOAbbreviation>Headache</ISOAbbreviation> </Journal>
<ArticleTitle>Early impact of the COVID-19 pandemic on outpatient
migraine care in Hawaii: Results of a quality improvement
survey.</ArticleTitle> <Pagination> <StartPage>149</StartPage>
<EndPage>156</EndPage> <MedlinePgn>149-156</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1111/head.14030</ELocationID>
<Abstract> <AbstractText Label="OBJECTIVE">A survey was implemented for
early assessment of pandemic-related practice processes and quality
improvement (QI).</AbstractText> <AbstractText Label="BACKGROUND">In
response to the public health measures in Hawaii to curtail the
coronavirus 2019 pandemic, Hawaii Pacific Neuroscience (HPN) adapted
their patient care to ensure continuity of neurological
treatment.</AbstractText> <AbstractText Label="METHODS">The telephone
survey was conducted on patients seen at HPN during the period of April
22, 2020-May 18, 2020 to address four areas related to patients’
outpatient experience: delivery of care, general well-being, experience
with telemedicine, and disease-specific questions.</AbstractText>
<AbstractText Label="RESULTS">A total of 928 patients were contacted of
which 429 (46.2%) patients responded and 367 (85.5%) agreed to
participate. A total of 133 patients with migraine and 234 patients with
other neurological conditions provided responses. Our migraine patients’
survey responses suggest that their well-being was disproportionately
negatively affected by the pandemic. Survey respondents with migraine
were significantly more likely than their non-migraine peers to report
worsening anxiety and sleep problems \[62/132 (47.0%) vs. 78/234
(33.3%), χ<sup>2</sup>  = 6.64, p = 0.010, and 64/132 (48.5%) vs. 73/234
(31.2%), χ<sup>2</sup>  = 10.77, p = 0.001\]; migraine patients also
reported worsening of depression as a result of the pandemic more than
patients with other diagnoses, though this was not statistically
significant \[44/132 (33.3%) vs. 57/234 (24.4%), χ<sup>2</sup>  = 3.40,
p = 0.065\]. In regard to access to healthcare, significantly more
migraine patients reported running out of medications than those with
other diagnoses \[20/133 (15.0%) vs. 18/234 (7.7%), χ<sup>2</sup>
 = 4.93, p = 0.026\]. More avoided seeking medical help for new health
problems because of the pandemic \[30/133 (22.6%) vs. 30/234 (12.8%),
χ<sup>2</sup>  = 5.88, p = 0.015\]. Migraine patients were also
significantly impacted economically by the pandemic; 43/132 (32.4%) of
migraine patients reported losing their jobs as the result of the
pandemic versus 34/234 (14.5%) of their peers (χ<sup>2</sup>  = 11.20,
p \< 0.001). An increase in headache severity or frequency was reported
in 39/118 (33.1%) of respondents and 19/118 (16.1%) reported to using
more abortive therapy than usual. Telemedicine was well received by
almost all patients who took advantage of the option. Most of those
patients found telemedicine to be easy to use and as valuable as an
in-person visit. Migraine patients indicated with more frequency that
without the telemedicine option, they would have missed their medical
appointments \[37/68 (54.4%) vs. 56/144 (38.6%), χ<sup>2</sup>  = 4.31,
p = 0.038\]; a majority would prefer or consider telemedicine for future
appointments over in-person visits.</AbstractText>
<AbstractText Label="CONCLUSIONS">Insights gained from this QI survey to
the practice’s new pandemic-related processes include stressing
lifestyle modification, optimizing treatment plans, and continuing the
option of telemedicine.</AbstractText> <CopyrightInformation>© 2020
American Headache Society.</CopyrightInformation> </Abstract>
<AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Smith</LastName> <ForeName>Maiya</ForeName>
<Initials>M</Initials> <AffiliationInfo> <Affiliation>John Burns School
of Medicine, University of Hawaii, Honolulu, HI, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Nakamoto</LastName> <ForeName>Max</ForeName>
<Initials>M</Initials> <AffiliationInfo> <Affiliation>John Burns School
of Medicine, University of Hawaii, Honolulu, HI, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Crocker</LastName> <ForeName>Julie</ForeName>
<Initials>J</Initials> <AffiliationInfo> <Affiliation>John Burns School
of Medicine, University of Hawaii, Honolulu, HI, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y"> <LastName>Tiffany
Morden</LastName> <ForeName>Frances</ForeName> <Initials>F</Initials>
<AffiliationInfo> <Affiliation>John Burns School of Medicine, University
of Hawaii, Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Liu</LastName> <ForeName>Keke</ForeName>
<Initials>K</Initials> <AffiliationInfo> <Affiliation>John Burns School
of Medicine, University of Hawaii, Honolulu, HI, USA.</Affiliation>
</AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Ma</LastName> <ForeName>Enze</ForeName> <Initials>E</Initials>
<AffiliationInfo> <Affiliation>John Burns School of Medicine, University
of Hawaii, Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Chong</LastName>
<ForeName>Ariel</ForeName> <Initials>A</Initials> <AffiliationInfo>
<Affiliation>Undergraduate Education, University of Hawaii at Manoa,
Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Van</LastName>
<ForeName>Nicholas</ForeName> <Initials>N</Initials> <AffiliationInfo>
<Affiliation>Undergraduate Education, University of Hawaii at Manoa,
Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Vajjala</LastName>
<ForeName>Vimala</ForeName> <Initials>V</Initials> <AffiliationInfo>
<Affiliation>Headache & Facial Pain Center, Hawaii Pacific Neuroscience,
Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Carrazana</LastName>
<ForeName>Enrique</ForeName> <Initials>E</Initials> <AffiliationInfo>
<Affiliation>Clinical Research Center, Brain Research, Innovation &
Translation Labs, Hawaii Pacific Neuroscience, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Viereck</LastName> <ForeName>Jason</ForeName>
<Initials>J</Initials> <AffiliationInfo> <Affiliation>Clinical Research
Center, Brain Research, Innovation & Translation Labs, Hawaii Pacific
Neuroscience, Honolulu, HI, USA.</Affiliation> </AffiliationInfo>
</Author> <Author ValidYN="Y"> <LastName>Liow</LastName>
<ForeName>Kore</ForeName> <Initials>K</Initials> <AffiliationInfo>
<Affiliation>Clinical Research Center, Brain Research, Innovation &
Translation Labs, Hawaii Pacific Neuroscience, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> </AuthorList>
<Language>eng</Language> <PublicationTypeList>
<PublicationType UI="D016428">Journal Article</PublicationType>
</PublicationTypeList> <ArticleDate DateType="Electronic">
<Year>2020</Year> <Month>12</Month> <Day>14</Day> </ArticleDate>
</Article>

    <MedlineJournalInfo>
      <Country>United States</Country>
      <MedlineTA>Headache</MedlineTA>
      <NlmUniqueID>2985091R</NlmUniqueID>
      <ISSNLinking>0017-8748</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000293" MajorTopicYN="N">Adolescent</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000328" MajorTopicYN="N">Adult</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000368" MajorTopicYN="N">Aged</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000553" MajorTopicYN="Y">Ambulatory Care</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="Y">COVID-19</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002648" MajorTopicYN="N">Child</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006254" MajorTopicYN="N" Type="Geographic">Hawaii</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006297" MajorTopicYN="Y">Health Services Accessibility</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008297" MajorTopicYN="N">Male</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008875" MajorTopicYN="N">Middle Aged</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D008881" MajorTopicYN="Y">Migraine Disorders</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D010342" MajorTopicYN="Y">Patient Acceptance of Health Care</DescriptorName>
        <QualifierName UI="Q000706" MajorTopicYN="N">statistics &amp; numerical data</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D058996" MajorTopicYN="N">Quality Improvement</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086402" MajorTopicYN="N">SARS-CoV-2</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011795" MajorTopicYN="N">Surveys and Questionnaires</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D017216" MajorTopicYN="N">Telemedicine</DescriptorName>
        <QualifierName UI="Q000379" MajorTopicYN="N">methods</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D055815" MajorTopicYN="N">Young Adult</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">coronavirus 2019</Keyword>
      <Keyword MajorTopicYN="N">migraine</Keyword>
      <Keyword MajorTopicYN="N">neurology</Keyword>
      <Keyword MajorTopicYN="N">telemedicine</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="received"> <Year>2020</Year> <Month>6</Month>
<Day>19</Day> </PubMedPubDate> <PubMedPubDate PubStatus="revised">
<Year>2020</Year> <Month>8</Month> <Day>30</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="accepted"> <Year>2020</Year> <Month>9</Month>
<Day>4</Day> </PubMedPubDate> <PubMedPubDate PubStatus="pubmed">
<Year>2020</Year> <Month>12</Month> <Day>15</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="medline">
<Year>2021</Year> <Month>2</Month> <Day>26</Day> <Hour>6</Hour>
<Minute>0</Minute> </PubMedPubDate> <PubMedPubDate PubStatus="entrez">
<Year>2020</Year> <Month>12</Month> <Day>14</Day> <Hour>17</Hour>
<Minute>15</Minute> </PubMedPubDate> </History>
<PublicationStatus>ppublish</PublicationStatus> <ArticleIdList>
<ArticleId IdType="pubmed">33316097</ArticleId>
<ArticleId IdType="doi">10.1111/head.14030</ArticleId> </ArticleIdList>
<ReferenceList> REFERENCES \|A survey was implemented for early
assessment of pandemic-related practice processes and quality
improvement (QI). In response to the public health measures in Hawaii to
curtail the coronavirus 2019 pandemic, Hawaii Pacific Neuroscience (HPN)
adapted their patient care to ensure continuity of neurological
treatment. The telephone survey was conducted on patients seen at HPN
during the period of April 22, 2020-May 18, 2020 to address four areas
related to patients’ outpatient experience: delivery of care, general
well-being, experience with telemedicine, and disease-specific
questions. A total of 928 patients were contacted of which 429 (46.2%)
patients responded and 367 (85.5%) agreed to participate. A total of 133
patients with migraine and 234 patients with other neurological
conditions provided responses. Our migraine patients’ survey responses
suggest that their well-being was disproportionately negatively affected
by the pandemic. Survey respondents with migraine were significantly
more likely than their non-migraine peers to report worsening anxiety
and sleep problems \[62/132 (47.0%) vs. 78/234 (33.3%), χ2 = 6.64, p =
0.010, and 64/132 (48.5%) vs. 73/234 (31.2%), χ2 = 10.77, p = 0.001\];
migraine patients also reported worsening of depression as a result of
the pandemic more than patients with other diagnoses, though this was
not statistically significant \[44/132 (33.3%) vs. 57/234 (24.4%), χ2 =
3.40, p = 0.065\]. In regard to access to healthcare, significantly more
migraine patients reported running out of medications than those with
other diagnoses \[20/133 (15.0%) vs. 18/234 (7.7%), χ2 = 4.93, p =
0.026\]. More avoided seeking medical help for new health problems
because of the pandemic \[30/133 (22.6%) vs. 30/234 (12.8%), χ2 = 5.88,
p = 0.015\]. Migraine patients were also significantly impacted
economically by the pandemic; 43/132 (32.4%) of migraine patients
reported losing their jobs as the result of the pandemic versus 34/234
(14.5%) of their peers (χ2 = 11.20, p \< 0.001). An increase in headache
severity or frequency was reported in 39/118 (33.1%) of respondents and
19/118 (16.1%) reported to using more abortive therapy than usual.
Telemedicine was well received by almost all patients who took advantage
of the option. Most of those patients found telemedicine to be easy to
use and as valuable as an in-person visit. Migraine patients indicated
with more frequency that without the telemedicine option, they would
have missed their medical appointments \[37/68 (54.4%) vs. 56/144
(38.6%), χ2 = 4.31, p = 0.038\]; a majority would prefer or consider
telemedicine for future appointments over in-person visits. Insights
gained from this QI survey to the practice’s new pandemic-related
processes include stressing lifestyle modification, optimizing treatment
plans, and continuing the option of telemedicine. © 2020 American
Headache Society. \| \|Vaccine \|• It is imperative that vaccine
development and deployment include pregnant individuals because: • They
are at equal, if not greater, risk of severe COVID-19 illness than
nonpregnant patients. • Vertical transmission of SARS-CoV-2 to the fetus
has been documented. • Placental injury from SARS-CoV-2 may lead to
stillbirth and poor neonatal outcome. \| \|Annals of behavioral medicine
: a publication of the Society of Behavioral Medicine \|Investigating
antecedents of behaviors, such as wearing face coverings, is critical
for developing strategies to prevent SARS-CoV-2 transmission. The
purpose of this study was to determine associations between theory-based
behavioral predictors of intention to wear a face covering and actual
wearing of a face covering in public. Data from a cross-sectional panel
survey of U.S. adults conducted in May and June 2020 (N = 1,004) were
used to test a theory-based behavioral path model. We (a) examined
predictors of intention to wear a face covering, (b) reported use of
cloth face coverings, and (c) reported use of other face masks (e.g., a
surgical mask or N95 respirator) in public. We found that being female,
perceived importance of others wanting the respondent to wear a face
covering, confidence to wear a face covering, and perceived importance
of personal face covering use was positively associated with intention
to wear a face covering in public. Intention to wear a face covering was
positively associated with self-reported wearing of a cloth face
covering if other people were observed wearing cloth face coverings in
public at least “rarely” (aOR = 1.43), with stronger associations if
they reported “sometimes” (aOR = 1.83), “often” (aOR = 2.32), or
“always” (aOR = 2.96). For other types of face masks, a positive
association between intention and behavior was only present when
observing others wearing face masks “often” (aOR = 1.25) or “always”
(aOR = 1.48). Intention to wear face coverings and observing other
people wearing them are important behavioral predictors of adherence to
the CDC recommendation to wear face coverings in public. Published by
Oxford University Press on behalf of the Society of Behavioral Medicine
2020. \| \|International journal of infectious diseases : IJID :
official publication of the International Society for Infectious
Diseases \|This is a brief report on an unusual observation regarding
COVID-19 cases. The State of Hawaii is one of the most remote of the
Pacific islands and the population is approximately 1.4 million. The
racial and ethnic diversity is very high. For example, white Caucasians
comprise ∼25%, Asians including Japanese, Chinese, and other Asians
account for ∼30%, Hawaiians for 20%, and Pacific Islanders mostly from
Micronesia and Samoa comprise ∼4%. We discovered that the COVID-19 rate
in the latter group was up to 10 times that in all of the other groups
combined and they accounted for almost 30% of cases. Moreover, we are
unaware of COVID-19 transmission from Pacific Islanders to islanders
with other ethnicities. Thus, there is an epidemic within the epidemic
in Hawai’i. Copyright © 2020 The Author(s). Published by Elsevier Ltd..
All rights reserved. \| \|PloS one \|The purpose of this analysis was to
assess the variations in COVID-19 related mortality in relation to the
time differences in the commencement of virus circulation and
containment measures in the European Region. The data for the current
analysis (N = 50 countries) were retrieved from the John Hopkins
University dataset on the 7th of May 2020, with countries as study
units. A piecewise regression analysis was conducted with mortality and
cumulative incidence rates introduced as dependent variables and time
interval (days from the 22nd of January to the date when 100 first cases
were reported) as the main predictor. The country average life
expectancy at birth and outpatient contacts per person per year were
statistically adjusted for in the regression model. Mortality and
incidence were strongly and inversely intercorrelated with days from
January 22, respectively -0.83 (p\<0.001) and -0.73 (p\<0.001).
Adjusting for average life expectancy and outpatients contacts per
person per year, between days 33 to 50 from the 22nd of the January, the
average mortality rate decreased by 30.1/million per day (95% CI: 22.7,
37.6, p\<0.001). During interval 51 to 73 days, the change in mortality
was no longer statistically significant but still showed a decreasing
trend. A similar relationship with time interval was found for
incidence. Life expectancy and outpatients contacts per person per year
were not associated with mortality rate. Countries in Europe that had
the earliest COVID-19 circulation suffered the worst consequences in
terms of health outcomes, specifically mortality. The drastic social
isolation measures, quickly undertaken in response to those initial
outbreaks appear effective, especially in Eastern European countries,
where community circulation started after March 11th. The study
demonstrates that efforts to delay the early spread of the virus may
have saved an average 30 deaths daily per one million inhabitants. \|
\|Applied network science \|Deterministic epidemic models, such as the
Susceptible-Infected-Recovered (SIR) model, are immensely useful even if
they lack the nuance and complexity of social contacts at the heart of
network science modeling. Here we present a simple modification of the
SIR equations to include the heterogeneity of social connection
networks. A typical power-law model of social interactions from network
science reproduces the observation that individuals with a high number
of contacts, “hubs” or “superspreaders”, can become the primary conduits
for transmission. Conversely, once the tail of the distribution is
saturated, herd immunity sets in at a smaller overall recovered fraction
than in the analogous SIR model. The new dynamical equations suggest
that cutting off the tail of the social connection distribution, i.e.,
stopping superspreaders, is an efficient non-pharmaceutical intervention
to slow the spread of a pandemic, such as the Coronavirus Disease 2019
(COVID-19). © The Author(s) 2020. \| \|Scientific reports \|Since the
first case of the novel coronavirus disease (COVID-19) was confirmed in
Wuhan, China, social distancing has been promoted worldwide, including
in the United States, as a major community mitigation strategy. However,
our understanding remains limited in how people would react to such
control measures, as well as how people would resume their normal
behaviours when those orders were relaxed. We utilize an integrated
dataset of real-time mobile device location data involving 100 million
devices in the contiguous United States (plus Alaska and Hawaii) from
February 2, 2020 to May 30, 2020. Built upon the common human mobility
metrics, we construct a Social Distancing Index (SDI) to evaluate
people’s mobility pattern changes along with the spread of COVID-19 at
different geographic levels. We find that both government orders and
local outbreak severity significantly contribute to the strength of
social distancing. As people tend to practice less social distancing
immediately after they observe a sign of local mitigation, we identify
several states and counties with higher risks of continuous community
transmission and a second outbreak. Our proposed index could help
policymakers and researchers monitor people’s real-time mobility
behaviours, understand the influence of government orders, and evaluate
the risk of local outbreaks. \| \|ACS nano \|Coronavirus disease 2019
(COVID-19), due to infection by the severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2), is now causing a global pandemic. Aerosol
transmission of COVID-19, although plausible, has not been confirmed by
the World Health Organization (WHO) as a general transmission route.
Considering the rapid spread of SARS-CoV-2, especially nosocomial
outbreaks and other superspreading events, there is an urgent need to
study the possibility of airborne transmission and its impact on the
lung, the primary body organ attacked by the virus. Here, we review the
complete pathway of airborne transmission of SARS-CoV-2 from aerosol
dispersion in air to subsequent biological uptake after inhalation. In
particular, we first review the aerodynamic and colloidal mechanisms by
which aerosols disperse and transmit in air and deposit onto surfaces.
We then review the fundamental mechanisms that govern regional
deposition of micro- and nanoparticles in the lung. Focus is given to
biophysical interactions between particles and the pulmonary surfactant
film, the initial alveolar-capillary barrier and first-line host defense
system against inhaled particles and pathogens. Finally, we summarize
the current understanding about the structural dynamics of the
SARS-CoV-2 spike protein and its interactions with receptors at the
atomistic and molecular scales, primarily as revealed by molecular
dynamics simulations. This review provides urgent and multidisciplinary
knowledge toward understanding the airborne transmission of SARS-CoV-2
and its health impact on the respiratory system. \| \|Quality management
in health care \|The coronavirus disease-2019 (COVID-19) pandemic is
transforming the health care sector. As health care organizations move
from crisis mobilization to a new landscape of health and social needs,
organizational health literacy offers practical building blocks to
provide high-quality, efficient, and meaningful care to patients and
their families. Organizational health literacy is defined by the
Institute of Medicine as “the degree to which an organization implements
policies, practices, and systems that make it easier for people to
navigate, understand, and use information and services to take care of
their health.” This article synthesizes insights from organizational
health literacy in the context of current major health care challenges
and toward the goal of innovation in patient-centered care. We first
provide a brief overview of the origins and outlines of organizational
health literacy research and practice. Second, using an established
patient-centered innovation framework, we show how the existing work on
organizational health literacy can offer a menu of effective,
patient-centered innovative options for care delivery systems to improve
systems and outcomes. Finally, we consider the high value of management
focusing on organizational health literacy efforts, specifically for
patients in health care transitions and in the rapid transformation of
care into myriad distance modalities. This article provides practical
guidance for systems and informs decisions around resource allocation
and organizational priorities to best meet the needs of patient
populations even in the face of financial and workforce disruption.
Organizational health literacy principles and guidelines provide a road
map for promoting patient-centered care even in this time of crisis,
change, and transformation. Health system leaders seeking innovative
approaches can have access to well-established tool kits, guiding
models, and materials toward many organizational health literacy goals
across treatment, diagnosis, prevention, education, research, and
outreach. Copyright © 2020 Wolters Kluwer Health, Inc. All rights
reserved. \| \|Journal of medical Internet research \|The emergence of
SARS-CoV-2, the virus that causes COVID-19, has led to a global
pandemic. The United States has been severely affected, accounting for
the most COVID-19 cases and deaths worldwide. Without a coordinated
national public health plan informed by surveillance with actionable
metrics, the United States has been ineffective at preventing and
mitigating the escalating COVID-19 pandemic. Existing surveillance has
incomplete ascertainment and is limited by the use of standard
surveillance metrics. Although many COVID-19 data sources track
infection rates, informing prevention requires capturing the relevant
dynamics of the pandemic. The aim of this study is to develop dynamic
metrics for public health surveillance that can inform worldwide
COVID-19 prevention efforts. Advanced surveillance techniques are
essential to inform public health decision making and to identify where
and when corrective action is required to prevent outbreaks. Using a
longitudinal trend analysis study design, we extracted COVID-19 data
from global public health registries. We used an empirical difference
equation to measure daily case numbers for our use case in 50 US states
and the District of Colombia as a function of the prior number of cases,
the level of testing, and weekly shift variables based on a dynamic
panel model that was estimated using the generalized method of moments
approach by implementing the Arellano-Bond estimator in R. Examination
of the United States and state data demonstrated that most US states are
experiencing outbreaks as measured by these new metrics of speed,
acceleration, jerk, and persistence. Larger US states have high COVID-19
caseloads as a function of population size, density, and deficits in
adherence to public health guidelines early in the epidemic, and other
states have alarming rates of speed, acceleration, jerk, and 7-day
persistence in novel infections. North and South Dakota have had the
highest rates of COVID-19 transmission combined with positive
acceleration, jerk, and 7-day persistence. Wisconsin and Illinois also
have alarming indicators and already lead the nation in daily new
COVID-19 infections. As the United States enters its third wave of
COVID-19, all 50 states and the District of Colombia have positive rates
of speed between 7.58 (Hawaii) and 175.01 (North Dakota), and
persistence, ranging from 4.44 (Vermont) to 195.35 (North Dakota) new
infections per 100,000 people. Standard surveillance techniques such as
daily and cumulative infections and deaths are helpful but only provide
a static view of what has already occurred in the pandemic and are less
helpful in prevention. Public health policy that is informed by dynamic
surveillance can shift the country from reacting to COVID-19
transmissions to being proactive and taking corrective action when
indicators of speed, acceleration, jerk, and persistence remain positive
week over week. Implicit within our dynamic surveillance is an early
warning system that indicates when there is problematic growth in
COVID-19 transmissions as well as signals when growth will become
explosive without action. A public health approach that focuses on
prevention can prevent major outbreaks in addition to endorsing
effective public health policies. Moreover, subnational analyses on the
dynamics of the pandemic allow us to zero in on where transmissions are
increasing, meaning corrective action can be applied with precision in
problematic areas. Dynamic public health surveillance can inform
specific geographies where quarantines are necessary while preserving
the economy in other US areas. ©Lori Ann Post, Tariq Ziad Issa, Michael
J Boctor, Charles B Moss, Robert L Murphy, Michael G Ison, Chad J
Achenbach, Danielle Resnick, Lauren Nadya Singh, Janine White, Joshua
Marco Mitchell Faber, Kasen Culler, Cynthia A Brandt, James Francis
Oehmke. Originally published in the Journal of Medical Internet Research
(<http://www.jmir.org>), 03.12.2020. \| \|Frontiers in immunology \|The
current COVID-19 pandemic has claimed hundreds of thousands of lives and
its causative agent, SARS-CoV-2, has infected millions, globally. The
highly contagious nature of this respiratory virus has spurred massive
global efforts to develop vaccines at record speeds. In addition to
enhanced immunogen delivery, adjuvants may greatly impact protective
efficacy of a SARS-CoV-2 vaccine. To investigate adjuvant suitability,
we formulated protein subunit vaccines consisting of the recombinant S1
domain of SARS-CoV-2 Spike protein alone or in combination with either
CoVaccine HT™ or Alhydrogel. CoVaccine HT™ induced high titres of
antigen-binding IgG after a single dose, facilitated affinity maturation
and class switching to a greater extent than Alhydrogel and elicited
potent cell-mediated immunity as well as virus neutralizing antibody
titres. Data presented here suggests that adjuvantation with CoVaccine
HT™ can rapidly induce a comprehensive and protective immune response to
SARS-CoV-2. Copyright © 2020 Haun, Lai, Williams, Wong, Lieberman,
Pessaint, Andersen and Lehrer. \| \|International journal of infectious
diseases : IJID : official publication of the International Society for
Infectious Diseases \|Infection with severe acute respiratory syndrome
coronavirus 2 (SARS-CoV-2) is now a global pandemic. Emerging results
indicate a dysregulated immune response. Given the role of CCR5 in
immune cell migration and inflammation, we investigated the impact of
CCR5 blockade via the CCR5-specific antibody leronlimab on clinical,
immunological, and virological parameters in severe COVID-19 patients.
In March 2020, 10 terminally ill, critical COVID-19 patients received
two doses of leronlimab via individual emergency use indication. We
analyzed changes in clinical presentation, immune cell populations,
inflammation, as well as SARS-CoV-2 plasma viremia before and 14 days
after treatment. Over the 14-day study period, six patients survived,
two were extubated, and one discharged. We observed complete CCR5
receptor occupancy in all donors by day 7. Compared with the baseline,
we observed a concomitant statistically significant reduction in plasma
IL-6, restoration of the CD4/CD8 ratio, and resolution of SARS-CoV2
plasma viremia (pVL). Furthermore, the increase in the CD8 percentage
was inversely correlated with the reduction in pVL (r = -0.77, p =
0.0013). Our study design precludes clinical efficacy inferences but the
results implicate CCR5 as a therapeutic target for COVID-19 and they
form the basis for ongoing randomized clinical trials. Copyright © 2020
The Authors. Published by Elsevier Ltd.. All rights reserved. \| \|The
New England journal of medicine \|An outbreak of coronavirus disease
2019 (Covid-19) occurred on the U.S.S. Theodore Roosevelt, a
nuclear-powered aircraft carrier with a crew of 4779 personnel. We
obtained clinical and demographic data for all crew members, including
results of testing by real-time reverse-transcriptase polymerase chain
reaction (rRT-PCR). All crew members were followed up for a minimum of
10 weeks, regardless of test results or the absence of symptoms. The
crew was predominantly young (mean age, 27 years) and was in general
good health, meeting U.S. Navy standards for sea duty. Over the course
of the outbreak, 1271 crew members (26.6% of the crew) tested positive
for severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2)
infection by rRT-PCR testing, and more than 1000 infections were
identified within 5 weeks after the first laboratory-confirmed
infection. An additional 60 crew members had suspected Covid-19 (i.e.,
illness that met Council of State and Territorial Epidemiologists
clinical criteria for Covid-19 without a positive test result). Among
the crew members with laboratory-confirmed infection, 76.9% (978 of
1271) had no symptoms at the time that they tested positive and 55.0%
had symptoms develop at any time during the clinical course. Among the
1331 crew members with suspected or confirmed Covid-19, 23 (1.7%) were
hospitalized, 4 (0.3%) received intensive care, and 1 died. Crew members
who worked in confined spaces appeared more likely to become infected.
SARS-CoV-2 spread quickly among the crew of the U.S.S. Theodore
Roosevelt. Transmission was facilitated by close-quarters conditions and
by asymptomatic and presymptomatic infected crew members. Nearly half of
those who tested positive for the virus never had symptoms. Copyright ©
2020 Massachusetts Medical Society. \| \|Pediatric research \|NA \|
\|Journal of ethnopharmacology \|Traditional pharmacopeias have been
developed by multiple cultures and evaluated for efficacy and safety
through both historical/empirical iteration and more recently through
controlled studies using Western scientific paradigms and an increasing
emphasis on data science methodologies for network pharmacology.
Traditional medicines represent likely sources of relatively inexpensive
drugs for symptomatic management as well as potential libraries of new
therapeutic approaches. Leveraging this potential requires hard evidence
for efficacy that separates science from pseudoscience. We performed a
review of non-Western medical systems and developed case studies that
illustrate the epistemological and practical translative barriers that
hamper their transition to integration with Western approaches. We
developed a new data analytics approach, in silico convergence analysis,
to deconvolve modes of action, and potentially predict desirable
components of TM-derived formulations based on computational consensus
analysis across cultures and medical systems. Abstraction,
simplification and altered dose and delivery modalities were identified
as factors that influence actual and perceived efficacy once a medicine
is moved from a non-Western to Western setting. Case studies on these
factors highlighted issues with translation between non-Western and
Western epistemologies, including those where epistemological and
medicinal systems drive markets that can be epicenters for zoonoses such
as the novel Coronavirus. The proposed novel data science approach
demonstrated the ability to identify and predict desirable medicinal
components for a test indication, pain. Relegation of traditional
therapies to the relatively unregulated nutraceutical industry may lead
healthcare providers and patients to underestimate the therapeutic
potential of these medicines. We suggest three areas of emphasis for
this field: First, vertical integration and embedding of traditional
medicines into healthcare systems would subject them to appropriate
regulation and evidence-based practice, as viable integrative
implementation mode. Second, we offer a new Bradford-Hill-like framework
for setting research priorities and evaluating efficacy, with the goal
of rescuing potentially valuable therapies from the nutraceutical market
and discrediting those that are pseudoscience. Third, data analytics
pipelines offer new capacity to generate new types of TMS-inspired
medicines that are rationally-designed based on integrated knowledge
across cultures, and also provide an evaluative framework against which
to test claims of fidelity and efficacy to TMS made for nutraceuticals.
Copyright © 2020 The Author(s). Published by Elsevier B.V. All rights
reserved. \| \|Gastroenterology \|NA \| \|Prehospital and disaster
medicine \|This two-part article examines the global public health (GPH)
information system deficits emerging in the coronavirus disease 2019
(COVID-19) pandemic. It surveys past, missed opportunities for public
health (PH) information system and operational improvements, examines
current megatrend changes to information management, and describes a new
multi-disciplinary model for population-based management (PBM) supported
by a GPH Database applicable to pandemics and GPH crises. \| \|The
Journal of infectious diseases \|It is critical to identify potential
causal targets for SARS-CoV-2, which may guide drug repurposing options.
We assessed the associations between genetically predicted protein
levels and COVID-19 severity. Leveraging data from the COVID-19 Host
Genetics Initiative comparing 6492 hospitalized COVID-19 patients and 1
012 809 controls, we identified 18 proteins with genetically predicted
levels to be associated with COVID-19 severity at a false discovery rate
of \<0.05, including 12 that showed an association even after Bonferroni
correction. Of the 18 proteins, 6 showed positive associations and 12
showed inverse associations. In conclusion, we identified 18 candidate
proteins for COVID-19 severity. © The Author(s) 2020. Published by
Oxford University Press for the Infectious Diseases Society of America.
All rights reserved. For permissions, e-mail:
<journals.permissions@oup.com>. \| \|Acta neurologica Scandinavica \|NA
\| \|Journal of applied physiology (Bethesda, Md. : 1985) \|NA \|
\|Frontiers in oncology \|On March 11, 2020, the WHO has declared the
coronavirus disease 2019 (COVID-19) a global pandemic. As the last few
months have profoundly changed the delivery of health care in the world,
we should recognize the effort of numerous comprehensive cancer centers
to share experiences and knowledge to develop best practices to care for
oncological patients during the COVID-19 pandemic. Patients as well as
physicians must be aware of all these constraints and profound social,
personal, and medical challenges posed by the tackling of this deadly
disease in everyday life in order to adjust to such a completely novel
scenario. This review will discuss facing the challenges and the current
approaches that cancer centers in Italy and United States are adopting
in order to cope with clinical and research activities. Copyright © 2020
Terracciano, Buonerba, Scafuri, De Berardinis, Calin, Ferrajoli, Fabbri
and Cimmino. \| \|Journal of public health management and practice :
JPHMP \|NA \| \|World development \|Coffee supports the livelihoods of
millions of smallholder farmers in more than 52 countries, and generates
billions of dollars in revenue. The threats that COVID-19 pose to the
global coffee sector is daunting with profound implications for coffee
production. The financial impacts will be long-lived and uneven, and
smallholders will be among the hardest hit. We argue that the impacts
are rooted in the systemic vulnerability of the coffee production system
and the unequal ways the sector is organized: Large revenues from the
sale of coffee in the Global North are made possible by mostly
impoverished smallholders in the Global South. COVID-19 will accentuate
the existing vulnerabilities and create new ones, forcing many
smallholders into alternative livelihoods. This outcome, however, is not
inevitable. COVID-19 presents an opportunity to rebalance the system
that currently creates large profits on one end of the supply chain and
great vulnerability on the other. © 2020 Elsevier Ltd. All rights
reserved. \| \|Hawai’i journal of health & social welfare \|NA \|
\|Hawai’i journal of health & social welfare \|Infections with the
SARS-CoV-2 virus are increasing in Hawai’i at alarming rates. In the
absence of a SARS-CoV-2 virus vaccine, the options for control include
social distancing, improved hygiene, and face mask use. There is
evidence that mask use may decrease the rates of viral transmission. The
rate of effective face mask use has not yet been established in Hawai’i.
The authors performed an observational study at 2 locations in Honolulu
and evaluated outdoor face mask use compliance in 200 people.
Simultaneous observations were performed in a downtown Honolulu business
area and in Waikiki, an area focusing on tourism. Overall, 77% of all
subjects used face masks in an appropriate fashion, covering their nose
and mouth, while 23% were either incorrectly masked or not masked. The
rate of compliance with correct public mask use in downtown Honolulu
(88%) was significantly higher than in Waikiki (66%) (P=.0003, Odds
Ratio \[95% Confidence Interval\]=3.78 \[1.82, 7.85\]) These findings
suggest that there are opportunities for improvement in rates of public
face mask use and a potential decrease in the spread of COVID-19 in our
population. Four proposed actions are suggested, including a
reassessment of the face mask exemption requirements, enhanced mask
compliance education, non-threatening communication for non-compliance,
and centralization of information of the public compliance with face
mask use. ©Copyright 2020 by University Health Partners of Hawai‘i (UHP
Hawai‘i). \| \|Journal of psychiatric research \|NA \| \|BMJ (Clinical
research ed.) \|NA \| \|Disaster medicine and public health preparedness
\|The objectives of this study is to predict the possible trajectory of
coronavirus disease 2019 (COVID-19) spread in the United States.
Prediction and severity ratings of COVID-19 are essential for pandemic
control and economic reopening in the United States. In this study, we
apply the logistic and Gompertz model to evaluate possible turning
points of the COVID-19 pandemic in different regions. By combining
uncertainty and severity factors, this study constructed an indicator to
assess the severity of the coronavirus outbreak in various states. Based
on the index of severity ratings, different regions of the United States
are classified into 4 categories. The result shows that it is possible
to identify the first turning point in Montana and Hawaii. It is unclear
when the rest of the states will reach the first peak. However, it can
be inferred that 75% of regions will not reach the first peak of
coronavirus before August 2, 2020. It is still essential for the
majority of states to take proactive steps to fight against COVID-19
before August 2, 2020. \| \|Journal of the American Academy of
Dermatology \|To provide guidance about management of psoriatic disease
during the coronavirus disease 2019 (COVID-19) pandemic. A task force
(TF) of 18 physician voting members with expertise in dermatology,
rheumatology, epidemiology, infectious diseases, and critical care was
convened. The TF was supplemented by nonvoting members, which included
fellows and National Psoriasis Foundation (NPF) staff. Clinical
questions relevant to the psoriatic disease community were informed by
questions received by the NPF. A Delphi process was conducted. The TF
approved 22 guidance statements. The average of the votes was within the
category of agreement for all statements. All guidance statements
proposed were recommended, 9 with high consensus and 13 with moderate
consensus. The evidence behind many guidance statements is limited in
quality. These statements provide guidance for the management of
patients with psoriatic disease on topics ranging from how the disease
and its treatments impact COVID-19 risk and outcome, how medical care
can be optimized during the pandemic, what patients should do to lower
their risk of getting infected with severe acute respiratory syndrome
coronavirus 2 and what they should do if they develop COVID-19. The
guidance is intended to be a living document that will be updated by the
TF as data emerge. Copyright © 2020 American Academy of Dermatology,
Inc. Published by Elsevier Inc. All rights reserved. \| \|The Brazilian
journal of infectious diseases : an official publication of the
Brazilian Society of Infectious Diseases \|Severe Acute Respiratory
Syndrome Coronavirus 2 (SARS-CoV-2) is the cause of Coronavirus Disease
2019 (COVID-19). Although Real Time Reverse Transcription Polymerase
Chain Reaction (qRT-PCR) of respiratory specimens is the gold standard
test for detection of SARS-CoV-2 infection, collecting nasopharyngeal
swabs causes discomfort to patients and may represent considerable risk
for healthcare workers. The use of saliva as a diagnostic sample has
several advantages. The aim of this study was to validate the use of
saliva as a biological sample for diagnosis of COVID-19. This study was
conducted at Infectious Diseases Research Laboratory (LAPI), in
Salvador, Brazil. Participants presenting with signs/symptoms suggesting
SARS-CoV-2 infection underwent a nasopharyngeal swab (NPS) and/or
oropharyngeal swab (OPS), and saliva collection. Saliva samples were
diluted in PBS, followed by RNA isolation and RT-Real Time PCR for
SARS-CoV-2. Results of conventional vs saliva samples testing were
compared. Statistical analyses were performed using Statistical Package
for the Social Sciences software (SPSS) version 18.0. One hundred
fifty-five participants were recruited and samples pairs of NPS/OPS and
saliva were collected. The sensitivity and specificity of RT-PCR using
saliva samples were 94.4% (95% CI 86.4-97.8) and 97.62% (95% CI
91.7-99.3), respectively. There was an overall high agreement (96.1%)
between the two tests. Use of self-collected saliva samples is an easy,
convenient, and low-cost alternative to conventional NP swab-based
molecular tests. These results may allow a broader use of molecular
tests for management of COVID19 pandemic, especially in
resources-limited settings. Copyright © 2020 Sociedade Brasileira de
Infectologia. Published by Elsevier España, S.L.U. All rights reserved.
\| \|Journal of traumatic stress \|Leveraging technology to provide
evidence-based therapy for posttraumatic stress disorder (PTSD), such as
prolonged exposure (PE), during the COVID-19 pandemic helps ensure
continued access to first-line PTSD treatment. Clinical video
teleconferencing (CVT) technology can be used to effectively deliver PE
while reducing the risk of COVID-19 exposure during the pandemic for
both providers and patients. However, provider knowledge, experience,
and comfort level with delivering mental health care services, such as
PE, via CVT is critical to ensure a smooth, safe, and effective
transition to virtual care. Further, some of the limitations associated
with the pandemic, including stay-at-home orders and physical
distancing, require that providers become adept at applying principles
of exposure therapy with more flexibility and creativity, such as when
assigning in vivo exposures. The present paper provides the rationale
and guidelines for implementing PE via CVT during COVID-19 and includes
practical suggestions and clinical recommendations. Published 2020. This
article is a U.S. Government work and is in the public domain in the
USA. \| \|Behavior analysis in practice \|Applied behavior analysis
(ABA) services have been provided primarily in the fields of health care
and education across various settings using an in-person service
delivery model. Due to the COVID-19 pandemic, the necessity of and
demand for ABA services using telehealth have increased. The purpose of
the present article was to cross-examine the ethical codes and
guidelines of different, but related fields of practice and to discuss
potential implications for telehealth-based ABA service delivery. We
reviewed the telehealth-specific ethical codes and guidelines of the
American Psychological Association, the American Academy of Pediatrics,
and the National Association of Social Workers, along with the related
ABA literature. These organizations addressed several useful and unique
ethical concerns that have not been addressed in ABA literature. We also
developed a brief checklist for ABA practitioners to evaluate their
telehealth readiness by meeting the legal, professional, and ethical
requirements of ABA services. © Association for Behavior Analysis
International 2020. \| \|Oral surgery \|NA \| \|RMD open \|Reactive
arthritis (ReA) is typically preceded by sexually transmitted disease or
gastrointestinal infection. An association has also been reported with
bacterial and viral respiratory infections. Herein, we report the first
case of ReA after the he severe acute respiratory syndrome coronavirus 2
(SARS-CoV-2) infection. This male patient is in his 50s who was admitted
with COVID-19 pneumonia. On the second day of admission, SARS-CoV-2 PCR
was positive from nasopharyngeal swab specimen. Despite starting
standard dose of favipiravir, his respiratory condition deteriorated
during hospitalisation. On the fourth hospital day, he developed acute
respiratory distress syndrome and was intubated. On day 11, he was
successfully extubated, subsequently completing a 14-day course of
favipiravir. On day 21, 1 day after starting physical therapy, he
developed acute bilateral arthritis in his ankles, with mild enthesitis
in his right Achilles tendon, without rash, conjunctivitis, or preceding
diarrhoea or urethritis. Arthrocentesis of his left ankle revealed mild
inflammatory fluid without monosodium urate or calcium pyrophosphate
crystals. Culture of synovial fluid was negative. Plain X-rays of his
ankles and feet showed no erosive changes or enthesophytes. Tests for
syphilis, HIV, anti-streptolysin O (ASO), Mycoplasma, Chlamydia
pneumoniae, antinuclear antibody, rheumatoid factor, anticyclic
citrullinated peptide antibody and Human Leukocyte Antigen-B27 (HLA-B27)
were negative. Gonococcal and Chlamydia trachomatis urine PCR were also
negative. He was diagnosed with ReA. Nonsteroidal Anti-Inflammatory Drug
(NSAID)s and intra-articular corticosteroid injection resulted in
moderate improvement. © Author(s) (or their employer(s)) 2020. Re-use
permitted under CC BY-NC. No commercial re-use. See rights and
permissions. Published by BMJ. \| \|American journal of infection
control \|Anesthesia providers are at risk for contracting COVID-19 due
to close patient contact, especially during shortages of personal
protective equipment. We present an easy to follow and detailed protocol
for producing 3D printed face shields and an effective decontamination
protocol, allowing their reuse. The University of Nebraska Medical
Center (UNMC) produced face shields using a combination of 3D printing
and assembly with commonly available products, and produced a simple
decontamination protocol to allow their reuse. To evaluate the
effectiveness of the decontamination protocol, we inoculated bacterial
suspensions of E. coli and S. aureus on to the face shield components,
performed the decontamination procedure, and finally swabbed and
enumerated organisms onto plates that were incubated for 12-24 hours.
Decontamination effectiveness was evaluated using the average log10
reduction in colony counts. Approximately 112 face shields were
constructed and made available for use in 72 hours. These methods were
successfully implemented for in-house production at UNMC and at Tripler
Army Medical Center (Honolulu, Hawaii). Overall, the decontamination
protocol was highly effective against both E. coli and S. aureus,
achieving a ≥4 log10 (99.99%) reduction in colony counts for every
replicate from each component of the face shield unit. Face shields not
only act as a barrier against the soiling of N95 face masks, they also
serve as more effective eye protection from respiratory droplets over
standard eye shields. Implementation of decontamination protocols
successfully allowed face shield and N95 mask reuse, offering a higher
level of protection for anesthesiology providers at the onset of the
COVID-19 pandemic. In a time of urgent need, our protocol enabled the
rapid production of face shields by individuals with little to no 3D
printing experience, and provided a simple and effective decontamination
protocol allowing reuse of the face shields. Copyright © 2020
Association for Professionals in Infection Control and Epidemiology,
Inc. All rights reserved. \| \|Journal of perinatal medicine
<ISOAbbreviation>J Perinat Med</ISOAbbreviation> </Journal>
<ArticleTitle>Obstetric hospital preparedness for a pandemic: an
obstetric critical care perspective in response to
COVID-19.</ArticleTitle> <Pagination> <StartPage>874</StartPage>
<EndPage>882</EndPage> <MedlinePgn>874-882</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1515/jpm-2020-0281</ELocationID>
<Abstract> <AbstractText>The Coronavirus disease 2019 (COVID-19) caused
by Severe Acute Respiratory Syndrome Coronavirus-2 (SARS-CoV-2) pandemic
has had a rapid and deadly onset, spreading quickly throughout the
world. Pregnant patients have had high mortality rates, perinatal
losses, and Intensive Care Unit (ICU) admissions from acute respiratory
syndrome Coronavirus (SARS-CoV) and Middle East respiratory syndrome
Coronavirus (MERS-CoV) in the past. Potentially, a surge of patients may
require hospitalization and ICU care beyond the capacity of the health
care system. This article is to provide institutional guidance on how to
prepare an obstetric hospital service for a pandemic, mass casualty, or
natural disaster by identifying a care model and resources for a large
surge of critically ill pregnant patients over a short time. We
recommend a series of protocols, education, and simulation training,
with a structured and tiered approach to match the needs for the
patients, for hospitals specialized in obstetrics.</AbstractText>
</Abstract> <AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Harvey</LastName> <ForeName>Scott</ForeName>
<Initials>S</Initials> <AffiliationInfo> <Affiliation>University of
Hawaii at Manoa, John A Burns School of Medicine, Department of
Obstetrics, Gynecology and Women’s Health, Honolulu, Hawaii,
USA.</Affiliation> </AffiliationInfo> <AffiliationInfo>
<Affiliation>Kapiolani Medical Center for Women and Children, Department
of Obstetrics/Gynecology and Womens Health1319 Punahou Street Suite 824,
Honolulu, Hawaii 96816, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Zalud</LastName>
<ForeName>Ivica</ForeName> <Initials>I</Initials> <AffiliationInfo>
<Affiliation>University of Hawaii at Manoa, John A Burns School of
Medicine, Department of Obstetrics, Gynecology and Women’s Health,
Honolulu, Hawaii, USA.</Affiliation> </AffiliationInfo>
<AffiliationInfo> <Affiliation>Kapiolani Medical Center for Women and
Children, Department of Obstetrics/Gynecology and Womens Health1319
Punahou Street Suite 824, Honolulu, Hawaii 96816, USA.</Affiliation>
</AffiliationInfo> </Author> </AuthorList> <Language>eng</Language>
<PublicationTypeList> <PublicationType UI="D016428">Journal
Article</PublicationType> </PublicationTypeList>
</Article>

    <MedlineJournalInfo>
      <Country>Germany</Country>
      <MedlineTA>J Perinat Med</MedlineTA>
      <NlmUniqueID>0361031</NlmUniqueID>
      <ISSNLinking>0300-5577</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000073640" MajorTopicYN="Y">Betacoronavirus</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="N">COVID-19</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D018352" MajorTopicYN="N">Coronavirus Infections</DescriptorName>
        <QualifierName UI="Q000150" MajorTopicYN="Y">complications</QualifierName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
        <QualifierName UI="Q000628" MajorTopicYN="N">therapy</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D003422" MajorTopicYN="Y">Critical Care</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D016638" MajorTopicYN="N">Critical Illness</DescriptorName>
        <QualifierName UI="Q000628" MajorTopicYN="N">therapy</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004189" MajorTopicYN="N">Disaster Planning</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D004632" MajorTopicYN="N">Emergency Medical Services</DescriptorName>
        <QualifierName UI="Q000941" MajorTopicYN="N">ethics</QualifierName>
        <QualifierName UI="Q000458" MajorTopicYN="N">organization &amp; administration</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006771" MajorTopicYN="N">Hospitals, Maternity</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D009774" MajorTopicYN="N">Obstetrics</DescriptorName>
        <QualifierName UI="Q000379" MajorTopicYN="Y">methods</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D058873" MajorTopicYN="Y">Pandemics</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D010561" MajorTopicYN="N">Personnel Staffing and Scheduling</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011024" MajorTopicYN="N">Pneumonia, Viral</DescriptorName>
        <QualifierName UI="Q000150" MajorTopicYN="Y">complications</QualifierName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
        <QualifierName UI="Q000628" MajorTopicYN="N">therapy</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011247" MajorTopicYN="N">Pregnancy</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011251" MajorTopicYN="N">Pregnancy Complications, Infectious</DescriptorName>
        <QualifierName UI="Q000628" MajorTopicYN="N">therapy</QualifierName>
        <QualifierName UI="Q000821" MajorTopicYN="Y">virology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086402" MajorTopicYN="N">SARS-CoV-2</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D055872" MajorTopicYN="N">Surge Capacity</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">COVID-19</Keyword>
      <Keyword MajorTopicYN="N">disaster manual</Keyword>
      <Keyword MajorTopicYN="N">hospital preparedness</Keyword>
      <Keyword MajorTopicYN="N">isolation</Keyword>
      <Keyword MajorTopicYN="N">obstetric</Keyword>
      <Keyword MajorTopicYN="N">pandemic</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="received"> <Year>2020</Year> <Month>6</Month>
<Day>18</Day> </PubMedPubDate> <PubMedPubDate PubStatus="accepted">
<Year>2020</Year> <Month>6</Month> <Day>24</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="pubmed"> <Year>2020</Year> <Month>8</Month>
<Day>4</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="medline"> <Year>2020</Year> <Month>11</Month>
<Day>21</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="entrez"> <Year>2020</Year> <Month>8</Month>
<Day>4</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
</History> <PublicationStatus>ppublish</PublicationStatus>
<ArticleIdList> <ArticleId IdType="pubmed">32745072</ArticleId>
<ArticleId IdType="doi">10.1515/jpm-2020-0281</ArticleId>
<ArticleId IdType="pii">jpm-2020-0281</ArticleId> </ArticleIdList>
<ReferenceList> References \|The Coronavirus disease 2019 (COVID-19)
caused by Severe Acute Respiratory Syndrome Coronavirus-2 (SARS-CoV-2)
pandemic has had a rapid and deadly onset, spreading quickly throughout
the world. Pregnant patients have had high mortality rates, perinatal
losses, and Intensive Care Unit (ICU) admissions from acute respiratory
syndrome Coronavirus (SARS-CoV) and Middle East respiratory syndrome
Coronavirus (MERS-CoV) in the past. Potentially, a surge of patients may
require hospitalization and ICU care beyond the capacity of the health
care system. This article is to provide institutional guidance on how to
prepare an obstetric hospital service for a pandemic, mass casualty, or
natural disaster by identifying a care model and resources for a large
surge of critically ill pregnant patients over a short time. We
recommend a series of protocols, education, and simulation training,
with a structured and tiered approach to match the needs for the
patients, for hospitals specialized in obstetrics. \| \|Journal of
general and family medicine \|NA \| \|Journal of perinatal medicine
<ISOAbbreviation>J Perinat Med</ISOAbbreviation> </Journal>
<ArticleTitle>Academic clinical learning environment in obstetrics and
gynecology during the COVID-19 pandemic: responses and lessons
learned.</ArticleTitle> <Pagination> <StartPage>1013</StartPage>
<EndPage>1016</EndPage> <MedlinePgn>1013-1016</MedlinePgn> </Pagination>
<ELocationID EIdType="doi" ValidYN="Y">10.1515/jpm-2020-0239</ELocationID>
<Abstract> <AbstractText>COVID-19 pandemic is changing profoundly the
obstetrics and gynecology (OB/GYN) academic clinical learning
environment in many different ways. Rapid developments affecting our
learners, patients, faculty and staff require unprecedented
collaboration and quick, deeply consequential readjustments, almost on a
daily basis. We summarized here our experiences, opportunities,
challenges and lessons learned and outline how to move forward. The
COVID-19 pandemic taught us there is a clear need for collaboration in
implementing the most current evidence-based medicine, rapidly assess
and improve the everchanging healthcare environment by problem solving
and “how to” instead of “should we” approach. In addition, as a
community with very limited resources we have to rely heavily on
internal expertise, ingenuity and innovation. The key points to succeed
are efficient and timely communication, transparency in decision making
and reengagement. As time continues to pass, it is certain that more
lessons will emerge.</AbstractText> </Abstract>
<AuthorList CompleteYN="Y"> <Author ValidYN="Y">
<LastName>Olson</LastName> <ForeName>Holly L</ForeName>
<Initials>HL</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, John A Burns School of Medicine, University
of Hawaii, Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Towner</LastName>
<ForeName>Dena</ForeName> <Initials>D</Initials> <AffiliationInfo>
<Affiliation>Department of OB/GYN and Women’s Health, John A Burns
School of Medicine, University of Hawaii, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Hiraoka</LastName> <ForeName>Mark</ForeName>
<Initials>M</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, John A Burns School of Medicine, University
of Hawaii, Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
<Author ValidYN="Y"> <LastName>Savala</LastName>
<ForeName>Michael</ForeName> <Initials>M</Initials> <AffiliationInfo>
<Affiliation>Department of OB/GYN and Women’s Health, John A Burns
School of Medicine, University of Hawaii, Honolulu, HI,
USA.</Affiliation> </AffiliationInfo> </Author> <Author ValidYN="Y">
<LastName>Zalud</LastName> <ForeName>Ivica</ForeName>
<Initials>I</Initials> <AffiliationInfo> <Affiliation>Department of
OB/GYN and Women’s Health, John A Burns School of Medicine, University
of Hawaii, Honolulu, HI, USA.</Affiliation> </AffiliationInfo> </Author>
</AuthorList> <Language>eng</Language> <PublicationTypeList>
<PublicationType UI="D016428">Journal Article</PublicationType>
</PublicationTypeList>
</Article>

    <MedlineJournalInfo>
      <Country>Germany</Country>
      <MedlineTA>J Perinat Med</MedlineTA>
      <NlmUniqueID>0361031</NlmUniqueID>
      <ISSNLinking>0300-5577</ISSNLinking>
    </MedlineJournalInfo>
    <CitationSubset>IM</CitationSubset>
    <MeshHeadingList>
      <MeshHeading>
        <DescriptorName UI="D000073640" MajorTopicYN="Y">Betacoronavirus</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086382" MajorTopicYN="N">COVID-19</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D002982" MajorTopicYN="N">Clinical Clerkship</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D018352" MajorTopicYN="N">Coronavirus Infections</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="Y">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D003479" MajorTopicYN="N">Curriculum</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D003695" MajorTopicYN="N">Delivery of Health Care</DescriptorName>
        <QualifierName UI="Q000639" MajorTopicYN="N">trends</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D019317" MajorTopicYN="N">Evidence-Based Medicine</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005257" MajorTopicYN="N">Fellowships and Scholarships</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D005260" MajorTopicYN="N">Female</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006176" MajorTopicYN="N">Gynecology</DescriptorName>
        <QualifierName UI="Q000193" MajorTopicYN="Y">education</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006254" MajorTopicYN="N" Type="Geographic">Hawaii</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="N">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D006801" MajorTopicYN="N">Humans</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D007396" MajorTopicYN="N">Internship and Residency</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D009774" MajorTopicYN="N">Obstetrics</DescriptorName>
        <QualifierName UI="Q000193" MajorTopicYN="Y">education</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D058873" MajorTopicYN="Y">Pandemics</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011024" MajorTopicYN="N">Pneumonia, Viral</DescriptorName>
        <QualifierName UI="Q000453" MajorTopicYN="Y">epidemiology</QualifierName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D011247" MajorTopicYN="N">Pregnancy</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D000086402" MajorTopicYN="N">SARS-CoV-2</DescriptorName>
      </MeshHeading>
      <MeshHeading>
        <DescriptorName UI="D013337" MajorTopicYN="N">Students, Medical</DescriptorName>
      </MeshHeading>
    </MeshHeadingList>
    <KeywordList Owner="NOTNLM">
      <Keyword MajorTopicYN="N">COVID-19 pandemic</Keyword>
      <Keyword MajorTopicYN="N">academic learning in obstetrics and gynecology</Keyword>
      <Keyword MajorTopicYN="N">on-line learning</Keyword>
    </KeywordList>

</MedlineCitation> <PubmedData> <History>
<PubMedPubDate PubStatus="received"> <Year>2020</Year> <Month>5</Month>
<Day>28</Day> </PubMedPubDate> <PubMedPubDate PubStatus="accepted">
<Year>2020</Year> <Month>7</Month> <Day>2</Day> </PubMedPubDate>
<PubMedPubDate PubStatus="pubmed"> <Year>2020</Year> <Month>7</Month>
<Day>22</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="medline"> <Year>2020</Year> <Month>11</Month>
<Day>21</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
<PubMedPubDate PubStatus="entrez"> <Year>2020</Year> <Month>7</Month>
<Day>22</Day> <Hour>6</Hour> <Minute>0</Minute> </PubMedPubDate>
</History> <PublicationStatus>ppublish</PublicationStatus>
<ArticleIdList> <ArticleId IdType="pubmed">32692706</ArticleId>
<ArticleId IdType="doi">10.1515/jpm-2020-0239</ArticleId>
<ArticleId IdType="pii">jpm-2020-0239</ArticleId> </ArticleIdList>
<ReferenceList> References \|COVID-19 pandemic is changing profoundly
the obstetrics and gynecology (OB/GYN) academic clinical learning
environment in many different ways. Rapid developments affecting our
learners, patients, faculty and staff require unprecedented
collaboration and quick, deeply consequential readjustments, almost on a
daily basis. We summarized here our experiences, opportunities,
challenges and lessons learned and outline how to move forward. The
COVID-19 pandemic taught us there is a clear need for collaboration in
implementing the most current evidence-based medicine, rapidly assess
and improve the everchanging healthcare environment by problem solving
and “how to” instead of “should we” approach. In addition, as a
community with very limited resources we have to rely heavily on
internal expertise, ingenuity and innovation. The key points to succeed
are efficient and timely communication, transparency in decision making
and reengagement. As time continues to pass, it is certain that more
lessons will emerge. \| \|Paediatric respiratory reviews \|Models have
played an important role in policy development to address the COVID-19
outbreak from its emergence in China to the current global pandemic.
Early projections of international spread influenced travel restrictions
and border closures. Model projections based on the virus’s
infectiousness demonstrated its pandemic potential, which guided the
global response to and prepared countries for increases in
hospitalisations and deaths. Tracking the impact of distancing and
movement policies and behaviour changes has been critical in evaluating
these decisions. Models have provided insights into the epidemiological
differences between higher and lower income countries, as well as
vulnerable population groups within countries to help design
fit-for-purpose policies. Economic evaluation and policies have combined
epidemic models and traditional economic models to address the economic
consequences of COVID-19, which have informed policy calls for easing
restrictions. Social contact and mobility models have allowed evaluation
of the pathways to safely relax mobility restrictions and distancing
measures. Finally, models can consider future end-game scenarios,
including how suppression can be achieved and the impact of different
vaccination strategies. Copyright © 2020. Published by Elsevier Ltd. \|
\|Paediatric respiratory reviews \|Coronavirus disease 2019 (COVID-19)
is a newly emerged infectious disease caused by the severe acute
respiratory syndrome coronavirus-2 (SARS-CoV-2) that was declared a
pandemic by the World Health Organization on 11th March, 2020. Response
to this ongoing pandemic requires extensive collaboration across the
scientific community in an attempt to contain its impact and limit
further transmission. Mathematical modelling has been at the forefront
of these response efforts by: (1) providing initial estimates of the
SARS-CoV-2 reproduction rate, R0 (of approximately 2-3); (2) updating
these estimates following the implementation of various interventions
(with significantly reduced, often sub-critical, transmission rates);
(3) assessing the potential for global spread before significant case
numbers had been reported internationally; and (4) quantifying the
expected disease severity and burden of COVID-19, indicating that the
likely true infection rate is often orders of magnitude greater than
estimates based on confirmed case counts alone. In this review, we
highlight the critical role played by mathematical modelling to
understand COVID-19 thus far, the challenges posed by data availability
and uncertainty, and the continuing utility of modelling-based
approaches to guide decision making and inform the public health
response. †Unless otherwise stated, all bracketed error margins
correspond to the 95% credible interval (CrI) for reported estimates.
Copyright © 2020. Published by Elsevier Ltd. \| \|Hawai’i journal of
health & social welfare \|NA \| \|Journal of the American College of
Nutrition \|Background: In December 2019, the viral pandemic of
respiratory illness caused by COVID-19 began sweeping its way across the
globe. Several aspects of this infectious disease mimic metabolic events
shown to occur during latent subclinical magnesium deficiency.
Hypomagnesemia is a relatively common clinical occurrence that often
goes unrecognized since magnesium levels are rarely monitored in the
clinical setting. Magnesium is the second most abundant intracellular
cation after potassium. It is involved in \>600 enzymatic reactions in
the body, including those contributing to the exaggerated immune and
inflammatory responses exhibited by COVID-19 patients.Methods: A summary
of experimental findings and knowledge of the biochemical role magnesium
may play in the pathogenesis of COVID-19 is presented in this
perspective. The National Academy of Medicine’s Standards for Systematic
Reviews were independently employed to identify clinical and prospective
cohort studies assessing the relationship of magnesium with
interleukin-6, a prominent drug target for treating COVID-19.Results:
Clinical recommendations are given for prevention and treatment of
COVID-19. Constant monitoring of ionized magnesium status with
subsequent repletion, when appropriate, may be an effective strategy to
influence disease contraction and progression. The peer-reviewed
literature supports that several aspects of magnesium nutrition warrant
clinical consideration. Mechanisms include its “calcium-channel
blocking” effects that lead to downstream suppression of nuclear
factor-Kβ, interleukin-6, c-reactive protein, and other related
endocrine disrupters; its role in regulating renal potassium loss; and
its ability to activate and enhance the functionality of vitamin D,
among others.Conclusion: As the world awaits an effective vaccine,
nutrition plays an important and safe role in helping mitigate patient
morbidity and mortality. Our group is working with the Academy of
Nutrition and Dietetics to collect patient-level data from intensive
care units across the United States to better understand nutrition care
practices that lead to better outcomes. \| \|Hawai’i journal of health &
social welfare \|NA \| \|The American journal of clinical nutrition \|NA
\| \|Journal of hospital medicine \|NA \| \|Journal of gerontological
social work \|NA \| \|Journal of diabetes science and technology \|NA \|
\|Infectious diseases and therapy \|The emergence of SARS-CoV-2/2019
novel coronavirus (COVID-19) has created a global pandemic with no
approved treatments or vaccines. Many treatments have already been
administered to COVID-19 patients but have not been systematically
evaluated. We performed a systematic literature review to identify all
treatments reported to be administered to COVID-19 patients and to
assess time to clinically meaningful response for treatments with
sufficient data. We searched PubMed, BioRxiv, MedRxiv, and ChinaXiv for
articles reporting treatments for COVID-19 patients published between 1
December 2019 and 27 March 2020. Data were analyzed descriptively. Of
the 2706 articles identified, 155 studies met the inclusion criteria,
comprising 9152 patients. The cohort was 45.4% female and 98.3%
hospitalized, and mean (SD) age was 44.4 years (SD 21.0). The most
frequently administered drug classes were antivirals, antibiotics, and
corticosteroids, and of the 115 reported drugs, the most frequently
administered was combination lopinavir/ritonavir, which was associated
with a time to clinically meaningful response (complete symptom
resolution or hospital discharge) of 11.7 (1.09) days. There were
insufficient data to compare across treatments. Many treatments have
been administered to the first 9152 reported cases of COVID-19. These
data serve as the basis for an open-source registry of all reported
treatments given to COVID-19 patients at www.CDCN.org/CORONA . Further
work is needed to prioritize drugs for investigation in well-controlled
clinical trials and treatment protocols. \| \|Hawai’i journal of health
& social welfare \|Globally, coronavirus disease 2019 (COVID-19) is
threatening human health and changing the way people live. With the
increasing evidence showing comorbidities of COVID-19 and
non-communicable diseases (NCDs), the Pacific region, where
approximately 75% of deaths are due to NCDs, is significantly vulnerable
during this crisis unless urgent action is taken. Whilst enforcing the
critical mitigation measures of the COVID-19 pandemic in the Pacific, it
is also paramount to incorporate and strengthen NCD prevention and
control measures to safeguard people with NCDs and the general
population; keep people healthy and minimise the impact of COVID-19. To
sustain wellbeing of health, social relationships, and the economy in
the Pacific, it is a critical time for all governments, development
partners and civil societies to show regional solidarity in the fight
against emerging COVID-19 health crisis and existing Pacific NCDs crisis
through a whole of government and whole of society approach. ©Copyright
2020 by University Health Partners of Hawai‘i (UHP Hawai‘i). \|
\|Hawai’i journal of health & social welfare \|NA \| \|Hawai’i journal
of health & social welfare \|Nationwide shortages of tests that detect
severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) and
diagnose coronavirus disease 2019 (COVID-19) have led the US Food and
Drug Administration (FDA) to significantly relax regulations regarding
COVID-19 diagnostic testing. To date the FDA has given emergency use
authorization (EUA) to 48 COVID-19 in vitro diagnostic tests and 21 high
complexity molecular-based laboratory developed tests, as well as
implemented policies that give broad authority to clinical laboratories
and commercial manufacturers in the development, distribution, and use
of COVID-19 diagnostic tests. Currently, there are 2 types of diagnostic
tests available for the detection of SARS-CoV-2: (1) molecular and (2)
serological tests. Molecular detection of nucleic acid (RNA or DNA)
sequences relating to the suspected pathogen is indicative of an active
infection with the suspected pathogen. Serological tests detect
antibodies against the suspected pathogen, which are produced by an
individual’s immune system. A positive serological test result indicates
recent exposure to the suspected pathogen but cannot be used to
determine if the individual is actively infected with the pathogen or
immune to reinfection. In this article, the SARS-CoV-2 diagnostic tests
currently approved by the FDA under EUA are reviewed, and other
diagnostic tests that researchers are developing to detect SARS-CoV-2
infection are discussed. ©Copyright 2020 by University Health Partners
of Hawai‘i (UHP Hawai‘i). \| \|Biology of reproduction \|Expression of
angiotensin-converting enzyme 2, receptor of severe acute respiratory
syndrome coronavirus 2 (SARS-CoV-2), is high in the testes, therefore
SARS-CoV-2 infection and its association with male reproductive health
should be investigated in male coronavirus disease 2019 patients. \|
\|Pediatric critical care medicine : a journal of the Society of
Critical Care Medicine and the World Federation of Pediatric Intensive
and Critical Care Societies \|In the midst of the severe acute
respiratory syndrome coronavirus 2 pandemic, which causes coronavirus
disease 2019, there is a recognized need to expand critical care
services and beds beyond the traditional boundaries. There is
considerable concern that widespread infection will result in a surge of
critically ill patients that will overwhelm our present adult ICU
capacity. In this setting, one proposal to add “surge capacity” has been
the use of PICU beds and physicians to care for these critically ill
adults. Narrative review/perspective. Not applicable. Not applicable.
None. The virus’s high infectivity and prolonged asymptomatic shedding
have resulted in an exponential growth in the number of cases in the
United States within the past weeks with many (up to 6%) developing
acute respiratory distress syndrome mandating critical care services.
Coronavirus disease 2019 critical illness appears to be primarily
occurring in adults. Although pediatric intensivists are well versed in
the care of acute respiratory distress syndrome from viral pneumonia,
the care of differing aged adult populations presents some unique
challenges. In this statement, a team of adult and pediatric-trained
critical care physicians provides guidance on common “adult” issues that
may be encountered in the care of these patients and how they can best
be managed in a PICU. This concise scientific statement includes
references to the most recent and relevant guidelines and clinical
trials that shape management decisions. The intention is to assist PICUs
and intensivists in rapidly preparing for care of adult coronavirus
disease 2019 patients should the need arise. \| \|American journal of
otolaryngology \|The 2019 novel coronavirus (COVID-19) is
disproportionately impacting older individuals and healthcare workers.
Otolaryngologists are especially susceptible with the elevated risk of
aerosolization and corresponding high viral loads. This study utilizes a
geospatial analysis to illustrate the comparative risks of older
otolaryngologists across the United States during the COVID-19 pandemic.
Demographic and state population data were extracted from the State
Physician Workforce Reports published by the AAMC for the year 2018. A
geospatial heat map of the United States was then constructed to
illustrate the location of COVID-19 confirmed case counts and the
distributions of ENTs over 60 years for each state. In 2018, out of a
total of 9578 practicing U.S. ENT surgeons, 3081 were older than 60
years (32.2%). The states with the highest proportion of ENTs over 60
were Maine, Delaware, Hawaii, and Louisiana. The states with the highest
ratios of confirmed COVID-19 cases to the number of total ENTs over 60
were New York, New Jersey, Massachusetts, and Michigan. Based on our
models, New York, New Jersey, Massachusetts, and Michigan represent
states where older ENTs may be the most susceptible to developing severe
complications from nosocomial transmission of COVID-19 due to a
combination of high COVID-19 case volumes and a high proportion of ENTs
over 60 years. Copyright © 2020 Elsevier Inc. All rights reserved. \|
\|American journal of physical medicine & rehabilitation \|The global
outbreak of coronavirus disease 2019 has created an unprecedented
challenge to the society. Currently, the United States stands as the
most affected country, and the entire healthcare system is affected,
from emergency department, intensive care unit, postacute care,
outpatient, to home care. Considering the debility, neurological,
pulmonary, neuromuscular, and cognitive complications, rehabilitation
professionals can play an important role in the recovery process for
individuals with coronavirus disease 2019. Clinicians across the
nation’s rehabilitation system have already begun working to initiate
intensive care unit-based rehabilitation care and develop programs,
settings, and specialized care to meet the short- and long-term needs of
these individuals. We describe the anticipated rehabilitation demands
and the strategies to meet the needs of this population. The
complications from coronavirus disease 2019 can be reduced by (1)
delivering interdisciplinary rehabilitation that is initiated early and
continued throughout the acute hospital stay, (2) providing
patient/family education for self-care after discharge from inpatient
rehabilitation at either acute or subacute settings, and (3) continuing
rehabilitation care in the outpatient setting and at home through
ongoing therapy either in-person or via telehealth. \| \|Cancer
epidemiology, biomarkers & prevention : a publication of the American
Association for Cancer Research, cosponsored by the American Society of
Preventive Oncology \|The rapid pace of the severe acute respiratory
syndrome coronavirus 2 (SARS-CoV-2; COVID-19) pandemic presents
challenges to the real-time collection of population-scale data to
inform near-term public health needs as well as future investigations.
We established the COronavirus Pandemic Epidemiology (COPE) consortium
to address this unprecedented crisis on behalf of the epidemiology
research community. As a central component of this initiative, we have
developed a COVID Symptom Study (previously known as the COVID Symptom
Tracker) mobile application as a common data collection tool for
epidemiologic cohort studies with active study participants. This mobile
application collects information on risk factors, daily symptoms, and
outcomes through a user-friendly interface that minimizes participant
burden. Combined with our efforts within the general population, data
collected from nearly 3 million participants in the United States and
United Kingdom are being used to address critical needs in the emergency
response, including identifying potential hot spots of disease and
clinically actionable risk factors. The linkage of symptom data
collected in the app with information and biospecimens already collected
in epidemiology cohorts will position us to address key questions
related to diet, lifestyle, environmental, and socioeconomic factors on
susceptibility to COVID-19, clinical outcomes related to infection, and
long-term physical, mental health, and financial sequalae. We call upon
additional epidemiology cohorts to join this collective effort to
strengthen our impact on the current health crisis and generate a new
model for a collaborative and nimble research infrastructure that will
lead to more rapid translation of our work for the betterment of public
health. ©2020 American Association for Cancer Research. \| \|Clinical
infectious diseases : an official publication of the Infectious Diseases
Society of America \|With evidence of sustained transmission in more
than 190 countries, coronavirus disease 2019 (COVID-19) has been
declared a global pandemic. Data are urgently needed about risk factors
associated with clinical outcomes. A retrospective review of 323
hospitalized patients with COVID-19 in Wuhan was conducted. Patients
were classified into 3 disease severity groups (nonsevere, severe, and
critical), based on initial clinical presentation. Clinical outcomes
were designated as favorable and unfavorable, based on disease
progression and response to treatments. Logistic regression models were
performed to identify risk factors associated with clinical outcomes,
and log-rank test was conducted for the association with clinical
progression. Current standard treatments did not show significant
improvement in patient outcomes. By univariate logistic regression
analysis, 27 risk factors were significantly associated with clinical
outcomes. Multivariate regression indicated age \>65 years (P \< .001),
smoking (P = .001), critical disease status (P = .002), diabetes (P =
.025), high hypersensitive troponin I (\>0.04 pg/mL, P = .02),
leukocytosis (\>10 × 109/L, P \< .001), and neutrophilia (\>75 × 109/L,
P \< .001) predicted unfavorable clinical outcomes. In contrast, the
administration of hypnotics was significantly associated with favorable
outcomes (P \< .001), which was confirmed by survival analysis.
Hypnotics may be an effective ancillary treatment for COVID-19. We also
found novel risk factors, such as higher hypersensitive troponin I,
predicted poor clinical outcomes. Overall, our study provides useful
data to guide early clinical decision making to reduce mortality and
improve clinical outcomes of COVID-19. © The Author(s) 2020. Published
by Oxford University Press for the Infectious Diseases Society of
America. All rights reserved. For permissions, e-mail:
<journals.permissions@oup.com>. \| \|Critical care (London, England)
\|NA \| \|Surgical endoscopy \|The unprecedented pandemic of COVID-19
has impacted many lives and affects the whole healthcare systems
globally. In addition to the considerable workload challenges, surgeons
are faced with a number of uncertainties regarding their own safety,
practice, and overall patient care. This guide has been drafted at short
notice to advise on specific issues related to surgical service
provision and the safety of minimally invasive surgery during the
COVID-19 pandemic. Although laparoscopy can theoretically lead to
aerosolization of blood borne viruses, there is no evidence available to
confirm this is the case with COVID-19. The ultimate decision on the
approach should be made after considering the proven benefits of
laparoscopic techniques versus the potential theoretical risks of
aerosolization. Nevertheless, erring on the side of safety would warrant
treating the coronavirus as exhibiting similar aerosolization properties
and all members of the OR staff should use personal protective equipment
(PPE) in all surgical procedures during the pandemic regardless of known
or suspected COVID status. Pneumoperitoneum should be safely evacuated
via a filtration system before closure, trocar removal, specimen
extraction, or conversion to open. All emergent endoscopic procedures
performed during the pandemic should be considered as high risk and PPE
must be used by all endoscopy staff. \| \|Disaster medicine and public
health preparedness \|All levels of government are authorized to apply
coronavirus disease 2019 (COVID-19) protection measures; however, they
must consider how and when to ease lockdown restrictions to limit
long-term societal harm and societal instability. Leaders that use a
well-considered framework with an incremental approach will be able to
gradually restart society while simultaneously maintaining the public
health benefits achieved through lockdown measures. Economically
vulnerable populations cannot endure long-term lockdown, and most
countries lack the ability to maintain a full nationwide relief
operation. Decision-makers need to understand this risk and how the
Maslow hierarchy of needs and the social determinants of health can
guide whole of society policies. Aligning decisions with societal needs
will help ensure all segments of society are catered to and met while
managing the crisis. This must inform the process of incremental easing
of lockdowns to facilitate the resumption of community foundations, such
as commerce, education, and employment in a manner that protects those
most vulnerable to COVID-19. This study proposes a framework for
identifying a path forward. It reflects on baseline requirements,
regulations and recommendations, triggers, and implementation. Those
desiring a successful recovery from the COVID-19 pandemic need to adopt
an evidence-based framework now to ensure community stabilization and
sustainability. \| \|Journal for immunotherapy of cancer \|NA \| \|Pain
medicine (Malden, Mass.) \|It is nearly impossible to overestimate the
burden of chronic pain, which is associated with enormous personal and
socioeconomic costs. Chronic pain is the leading cause of disability in
the world, is associated with multiple psychiatric comorbidities, and
has been causally linked to the opioid crisis. Access to pain treatment
has been called a fundamental human right by numerous organizations. The
current COVID-19 pandemic has strained medical resources, creating a
dilemma for physicians charged with the responsibility to limit spread
of the contagion and to treat the patients they are entrusted to care
for. To address these issues, an expert panel was convened that included
pain management experts from the military, Veterans Health
Administration, and academia. Endorsement from stakeholder societies was
sought upon completion of the document within a one-week period. In
these guidelines, we provide a framework for pain practitioners and
institutions to balance the often-conflicting goals of risk mitigation
for health care providers, risk mitigation for patients, conservation of
resources, and access to pain management services. Specific issues
discussed include general and intervention-specific risk mitigation,
patient flow issues and staffing plans, telemedicine options, triaging
recommendations, strategies to reduce psychological sequelae in health
care providers, and resource utilization. The COVID-19 public health
crisis has strained health care systems, creating a conundrum for
patients, pain medicine practitioners, hospital leaders, and regulatory
officials. Although this document provides a framework for pain
management services, systems-wide and individual decisions must take
into account clinical considerations, regional health conditions,
government and hospital directives, resource availability, and the
welfare of health care providers. The Author(s) 2020. Published by
Oxford University Press on behalf of the American Academy of Pain
Medicine. This work is written by a US Government employee and is in the
public domain in the US. \| \|Journal of thoracic oncology : official
publication of the International Association for the Study of Lung
Cancer \|NA \|

Done! Knit the document, commit, and push.

## Final Pro Tip (optional)

You can still share the HTML document on github. You can include a link
in your `README.md` file as the following:

``` md
View [here](https://cdn.jsdelivr.net/gh/:user/:repo@:tag/:file) 
```

For example, if we wanted to add a direct link the HTML page of lecture
6, we could do something like the following:

``` md
View Week 6 Lecture [here]()
```
