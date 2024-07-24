# Extract file directory as database

## Description
This is a personal project, mainly to get some Rust project under the hood.  
The idea is to explore have a configurable system that will explore a file directory then extract files adding extra information to them, and save then in a sqlite database (It might not be useful, but at least a good exercice) and in a JSON format for exploitation in a front end application


I thought off this project for a long time, and actually did a first attempt to do it in 2015.



## What it does

### Extract (tool)
The extrat will explore the file directory, save each file following a certain rules (path, extension), and adding meta data to them based on the path of the file (categories, languages).  
Those information will be saved in a sqlite database.  
This process will start from an empty database, and fill it completly (no notion of update or other)

### Export (tool)
Take the information from the sqlite database, and save it as a JSON file for easy direct exploitation in an interface


### Diff (tool)
Can compare 2 of those extracted database and notify the differences between the two on a file level, like: not present in A; not present in B; different location


### Interface
This is a html/javascript interface, that will exploit the JSON file exported.  
On the interface we will see the listing of type of file and a listing of the file.  
We are able to filter data by categories (so folder in the file directory), other fields (languages, special field) or names



## Data format

### Database
Since sqlite is a file base database, we will just save one database for each type of data, and the table have the following information

#### File 
This will save the main information of the file
- id
- name (cleaned from extension)
- extension
- full path (full path from the source of the starting folder)
- file_date (ISO date of the creation date from the OS)
- creation_date (date of creation of the entry)

#### Category
This is the list of the categories
- id
- key
- label

#### File and category relation
Since a file can have multiple categorier and that categories can be assigned to multiple files, we have a many to many relation so we use a junction table
- id
- file_id
- category_id

#### Special fields
In the configuration, it is possible to assign specific folder a specific meaning. This is retranscribed as special fields that have a name and a type (boolean, string).  
The "key" is the name of the folder and the "label" is its representation

- id
- key
- label
- type (enum of boolean or string)

#### Special field value
List of values for a special fields

- id
- special_field_id
- key
- label

For boolean type we will only have a single entry in the special field value labeled as "true"

#### File and special field relation
File can have multiple special field of different type, and those field can be assigned to multiple file, so we have a many to many relation represented in a junction
- id
- file_id
- special_field__value_id


### JSON

The JSON reprsentation will be used for exploitation from an interface.

#### Composition
It has meta data parts and a array of type of data.  
There is a single JSON file for all the type of data
```
{
  name: "foobar",
  database_date: "2023-02-24T08:36:37.911Z",
  export_date: "2023-02-25T10:24:11.325Z",
  types: [
    ... // type of data
  ]
}

```

#### Type of data
It has meta data parts related to the type of data to represent, and also the list of entries for this type
```
{
  name: "foobar",
  categories: ["new", "archive", "current"],
  special_field: [
    {
      label: "important",
      type: "boolean",
    },
    {
      label: "language",
      type: "string"
      values: ["English", "French", "German"]
    }
  ]
  files: [
    ... // list of files
  ]
}

```


#### Files
List of files for this type of data
```
{
  name: "foobar",
  extension: ".txt",
  categories: ["new", "current"],
  special_field: {
    important: true,
    language: "German"
  }
}

```

```
{
  name: "meeting_record",
  extension: ".mp3",
  categories: ["new"],
  special_field: {
    important: false,
    language: "German"
  }
}

```

