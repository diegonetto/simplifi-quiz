## Notes
- The assumption was made that every record is potentially a unique user. If at least one of its fields is present, it should be included in the datastore. 
- This repository contains a pre-generated SQLite datastore based on the parsed_data.txt file, but a new one can be generated by running the setup-datastore.js script.

## Usage
    $ node scripts/setup-datastore.js
    $ node scripts/search-by-keyword.js KEYWORD
    $ node scripts/search-by-uid.js UID

## Parsing
In an attempt to reduce the amount of data being written to the datastore, the following parsing decisions were made.

## IP Address and User Agent
Kept as is, if present

## URL and Referer 
Split into three segments.

### Host
- Strip protocol
- Split at "."s
- Remove "www"
- Remove common domain extensions

### Pathname
- Ignore empty paths "/"
- Remove "/", "=", "-", "_", "&", "|", "," and ";"
- Remove common domain extensions
- Remove common file extensions
- Ignore keyword if length < 3 (decision made after analyzing usefulness of [this file](https://github.com/diegonetto/simplifi-quiz/blob/master/keywords-under-three-characters.txt)'s contents)

### Query
- Parse params from query
- Use values of "q", "query", "refer", "referer", "referrer", and "search_query" keys

## Benchmarks
Since optimizations were to be made for seach efficiency, an SQLite [FTS4](http://www.sqlite.org/fts3.html) table was used for fast full text seaching of the keywords property. The trade-off is that a FTS4 table takes longer to initally build and uses up more space on disk than an ordinary SQLite table.

### Setting up the datastore
    $ time node scripts/setup-datastore.js 
    Setup: successfully created new /home/diego/Workspace/node/simplifi-quiz/db/datastore.sqlite
    Setup: successfully deleted old /home/diego/Workspace/node/simplifi-quiz/db/datastore.sqlite
    Setup: created users table
    Setup: finished parsing user data from /home/diego/Workspace/node/simplifi-quiz/parsed_data.txt

    real	40m9.033s
    user	0m46.107s
    sys  	2m22.569s

### Datastore size
    $ ls -lh db/datastore.sqlite 
    -rw-r--r-- 1 diego diego 37M Sep 17 10:33 db/datastore.sqlite

### Searching by keyword
    $ time node scripts/search-by-keyword.js toyota
    Keyword Search: successfully opened /home/diego/Workspace/node/simplifi-quiz/db/datastore.sqlite
    Keyword Search: searching for UIDs that are related to keyword = toyota
		19052
		49512
		58193
		74089
    Keyword Search: 4 UIDs found

    real	0m0.077s
    user	0m0.032s
    sys  	0m0.004s

### Searching by UID
    $ time node scripts/search-by-uid.js 19052
    UID Search: successfully opened /home/diego/Workspace/node/simplifi-quiz/db/datastore.sqlite
    UID Search: searching for UID = 19052
    { ip: '96.21.125.234',
      agent: 'Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; BTRS122130; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; BRI/2)',
      keywords: 'montreal kijiji autos vehicules autos camions 2010 Toyota RAV4 4WD 4x4 Juste 23k Comme Neuve W0QQAdIdZ345827041' }

    real	0m0.074s
    user	0m0.024s
    sys  	0m0.012s
