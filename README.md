# dbt-tutorial
Tutorial for DBT, following their Hello World docs:
https://docs.getdbt.com/tutorial/setting-up

## Notes
### Useful Commands
`dbt debug` :

Tells you about the ability to connect to your remote DB via your current set up. Very valuable.

Example output:
      
```
$ dbt debug
Running with dbt=0.19.0
dbt version: 0.19.0
python version: 3.8.8
python path: /usr/local/Cellar/dbt/0.19.0_1/libexec/bin/python3
os info: macOS-11.2.1-x86_64-i386-64bit
Using profiles.yml file at /Users/john.hanlon/.dbt/profiles.yml
Using dbt_project.yml file at /Users/john.hanlon/source/dbt-tutorial/jaffle-shop/dbt_project.yml

Configuration:
  profiles.yml file [OK found and valid]
  dbt_project.yml file [OK found and valid]

Required dependencies:
 - git [OK found]

Connection:
  method: service-account
  database: dbt-hello-world-305619
  schema: dbt_john
  location: US
  priority: interactive
  timeout_seconds: 300
  maximum_bytes_billed: None
  Connection test: OK connection ok
```

## Notes
DBT uses a profile YAML file located at `~/.dbt/profiles.yml`. Note that this file is used for managing credentials, and as such should not be committed into source control. If we used DBT for our project, we may need secrets or some other solution. 

If changing a model from a table into a view, use `dbt run --full-refresh` else you will run into an error at runtime, as the existing table conflicts with the view. Note that running with the `--full-refresh` flag will cause the table to be dropped.
