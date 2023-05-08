---
title: "Crabdb: A brief overview"
date: 2023-05-08T17:20:23+05:00
draft: false
---

In this blog post, I will give a brief overview of one of my semester project "crabdb".It is a simple database written in rust in which i have tried to implement both sql and nosql features by introducing a "mode" option at startup.
The main aim of this project for me is to get comfortable with rust and to know about the basic working of a database 
By the end of this post, we will try to have a basic working database.
So, let's get started!

# Introduction 
Repo : [https://github.com/OmerJauhar/Crabdb](https://github.com/OmerJauhar/Crabdb).   
The basic file structure of the crates will be as following 
```bash
-src
    -main.rs
-crates
    -parser
        -parser.rs
    -repl 
        -repl.rs
    -execution 
        -execution.rs
```


# REPl
A REPL (Read-Eval-Print Loop) is a tool that allows developers to interactively experiment with code, allowing them to type in commands and immediately see their output. It works by taking input from the user, interpreting and executing it, and then displaying the result back to the user.

In a database context, a REPL can be used to interact with the database by allowing developers to enter commands to query, insert, update or delete data. This makes it a very useful tool for testing and debugging database applications. For example, developers can use a REPL to quickly prototype a new query or test how a specific query will behave under different scenarios.

For the REPL, I am using [rustyline](https://github.com/kkawakam/rustyline).

```rust

use rustyline::error::ReadlineError;
use rustyline::{DefaultEditor, Result};
use parser::parser::parserftn ;
 
//main function will return a Result Enum 
pub fn replfunction() -> Result<()> {
    // `()` can be used when no completer is required
    let mut rl = DefaultEditor::new()?;
    #[cfg(feature = "with-file-history")]
    //rl.load history return a result enum 
    // err checks if the result is err or ok()
    if rl.load_history("history.txt").is_err() {
        println!("No previous history.");

    }
    println!("
                          _         _ _     
                         | |       | | |    
            ___ _ __ __ _| |__   __| | |__  
           / __| '__/ _` | '_ \\ / _` | '_ \\ 
          | (__| | | (_| | |_) | (_| | |_) |
           \\___|_|  \\__,_|_.__/ \\__,_|_.__/ 

                                       BY OJ        
");
    loop {
        // let a = 43 ; 
        println!("");
        let readline = rl.readline("crabdb >> ");
        match readline {
            Ok(line) => {
                // println!("{}",line);
                match rl.add_history_entry(line.as_str())
                {
                    Ok(_meow) => 
                    {
                        // println!("{:?}",meow) ; 
                        println!("");
                    }
                    Err(error)=>
                    {
                        println!("{:?}",error) ; 
                    }
                }
                // println!("Line: {}", line);
                parserftn(&line);
            },
            Err(ReadlineError::Interrupted) => {
                println!("CTRL-C");
                break
            },
            Err(ReadlineError::Eof) => {
                println!("CTRL-D");
                break
            },
            Err(err) => {
                println!("Error: {:?}", err);
                break
            }
        }
    }
    #[cfg(feature = "with-file-history")]
    rl.save_history("history.txt");
    Ok(())
}
```
# Parser 
A parser is used to analyze and interpret input data in a structured way, allowing your Rust database to understand and process commands or queries from users.
For parser I am using the sqlparser crate "[sqlparser](https://crates.io/crates/sqlparser)".
```rust 
let ast = Parser::parse_sql(&sql_dialect,sql_string); 
```
The parse_sql command takes two parameters the dialect and the string the is returned to use from the repl.  
Available dialects in sqlparser are 
```bash 
AnsiDialect
BigQueryDialect
ClickHouseDialect
GenericDialect
HiveDialect
MsSqlDialect
MySqlDialect
PostgreSqlDialect
RedshiftSqlDialect
SQLiteDialect
SnowflakeDialect
```
I am using the AnsiDialect.   

The Parser::parse_sql function fill return a Result Enum in which "Ok"s variant is a vector with  "statement".
```rust 
pub fn parse_sql(
    dialect: &dyn Dialect,
    sql: &str
) -> Result<Vec<Statement>, ParserError>
``` 
where statement is a enum with "58" variants.We can match on individual variant and use it accordingly.   
## Databases and Tables 
Inorder to create databases , use them and create tables in them i am using the following structure.  
```rust 
#[derive(Debug,Serialize, Deserialize,Clone)]
struct DataBases  
{
    name : String,
    tables : Vec<String>
}

```
should have named it DataBase instead of Databases but anyhow, the DatabasesArray will keep a track of the created DataBases by using an array of the above struct.  
```rust 
#[derive(Debug , Serialize , Deserialize)]
struct DatabasesArray
{
    array : Vec<DataBases> 
}

```
# Disk Management 
Initially I thought of using proper Disk Management but since I had a deadline to complete it and my knowledge of Disk Managament was not enough so therefore I am currently goona use serde serialize and deserialize to convert the structures to .json files.
```bash 
serde = { version = "1.0", features = ["derive"] }
```
# InsertFunction 
A quick overview of the parser usage and storing the data can be fetched from the implementation of insert function code.
```rust
Statement::Insert { or:_, into:_, table_name, columns, overwrite:_, source, partitioned:_, after_columns:_, table:_, on:_, returning:_ }  =>
{
    let mut finalvector = Vec::new();
    match *source.body{
        SetExpr::Values(values) =>
        {
            println!("{}",values);
            for i in values.rows[0].clone()
            {
                finalvector.push(i.to_string());               
            }
        }
        (_) => {}
    }
    let tablename = table_name.0[0].value.clone();
    let mut boolvar = false ;
    let mut contents1 = String::new();
    let mut file = File::open("current.txt");
    match &mut file 
    {
        Ok(file_unwrapped) =>
        {
            match file_unwrapped.read_to_string(&mut contents1)
            {
                Ok(_) => 
                {
                    if contents1 == String::from("DEFAULT"){
                        println!("No Database Selected ");
                    }
                    else  {
                        let mut fileread = File::open("person.json");
                        match &mut fileread {
                            Ok(file) =>
                            {
                                let mut contents = String::new();
                                file.read_to_string(&mut contents).unwrap();
                                let read_database_array :DatabasesArray = serde_json::from_str(&contents).unwrap();
                                 for i in read_database_array.array.iter()
                                 { 
                                    if i.name == contents1 
                                    {
                                        // println!("Table do exists");
                                        boolvar = true ; 
                                        break;
                                    }
                                 }
                                
                            }
                            Err(errormsg ) =>
                            {
                                println!("Inside error");
                                println!("{}",errormsg);
                            }
                        }
                    }
                }
                Err(errorstatement) =>
                {
                    println!("{}",errorstatement);
                }
            }
        }
        Err(errorstatement) =>
        {
            println!("{}",errorstatement);
        }
    }
    if boolvar
    {
        let filepath = tablename +".json";
        let mut fileread = File::open(filepath.clone());
            match &mut fileread {
                Ok(file) =>
                {
                    let mut contents = String::new();
                    // file.read_to_string(&mut contents).unwrap();
                    file.read_to_string(&mut contents).unwrap();
                    let mut read_table :Table = serde_json::from_str(&contents).unwrap();
                    read_table.insert(finalvector);
                    let mut filewrite = File::create(filepath);
                                             file.set_len(0);
                                             match &mut filewrite 
                                             {
                                                 Ok(file) => 
                                                 {
                                                     let serialized_parser_database_array  = serde_json::to_string(&read_table);
                                                     match &serialized_parser_database_array
                                                     {
                                                         Ok(spda_string) =>
                                                         {
                                                             match file.write_all(spda_string.as_bytes()) 
                                                             {
                                                                 Ok(_) =>
                                                                 {
                                                                    println!("Sucessful");
                                                                 }
                                                                 Err(errormsg) =>
                                                                 {   
                                                                     println!("error : {}",errormsg);
                                                                 }
                                                             }
                                                         }
                                                         Err(_) =>
                                                         {
                                                             println!("Error at serde_json");
                                                         }
                                                     }
                                                 }
                                                 Err(errorstatement) => { println!("{}",errorstatement)}
                }
                Err(errorstatement) => {println!("{}",errorstatement)}
    
    }
}
```
### => 
Since I found it hard to use static mutable global variables in rust therefore i am using basic file handling to create a file "current.txt" and store the current database being use in it by default  it will have "DEFAULT".   
### => 
Initially in the above code i am checking whether a database is selected and the table exists or not.   
Afterwards i am deserializing the data into the Table struct, erasing the file , inserting new data and than rewriting the file.   
```rust
let mut contents = String::new();
file.read_to_string(&mut contents).unwrap();
let mut read_table :Table = serde_json::from_str(&contents).unwrap();
read_table.insert(finalvector);
```
Serializing it and rewriting it here 
```rust 
 match &mut filewrite 
                                             
Ok(file) => 
{
    let serialized_parser_database_array  = serde_json::to_string(&read_table);
    match &serialized_parser_database_array
    {
        Ok(spda_string) =>
        {
            match file.write_all(spda_string.as_bytes()) 
            {
                Ok(_) =>
                {
                   println!("Sucessful");
                }
                Err(errormsg) =>
                {   
                    println!("error : {}",errormsg);
                }
            }
        }
        Err(_) =>
        {
            println!("Error at serde_json");
        }
    }
}
```
Whereas inorder to store the table data i am generating a .json file on the name of the table and again serializing and deserializing the Table struct
(Table struct will be explained in execution crate)
```rust 
if boolvar
{
    let filepath = tablename +".json";
    let mut fileread = File::open(filepath.clone());
        match &mut fileread {
            Ok(file) =>
            {
                let mut contents = String::new();
                // file.read_to_string(&mut contents).unwrap();
                file.read_to_string(&mut contents).unwrap();
                let mut read_table :Table = serde_json::from_str(&contents).unwrap();
                read_table.insert(finalvector);
                let mut filewrite = File::create(filepath);
                                         file.set_len(0);
                                         match &mut filewrite 
                                         {
                                             Ok(file) => 
                                             {
                                                 let serialized_parser_database_array  = serde_json::to_string(&read_table);
                                                 match &serialized_parser_database_array
                                                 {
                                                     Ok(spda_string) =>
                                                     {
                                                         match file.write_all(spda_string.as_bytes()) 
                                                         {
                                                             Ok(_) =>
                                                             {
                                                                println!("Sucessful");
                                                             }
                                                             Err(errormsg) =>
                                                             {   
                                                                 println!("error : {}",errormsg);
                                                             }
                                                         }
                                                     }
                                                     Err(_) =>
                                                     {
                                                         println!("Error at serde_json");
                                                     }
                                                 }
                                             }
                                             Err(errorstatement) => { println!("{}",errorstatement)}
            }
            Err(errorstatement) => {println!("{}",errorstatement)}

}

```
Where boolvar will be true only if the database is selected and the table do exist.
# Execution
Since I am planning to shift to disk management in the future therefore i am using a B-Tree for better indexing.   
The following structure is inspired by Johnscode in his project of building a database. 
```rust 
#[derive(Debug,Clone , Serialize, Deserialize)]
pub struct Column 
{
   pub name :String
}

#[derive(Debug, Clone, Default, Serialize, Deserialize)]
pub struct Table {
    /// row id to row
    rows: BTreeMap<usize, StoredRow>,
    /// Column info for all columns in the table
    columns: ColumnInfo,
}
```
 Table struct. It has two fields, rows and columns. rows is a BTreeMap that maps a row ID to a StoredRow, while columns is a ColumnInfo struct that contains information about all the columns in the table.   
 where stored row and ColumnInfo are 
 ```rust 
 type StoredRow = HashMap<String, String>;
type ColumnInfo = Vec<Column>;
 ```
 ![alt text](/gt.jpg)

 ## =>
  Implementation of all the functions and the full code can be viewed from the github repo

## Installation and Running 
Refer to the readme.md file of the github repo .    
Repo : [https://github.com/OmerJauhar/Crabdb](https://github.com/OmerJauhar/Crabdb).
# TODOs 
####  *  Implement Delete and update queries 
####  *  Implement Disk Management for "sql mode"
####  *  Make the parser more flexible by accepting nosql/sql interchangeable terms
```bash 
Table <==> Collection
Tuple/row <==> Document 
Column <==> Field 

```
####  *  Remove warning from current code 
####  *  Implement error handling with miette 

# Working Commands
Currently the following commands are working
```bash 
create database [database name] 
use [database name]
show databases 
show tables 
create table commands  
insert table commands 
drop table commands 
Select commands 

```
# Wrap Up 
As this was my first time working on a project in rust therefore any suggestions will be highly encouraged. 
Thanks for reading!  
Good Bye!
