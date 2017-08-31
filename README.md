# AlumnForce Coding Conventions

_WIP_

## Table of Contents
* [MySQL](#mysql)
  * [Schemas](#schemas)
  * [Queries](#queries)
  * [Patches](#patches)
* [PHP](#php)
 
## <a name="mysql"></a>MySQL

### <a name="schemas"></a>Schemas

* Naming:
  * table names in the singular
  * both table and column names in `snake_case`
  * PK: do not repeat the table's name, and typically call it simply `id`.
  * FK column's name MUST be named `<referenced-table>_<referenced-column>`, example: `user_id`.
  * Add `_id` suffix for int columns
  * Add `_ext_ref` suffix for external references columns, example: `diploma_ext_ref`
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

* Don't forget our 2 "system" columns managed by MySQL itself:
  ```sql
  created TIMESTAMP NOT NULL DEFAULT NOW(),
  updated TIMESTAMP NOT NULL DEFAULT NOW() ON UPDATE NOW(),
  ```
  These 2 columns should never be used in writing by application code!
  
### <a name="queries"></a>Queries

### <a name="patches"></a>Patches

## <a name="php"></a>PHP

We follow the standards defined in the
[PSR-0](http://www.php-fig.org/psr/psr-0/), 
[PSR-1](http://www.php-fig.org/psr/psr-1/) and 
[PSR-2](http://www.php-fig.org/psr/psr-2/) documents.
