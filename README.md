Here’s a comprehensive README document for the Laravel package you've created (`CassandraEloquent`). This will guide users through installing, configuring, and using the package in their Laravel applications.

---

# Cassandra Eloquent Laravel Package

A Laravel package that mimics Eloquent ORM's functionality for Cassandra NoSQL database. This package enables you to use Cassandra like a relational database in Laravel, providing an elegant interface to interact with your data.

## Features

- Basic Eloquent-like model methods such as `all()`, `find()`, `create()`, `update()`, `save()`, and `delete()`.
- Query builder methods like `where()`, `orWhere()`, `orderBy()`, `limit()`, and `paginate()`.
- Utility methods such as `toArray()`, `toJson()`, `fresh()`, `isDirty()`, and `wasRecentlyCreated()`.
- Integration with Laravel’s database connection, making it simple to interact with Cassandra.

## Installation

### Step 1: Install the Package

Run the following command in your Laravel project directory to install the package via Composer:

```bash
composer require your-vendor-name/cassandra-eloquent
```

### Step 2: Register the Service Provider

In Laravel, add the service provider to the `config/app.php` file under the `providers` array:

```php
'providers' => [
    // Other service providers...
    YourVendorName\CassandraEloquent\CassandraServiceProvider::class,
],
```

### Step 3: Publish the Configuration (Optional)

If you have custom configuration for Cassandra, publish the configuration file using the following command:

```bash
php artisan vendor:publish --provider="YourVendorName\CassandraEloquent\CassandraServiceProvider"
```

This will publish the `cassandra.php` configuration file to `config/cassandra.php`.

### Step 4: Configure Cassandra Connection

Add your Cassandra connection settings in the `.env` file:

```env
CASSANDRA_HOST=127.0.0.1
CASSANDRA_PORT=9042
```

Optionally, you can edit the `config/cassandra.php` file to fine-tune your Cassandra connection.

### Step 5: Run Migrations (Optional)

Since Cassandra doesn't use migrations like traditional SQL databases, this step is not required. However, you should define your table schema manually and handle data operations as needed.

---

## Usage

### Step 1: Create a Model

To create a model that interacts with Cassandra, extend the `CassandraBaseModel` class provided by the package. Here's an example of how to create a `User` model:

```php
namespace App\Models;

use YourVendorName\CassandraEloquent\Models\CassandraBaseModel;

class User extends CassandraBaseModel
{
    protected $primaryKey = 'user_id';  // Define primary key if it's different
}
```

### Step 2: Use the Model

Now, you can use the `User` model as you would any Eloquent model. Below are examples of how to interact with your data.

#### Fetch All Users

```php
$users = User::all();
```

#### Find a User by ID

```php
$user = User::find(1);
```

#### Create a New User

```php
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john.doe@example.com',
]);
```

#### Update a User

```php
$user = User::find(1);
$user->update(['name' => 'Jane Doe']);
```

#### Delete a User

```php
$user = User::find(1);
$user->delete();
```

#### Query with Conditions

```php
$activeUsers = User::where('status', '=', 'active')->limit(10)->get();
```

#### Paginate Results

```php
$paginatedUsers = User::where('status', '=', 'active')->paginate(10);
```

---

## Methods Overview

### Basic Eloquent Methods

- **all()** – Fetches all records from the table.
- **find($id)** – Finds a record by its primary key.
- **findOrFail($id)** – Finds a record by primary key or throws an exception if not found.
- **first()** – Fetches the first record from the table.
- **firstOrFail()** – Fetches the first record or throws an exception if not found.
- **create(array $attributes)** – Inserts a new record with the provided attributes.
- **update(array $attributes)** – Updates the current record with the given attributes.
- **delete()** – Deletes the current record.
- **save()** – Saves the current model (creates or updates depending on its state).

### Query Builder Methods

- **where($column, $operator, $value)** – Adds a WHERE condition to the query.
- **orWhere($column, $operator, $value)** – Adds an OR WHERE condition.
- **orderBy($column, $direction)** – Orders the query results.
- **limit($value)** – Limits the number of records.
- **paginate($perPage)** – Paginates the results with the specified records per page.
- **count()** – Returns the number of records in the result set.
- **exists()** – Checks if a record exists.
- **pluck($column)** – Retrieves a single column’s values from the result set.

### Other Useful Methods

- **toArray()** – Converts the model to an array.
- **toJson()** – Converts the model to a JSON string.
- **fresh()** – Refreshes the model’s data from the database.
- **refresh()** – Reloads the model’s data.
- **isDirty()** – Checks if the model has unsaved changes.
- **isClean()** – Checks if the model has no unsaved changes.
- **wasRecentlyCreated()** – Checks if the model was recently created.

---

## Testing

### Step 1: Write Test Cases

To ensure the functionality of your package, you can write tests using PHPUnit. Here’s an example of a basic test for the `CassandraBaseModel`:

1. Create a test file at `tests/Feature/CassandraBaseModelTest.php`.

```php
<?php

namespace YourVendorName\CassandraEloquent\Tests\Feature;

use YourVendorName\CassandraEloquent\Models\CassandraBaseModel;
use PHPUnit\Framework\TestCase;

class CassandraBaseModelTest extends TestCase
{
    public function test_find_method()
    {
        $record = CassandraBaseModel::find(1);
        $this->assertNotNull($record);
        $this->assertEquals(1, $record['id']);
    }

    public function test_save_method()
    {
        $model = new CassandraBaseModel();
        $model->name = 'Test Model';
        $model->save();

        $this->assertEquals('Test Model', $model->name);
    }

    public function test_create_method()
    {
        $model = CassandraBaseModel::create([
            'name' => 'New Model',
            'email' => 'new.model@example.com',
        ]);

        $this->assertNotNull($model);
        $this->assertEquals('New Model', $model->name);
    }
}
```

2. Run your tests:

```bash
php artisan test
```

### Step 2: Mocking Cassandra for Tests

You can mock the Cassandra connection using Laravel’s built-in testing helpers if you don’t want to interact with a real Cassandra database in your tests. For more complex testing scenarios, consider using `Mockery` or PHPUnit’s mocking capabilities.

---

## License

This package is open-source and available under the [MIT License](LICENSE).

---

### Conclusion

With this package, you can use Cassandra in Laravel as if you're working with an Eloquent model. The package implements basic CRUD operations, query building, pagination, and other helper methods that make working with Cassandra data simple and elegant in a Laravel environment.

