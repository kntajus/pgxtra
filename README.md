# Easy registration of custom PostgreSQL enum arrays with pgx

## Motivation

After deciding to try out [sqlc](https://github.com/kyleconroy/sqlc) along with [pgx](https://github.com/jackc/pgx)
for the very first time, I very quickly hit a bump in the road. I had a schema similar to:

```sql
CREATE TYPE fruit AS ENUM ('apple', 'banana', 'kiwi');

CREATE TABLE choices (
    choice_id serial PRIMARY KEY,
    fruits fruit[] NOT NULL
);
```

`sqlc` generated sensible-looking code for me, but when trying to do a basic `INSERT` or `SELECT` on the table I
was getting errors similar to:

`Cannot encode []store.Fruit into oid 16392 - []store.Fruit must implement Encoder or be converted to a string`

It took me a surprisingly long time to find enough pieces of the puzzle via search engines to find a solution,
enough that I thought it was worth codifying here for posterity and future use.

# Usage

Once you've created your `pgx` connection, pass it into `RegisterEnumArrayTypes` before carrying on as normal:

```go
package main

import (
    "context"
    "fmt"
    "os"

    "github.com/jackc/pgx/v4"
    "github.com/kntajus/pgxtra/v4"
)

func main() {
   	conn, err := pgx.Connect(context.Background(), os.Getenv("DATABASE_URL"))
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
		os.Exit(1)
	}
	defer conn.Close(context.Background())

	err = pgxtra.RegisterEnumArrayTypes(conn)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Unable to register enum array types: %v\n", err)
		os.Exit(1)
	}

    // Continue as normal...
}
```

# Versioning

Given this is explicitly referencing (and for) `v4` of `pgx`, I've mirrored that here for consistency.
