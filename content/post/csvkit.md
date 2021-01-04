+++
author = "tdonovic"
date = 2021-01-05
title = "csvkit for schema inference"
+++
Let's say you use some db management tool like `alembic` or `flyway`, and you need to import some data into your db, as well as its schema, but you need to generate all the `sql` statements for this, so they can be comitted to git. On first inspection, sounded like I was going to have to write some gross python. Instead, a tool called `csvkit` can do this for you, nearly trivially.

1. Install `csvkit`
`brew install csvkit`

2. Import data to `sqlite`
`csvsql ~/foo.csv` --db=sqlite:///test.db --insert

3. Get `sqlite` to give you create table, as well as inserts
echo ".dump" | sqlite3 test.db > test.sql

As long as the inserts are compatible with your DB, this should work trivially. If they don't, you probably have a bit of `sed` to massage it into the correct format.
`csvsql` supports writing directly to db, and will generate the correct statments for your particular db flavour, but it will not output this directly, it must be connected directly to the db.


Upon further inspection, this doesn't look like it will work directly, and some `sed`'ing will need to be done to make this work. So instead, I use the method described in [here](https://stackoverflow.com/questions/18671/quick-easy-way-to-migrate-sqlite3-to-mysql) where [Alex Martelli](https://stackoverflow.com/questions/1067060/translating-perl-to-python) did something simmilar. 
