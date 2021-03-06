
# Laravel FluentFM [![](https://travis-ci.org/thyyppa/laravel-fluent-fm.svg?branch=master)](https://travis-ci.org/thyyppa/laravel-fluent-fm) [![](https://github.styleci.io/repos/183276034/shield?branch=master)](https://github.styleci.io/repos/183276034)

FluentFM is a PHP package that connects to FileMaker Server's Data API using a fluent query builder style interface.

### Requirements  

- PHP 7.2+  
- FileMaker Server 17  
- Laravel 5+
  
### Installation  
  
#### Using Composer  
  
Use the command
  
```composer require thyyppa/laravel-fluent-fm```  
  
or include in your `composer.json` file  
```json  
{  
    "require": {  
        "thyyppa/laravel-fluent-fm": "dev-master"  
    }  
}  
```  

#### Prepare FileMaker

**Important! All tables and layouts *must* contain an `id` field.**

If you wish to use soft deletes your table and layout *must* contain the field `deleted_at`

The following fields are also recommended:
- `created_at` (for sorting by latest)
- `updated_at` (for sorting by last update)

All fields that you wish to access must be available in the layout that you provide while performing FileMaker operations through the api.

Ideally these will be hidden layouts that contain all fields raw, mirroring the table, but depending on your layout structure you may be able to use existing human-facing layouts. 

| ![](./assets/layout_thumbnail.png) | ![](./assets/container_thumbnail.png) |
|:---:|:---:|
| ![](./assets/database_thumbnail.png) ||

Ensure that `fmrest` is enabled on your privilege set, which you can define in 

`File > Manage > Security > [highlight user] > Privilege Set [Edit]`

![](./assets/privileges_thumbnail.png)

Also be sure that the Data API is enabled on the server.

If your server is installed locally this link should take you there:

[http://localhost:16001/admin-console/app/connectors/fmdapi](http://localhost:16001/admin-console/app/connectors/fmdapi)

Otherwise replace `localhost` with the server address.

![](./assets/admin_thumbnail.png)

---

### Register package

This package features auto-discovery, so registering the provider and 
facade should not be necessary, but if for any reason auto-discovery does not work for you, 
you can register the package manually as shown below.

Register the service provider in `config/app.php` (optional)
```php
'providers' => [
    // Other Service Providers

    Hyyppa\LaravelFluentFM\Providers\LaravelFluentFMServiceProvider::class,
],
```

Register the facade alias in `config/app.php` (optional)
```php
'aliases' => [
    // Other Aliases

    Hyyppa\LaravelFluentFM\Facades\FluentFM::class,
]
```

### Config

Publish the config files by running (optional)
```bash
php ./artisan vendor:publish --provider="Hyyppa\\LaravelFluentFM\\Providers\\LaravelFluentFMServiceProvider"
```

### Set Environment Variables

In your `.env` file, add the following items

```ini
FILEMAKER_FILE=[FileMaker file name without extension]
FILEMAKER_HOST=[FileMaker server address]
FILEMAKER_USER=[FileMaker file user]
FILEMAKER_PASS=[FileMaker file password]
```
  
### Usage

#### Getting records from layout

```php  
<?php  
  
use Hyyppa\LaravelFluentFM\Facades\FluentFM;
  
// get a single record as array
$record = FluentFM::record('layout', 'id')->get();

// get multiple records as array
$records = FluentFM::records('layout')->limit(10)->get();  
```

#### Performing a find operation

```php
$bobs = FluentFM::find('customers')->where('first','Bob')->get();
```

#### Creating a record

```php
$recordId = FluentFM::create('customers', [
    'id'    => 13
    'first' => 'Robert',
    'last'  => 'Paulson',
    'phone' => '406-555-0112',
]);
```

#### Updating a record

```php
// if multiple records are matched each will be updated
FluentFM::update('customers', [ 'phone' => '406-555-0199' ])
        ->where('id',13)
        ->limit(1)
        ->exec();
```

#### Deleting records

If you wish to use soft deletes your table and layout *must* contain the field `deleted_at`

```php
// hard delete removes record
FluentFM::delete('customers')
        ->where('id',13)
        ->limit(1)
        ->exec();

// soft delete sets record's deleted_at field
FluentFM::softDelete('customers')
        ->where('id',13)
        ->limit(1)
        ->exec();

// undeletes soft deleted records
FluentFM::undelete('customers')
        ->where('id',13)
        ->limit(1)
        ->exec();

// returns matching records that have not been soft deleted
$active = FluentFM::find('customers')
                  ->where('first','Bob')
                  ->withoutDeleted()
                  ->get();

// returns matching records even if soft deleted (default behavior)
$all = FluentFM::find('customers')
               ->where('first','Bob')
               ->withDeleted()
               ->get();
```

#### Uploading and downloading files to a record's container

```php
// if query matches multiple, file will be added to each
FluentFM::upload('customers', 'photo', './path/to/photo.jpg')
        ->where('id', 13)
        ->limit(1)
        ->exec();

// if query matches multiple, all files will be downloaded to path
FluentFM::download('customers', 'photo', './save/to/path/')
        ->where('id', 13)
        ->limit(1)
        ->exec();
```

#### Running FileMaker scripts

```php
FluentFM::find('customers')
        ->where('id',13)
        ->script('scriptname', 'parameter')
        ->presort('presort_scriptname', 'presort_scriptparam')
        ->prerequest('prerequest_scriptname', 'prerequest_scriptparam')
        ->get()
```

---

#### Chainable commands

```php
...

FluentFM::find( <layout> )
FluentFM::update( <layout>, [fields], [recordId] )
FluentFM::delete( <layout>, [recordId] )
FluentFM::softDelete( <layout>, [recordId] )
FluentFM::undelete( <layout>, [recordId] )
FluentFM::upload( <layout>, <field>, <filename>, [recordId] )
FluentFM::download( <layout>, <field>, [output_dir], [recordId] )
```

#### Chainable modifiers

```php
...

->record( <layout>, <id> )
->records( <layout>, [id] )
->limit( <limit> )
->offset( <offset> )
->sort( <field>, [ascending] )
->sortAsc( <field> )
->sortDesc( <field> )
->withPortals()
->withoutPortals()
->where( <field>, <params> ) // multiple calls act as "and"
->orWhere( <field>, <params> )
->whereEmpty( <field> )
->has( <field> )
->whereNotEmpty( <field> )
->withDeleted()
->withoutDeleted()
->script( <script>, [param], [type] )
->prerequest( <script>, [param] )
->presort( <script>, [param] )
```

#### End of chain methods

```php
...

->get()
->exec()
->create( <layout>, [fields] )
->latest( <layout>, [field] )       # table must have created_at field if [field] undefined 
->oldest( <layout>, [field] )       # table must have created_at field if [field] undefined 
->lastUpdate( <layout>, [field] )   # table must have updated_at field if [field] undefined 
->first()
->last()
```

#### Misc commands
```php
...

// set global fields on table
FluentFM::globals( [table], [ key => value ] )

// clear query parameters
FluentFM::clearQuery()

// clear query parameters and reset to default options
FluentFM::reset()
```


### License

MIT License  
  
#### Disclaimer  

> This project is an independent entity and has not been authorized,
> sponsored, or otherwise affiliated with FileMaker, Inc.
> FileMaker is a trademark of FileMaker, Inc., registered in the U.S. and other
> countries.
