# AlumnForce Coding Conventions

_WIP_

## Table of Contents
* [MySQL](#mysql)
  * [Schemas](#schemas)
  * [Queries](#queries)
  * [Patches](#patches)
* [PHP](#php)
  * [General](#general)
  * [Composer](#composer)
  * [Notation](#notation)
  * [OOP](#oop)
  * [PHPDoc](#phpdoc)
  * [Practical cases](#practicalcases)



## <a name="mysql"></a>MySQL



### <a name="schemas"></a>Schemas

* Naming:
  * table names in the singular
  * both table and column names in `snake_case`
  * PK: do not repeat the table's name, and typically call it simply `id`.
  * FK column's name MUST be named `<referenced-table>_<referenced-column>`, example: `user_id`.
  * Add `_id` suffix for int columns
  * Add `_ext_ref` suffix for external references columns, example: `degree_ext_ref`
  * Boolean: `is_x TINYINT`

* No need of CONSTRAINT clause when defining a foreign key:
  ```sql
  FOREIGN KEY user_id_fk (user_id) REFERENCES user (id) ON DELETE RESTRICT ON UPDATE CASCADE
  ```
  
* To name an FK or an index (KEY), prefer this naming: `<column-name>_[idx|fk]`. 
Examples: `user_id_fk` or `col1_col2_idx`.

* By default, always set FK with `ON UPDATE CASCADE ON DELETE RESTRICT`.

* Don't forget `UNSIGNED` types: `user_id UNSIGNED INT`.

* Stop specifying a never used display width with integers' declaration:
`INT(1)` allows to store exactly same integers as `INT(11)`. 
So prefer: `user_id UNSIGNED INT`.

* Add a maximum of integrity constraints: `UNIQUE`, `FK`, `NOT NULL`…

* About text columns:
  * use `CHAR(x)` for fixed-length strings, else `VARCHAR(x)` for strings of length at most `x`
  * if no product justification, write `VARCHAR(255)` instead of any value < 255
  * prefer `VARCHAR(255)` to `VARCHAR()` because by default the latter is equivalent to `VARCHAR(65535)`
    
* Use COMMENT on columns and keep them up to date.

* Don't forget our 2 “system” columns managed by MySQL itself:
  ```sql
  created TIMESTAMP NOT NULL DEFAULT NOW(),
  updated TIMESTAMP NOT NULL DEFAULT NOW() ON UPDATE NOW(),
  ```
  These 2 columns should never be used in writing by application code!



### <a name="queries"></a>Queries

* All keywords in CAPITALS.

* Do not put backticks `` ` `` around a table or column name if not necessary, this complicates reading.

* Always specify which type of join: `LEFT JOIN`, `INNER JOIN`, other?

* Use table alias as soon as there is more than one table in the query, and prefix each column name with them.
Example: 
```sql
    SELECT U.id, D.name
    FROM user U
    INNER JOIN degree D ON D.id = U.degree_id
    WHERE ...
```

* Any SELECT must have a `LIMIT` clause, with the possible exception of exports.



### <a name="patches"></a>Patches

Because `IF EXISTS` doesn't exist for many [DDL](https://en.wikipedia.org/wiki/Data_definition_language) statements
we need to encapsulate some SQL instructions in a stored procedure.

Systematically make sure to remove the possible existing in order to reconstruct and guarantee the expected result:
for example make a `DROP INDEX` before the `CREATE INDEX` because perhaps it does not target the same columns.  

Approach adopted after **vote**:

```sql
    DROP PROCEDURE IF EXISTS clean;
    
    DELIMITER //
    CREATE PROCEDURE clean()
    BEGIN
        SET @exists = (
            SELECT count(*)
            FROM information_schema.statistics
            WHERE table_name = 'bounce' AND index_name = 'campaign_id' AND table_schema = database()
        );
        IF @exists > 0 THEN
            DROP INDEX campaign_id ON bounces;
        END IF;
    END //
    DELIMITER ;
    
    CALL clean();
    DROP PROCEDURE IF EXISTS clean;
    
    -- True payload:
    CREATE INDEX campaign_id ON bounce (campaign_id);
```



## <a name="php"></a>PHP



### <a name="general"></a>General

We follow the standards defined in the
[PSR-0](http://www.php-fig.org/psr/psr-0/), 
[PSR-1](http://www.php-fig.org/psr/psr-1/) and 
[PSR-2](http://www.php-fig.org/psr/psr-2/) documents.



### <a name="composer"></a>Composer

* Allways specify tag/version for each dependency into `composer.json`.
* The `composer.lock` must be commit.



### <a name="notation"></a>Notation

* Use camelCase. Acronyms have only their first letter in capital: `getPsrLogger()`, `$mySqlQuery`.
* Always use `[]` notation for arrays.
* Place unary operators adjacent to the affected variable, 
with the excpetion of negation operator and casting:
`$a++`, `$a--`, `if (! $expr) {…}`, `$foo = (int) $bar;`
* Add a single space around binary operators, 
including concatenation operator: `$a && $b`, `$a . $b`



### <a name="oop"></a>OOP

* By default every class property and every instance property must be private.
* Do not use magical methods (or prove that there was no alternative).
* All function calls must have parenthesis, even constructor: `new MyClass()`.



### <a name="phpdoc"></a>PHPDoc

* Always document structure of array parameters and array returns.
* Avoid ~~@param array~~, be as accurate as possible. Examples:
`@param int[] <description> <structure>`,
`@param ThisClass[] <description> <structure>`.
* These annotations are forbidden because without any added value:
  * ~~@abstract~~
  * ~~@access public~~
  * ~~@access protected~~
  * ~~@access private~~
  * ~~@internal~~
  * ~~@static~~



### <a name="practicalcases"></a>Practical cases

#### On-the-fly assignment

On-the-fly assignment must be avoided because not very readable:
~~if (($myId = $obj->getById())) {...}~~.

Write:
```php
    $myId = $obj->getById();
    if ($myId !== null) {
        // ...
    }
```

#### Implicit if

Implicit `if` must be avoided because not very readable:
* ~~\<expression> and \<statement>;~~
* ~~\<expression> && \<statement>;~~

Write:
```php
    if (<expression>) {
        <statement>
    }
```

#### Approximate test

Let: ~~if (\<expression> === true) {...}~~,
if `<expression>` is of boolean type, then simply write: `if (<expression>) {…}`.

If `<expression>` is not a boolean then be specific, _e.g._ `if (preg_match(...) > 0) {...}`

#### Switch hack

To avoid: ~~switch (true) {...}~~

If `case` expressions are not deductible from the same variable, then use `if/elseif`.

#### Remote return

Instead of:
```php
    if ($expression) {
        return $foo;
    }

    // many lines...

    return $bar;
```

Write:
```php
    if ($expression) {
        return $foo;
    } else {
        // many lines...
        return $bar;
    }
```

Or better, only one `return` if possible:
```php
    if ($expression) {
        $value = $foo;
    } else {
        // many lines...
        $value = $bar;
    }
    
    return $value;
```

#### Mixed return types

For a given function, do not use more than one data type for return value.

Bad, because of the mixture between `array` and `int`:
```php
    /**
     * @return array|int
     */
    function (…) {
        if (…) {
            return [1, 2, 3];
        } else {
            return 5;
        }
    }
```

Rewrite as follows:
```php
    /**
     * @return array
     */
    function (…) {
        if (…) {
            return [1, 2, 3];
        } else {
            return [5];
        }
    }
```

* Same for the `string|null` case.
* The empty object being `null`, it's ok with `MyClass|null` case.
* The management of unexpected errors must be done via exceptions.

#### Emptyness

Redundant: `if (! isset($params['foo']) || empty($params['foo'])) {…}`.

This is equivalent to: `if (empty($params['foo'])) {…}`.

#### Variable name differentiation

Not very readable: `foreach ($items as $item) {…}`.

For example, improve in this way: `foreach ($allItems as $item) {…}`.
