# Project Setup

## Check
 - Check installed or not:
    * Elixir:
        $ elixir --version
    * Pheonix:
        $ mix phx.new --version
    * Mix (Build tool for Elixir):
        $ mix --version

    If any of the above is not installed then install it first and then proceed.

## Generate/Create
 - Generate or Create a new Phoenix App:
    * Phoenix:
        $ mix phx.new your_desired_app_name
        
        when asked about Fetch and Install dependencies ? Choose Y (Y for Yes).
        It would run following commands sequentially (You don't need to do anything):
        ```
        $ mix deps.get
        $ mix assets.setup
        $ mix desps.compile
        ```

        Then you'll see a prompt for 4 consecutive commands :

        We are almost there! The following steps are missing:

            $ cd proplex

        Then configure your database in config/dev.exs and run:

            $ mix ecto.create

        Start your Phoenix app with:

            $ mix phx.server

        You can also run your app inside IEx (Interactive Elixir) as:

            $ iex -S mix phx.server

        So according to the prompt, lets run the missing steps:
        `$ cd proplex`

        Now Before running ecto we need to create the database for our app, So lets do it through pgcli:

            $ pgcli -h localhost -U postgres   [you can replace "pgcli" with "psql". And password is "postgres"]
            $ create DATABASE proplex_live;


        Now create a user for this database:

        `$ create user testuser1 with password 'TestPass12345678';`

        Now grant access to the user for this database:
        `$ grant all privileges on database proplex_live to testuser1;`

        *** Switch Context to the Target Database (CRITICAL):
        Before you can grant schema-level permissions, you must switch away from the default postgres database and connect directly to your new database.

         ```
         sql
            -- Connect to the application database
        > \c proplex_dev

        ```
        
            *Note: Your prompt should change from `postgres=#` to `proplex_dev=#`.*
            ## Steps: Grant Schema-Level Permissions

            Now that you are inside `proplex_dev`, assign full rights of the `public` schema to your user so Ecto can create the `schema_migrations` table and your application tables.

            ```
            sql
                -- Change the owner of the public schema to your user (Highly Recommended)
                # ALTER SCHEMA public OWNER TO testuser1;

                -- 6. Explicitly grant all permissions on the public schema just to be safe
                # GRANT ALL ON SCHEMA public TO testuser1;
            ```


        Now Exit the postgresql database server and run migration:
        ```
        sql
            -- Quit the database server
        # \q
        ```

        Now that our database and user is ready, we need to update database configuration in our app
        before running Ecto:
        `$ cd proplex/config`

        Now open and edit `dev.exs` file:
        Remember this configuration is for local development only, You should never do these
        in production settings.
        Change the following:
        ```
        config :proplex, Proplex.Repo,
            username: "postgres",
            password: "postgres",
            hostname: "localhost",
            database: "proplex_dev",
            stacktrace: true,
            show_sensitive_data_on_connection_error: true,
            pool_size: 10

        To

        config :proplex, Proplex.Repo,
            username: "testuser1",
            password: "TestPass12345678",
            hostname: "localhost",
            database: "proplex_live",
            stacktrace: true,
            show_sensitive_data_on_connection_error: true,
            pool_size: 10
            
        ```

        NOTE: exit pgcli by typing `\q`. And if you don't want to use pgcli just replace the pgcli word with "psql".

        Now we're ready. Let's run Ecto:
        `$ mix ecto.create`

        Now let's run our default server or our Phoenix app:
        `$ mix phx.server`

        Now our server/app is running and we can access it using the url:
        [http://localhost:4000](http://localhost:4000)


## Check database and tables
 - First log in to database using the command:
    `$ pgcli -h localhost -U postgres `  [for postgres database]

    OR
    `$ pgcli -h localhost -U testuser1 -d proplex_live ` [for proplex_live database]
    
 - Then check databases or tables
    `> \l ` [to list all databases]
    OR
    `> \dt ` [to list all tables within a database]

* If you don't specifically name the database then postgresql assumes you want to
connect to a database which has name same as the username. Example:
`$ pgcli -h localhost -U postgres` 
Here, you're connecting to a database name "postgres" which is exactly same as the provided username.

Instead of `> \l` to show all database you can use this command:
`$ SELECT datname FROM pg_database WHERE datistemplate = false;`

Instead of `> \dt` to show all tables of a database you can use this command:
`$ SELECT tablename FROM pg_catalog.pg_tables WHERE schemaname = 'public';`


## Best Practice Going Forward

To keep your sanity intact when working with Postgres:

    Write identifiers in lowercase without quotes: CREATE USER testuser1; and GRANT ALL ON SCHEMA public TO testuser1;.

    Only use double quotes if you absolutely must: The only times you need double quotes are if a username or table name contains spaces, special characters, or matches a reserved SQL keyword (e.g., CREATE USER "user"; because user is a reserved word).

Stick to unquoted lowercase, and you'll never have to guess whether Postgres is looking for testuser1, TestUser1, or "testuser1".

* Without Quotes (testuser1): Postgres automatically converts the name to lowercase.
* With Double Quotes ("testuser1" or "TestUser1"): Postgres preserves the exact casing and forces a strict, literal match.