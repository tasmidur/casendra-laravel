To organize the `CassandraBaseModel` in a clean and maintainable way, we should break it down into logical sections and separate concerns (methods for querying, saving, updating, deleting, etc.). This will help improve readability, scalability, and maintainability.

Below is an organized and well-structured version of the `CassandraBaseModel` using your requirements.

### Final Structure of `CassandraBaseModel`

```php
namespace App\Models;

use Cassandra\Cluster;
use Cassandra\SimpleStatement;
use Illuminate\Database\Eloquent\Model;

class CassandraBaseModel extends Model
{
    // Cassandra connection and table name
    protected $connection = 'cassandra';  // Cassandra connection name
    protected $primaryKey = 'id';         // Default primary key
    public $timestamps = false;           // Cassandra doesn't support timestamps

    // Querying Methods
    protected function query($query, $params = [])
    {
        $connection = app('Cassandra');
        $statement = new SimpleStatement($query);
        return $connection->execute($statement, $params);
    }

    // Core Eloquent Methods
    public static function all()
    {
        $model = new static();
        $table = $model->getTable();
        $query = "SELECT * FROM {$table}";
        return $model->query($query);
    }

    public static function find($id)
    {
        $model = new static();
        $table = $model->getTable();
        $query = "SELECT * FROM {$table} WHERE {$model->primaryKey} = ?";
        $result = $model->query($query, [$id]);
        return $result ? $result[0] : null;
    }

    public static function findOrFail($id)
    {
        $record = static::find($id);
        if (!$record) {
            throw new \Exception("Record not found for ID: {$id}");
        }
        return $record;
    }

    public static function first()
    {
        $model = new static();
        $table = $model->getTable();
        $query = "SELECT * FROM {$table} LIMIT 1";
        $result = $model->query($query);
        return $result ? $result[0] : null;
    }

    public static function firstOrFail()
    {
        $record = static::first();
        if (!$record) {
            throw new \Exception("No records found.");
        }
        return $record;
    }

    public static function create(array $attributes)
    {
        $model = new static();
        $columns = implode(', ', array_keys($attributes));
        $values = implode(', ', array_map(function ($value) {
            return "'" . addslashes($value) . "'";
        }, array_values($attributes)));
        
        $table = $model->getTable();
        $query = "INSERT INTO {$table} ({$columns}) VALUES ({$values})";
        $model->query($query);
        
        // Assuming 'id' is auto-incremented or handled by Cassandra
        return static::find($attributes[$model->primaryKey]);
    }

    public function update(array $attributes)
    {
        $setClause = implode(', ', array_map(function ($column, $value) {
            return "{$column} = '" . addslashes($value) . "'";
        }, array_keys($attributes), array_values($attributes)));

        $table = $this->getTable();
        $query = "UPDATE {$table} SET {$setClause} WHERE {$this->primaryKey} = ?";
        $this->query($query, [$this->{$this->primaryKey}]);
    }

    public function delete()
    {
        $table = $this->getTable();
        $query = "DELETE FROM {$table} WHERE {$this->primaryKey} = ?";
        $this->query($query, [$this->{$this->primaryKey}]);
    }

    public function save()
    {
        if (isset($this->{$this->primaryKey})) {
            return $this->update($this->attributes);
        } else {
            return $this->create($this->attributes);
        }
    }

    // Utility Methods
    public function getTable()
    {
        return strtolower(class_basename($this));  // Table name based on model name
    }

    public function getConnection()
    {
        return app('Cassandra');
    }

    public function getAttributes()
    {
        return $this->attributes;
    }

    // Query Builder Methods (Where, OrWhere, Limit, etc.)
    public function where($column, $operator, $value)
    {
        $this->whereClause = "{$column} {$operator} ?";
        $this->whereParams = [$value];
        return $this;
    }

    public function orWhere($column, $operator, $value)
    {
        $this->whereClause .= " OR {$column} {$operator} ?";
        $this->whereParams[] = $value;
        return $this;
    }

    public function orderBy($column, $direction = 'ASC')
    {
        $this->orderByClause = "ORDER BY {$column} {$direction}";
        return $this;
    }

    public function limit($value)
    {
        $this->limitClause = "LIMIT {$value}";
        return $this;
    }

    public function paginate($perPage = 10)
    {
        $query = "SELECT * FROM {$this->getTable()}";
        
        if (isset($this->whereClause)) {
            $query .= " WHERE {$this->whereClause}";
        }

        if (isset($this->orderByClause)) {
            $query .= " {$this->orderByClause}";
        }

        $query .= " {$this->limitClause}";

        return $this->query($query, $this->whereParams);
    }

    public function count()
    {
        $query = "SELECT COUNT(*) FROM {$this->getTable()}";
        
        if (isset($this->whereClause)) {
            $query .= " WHERE {$this->whereClause}";
        }

        $result = $this->query($query, $this->whereParams);
        return $result[0]['count'];
    }

    public function exists()
    {
        $query = "SELECT COUNT(*) FROM {$this->getTable()} WHERE {$this->primaryKey} = ?";
        $result = $this->query($query, [$this->{$this->primaryKey}]);
        return $result[0]['count'] > 0;
    }

    public function pluck($column)
    {
        $query = "SELECT {$column} FROM {$this->getTable()}";
        
        if (isset($this->whereClause)) {
            $query .= " WHERE {$this->whereClause}";
        }

        $results = $this->query($query, $this->whereParams);
        return array_column($results, $column);
    }

    // Other Utility Methods (toArray, toJson, etc.)
    public function toArray()
    {
        return $this->attributes;
    }

    public function toJson()
    {
        return json_encode($this->attributes);
    }

    public function fresh()
    {
        return static::find($this->{$this->primaryKey});
    }

    public function refresh()
    {
        $this->attributes = $this->fresh();
    }

    public function isDirty()
    {
        return $this->original != $this->attributes;
    }

    public function isClean()
    {
        return !$this->isDirty();
    }

    public function wasRecentlyCreated()
    {
        return isset($this->{$this->primaryKey}) && !$this->exists();
    }
}
```

---

### Explanation and Structure

#### 1. **Core Eloquent Methods**
   These methods include the basic functionality for retrieving, saving, updating, and deleting records, such as:
   - `all()`, `find()`, `findOrFail()`, `first()`, `create()`, `update()`, `delete()`, and `save()`.

#### 2. **Query Builder Methods**
   These methods implement the common query builder patterns:
   - `where()`, `orWhere()`, `orderBy()`, `limit()`, `paginate()`, `count()`, `exists()`, and `pluck()`.

#### 3. **Utility Methods**
   These methods help with interacting with the model's data, including:
   - `getAttributes()`, `getTable()`, `getConnection()`, `toArray()`, `toJson()`, `fresh()`, `refresh()`, `isDirty()`, `isClean()`, and `wasRecentlyCreated()`.

#### 4. **Additional Considerations**
   - Since Cassandra is a NoSQL database and does not support native relational features like **joins**, **foreign keys**, and **relationships**, handling relationships in Cassandra requires manual management (i.e., denormalization or using separate tables). Methods for `hasOne()`, `hasMany()`, etc., would need to be implemented manually based on your data structure and access patterns, such as maintaining references in different tables and fetching them separately.

---

### Usage Example:

```php
// Fetch all users
$users = User::all();

// Find user by ID
$user = User::find(1);

// Create a new user
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john.doe@example.com',
]);

// Update a user
$user->update(['name' => 'John Updated']);

// Delete a user
$user->delete();

// Paginate users
$paginatedUsers = User::where('status', '=', 'active')->paginate(10);
```

---

### Conclusion

This organized **CassandraBaseModel** provides a clean and maintainable structure for interacting with Cassandra using a Laravel Eloquent-like API. It includes core Eloquent methods, query builder methods, utility functions, and is flexible for customization based on the specific needs of your application (e.g., relationships, denormalization).
