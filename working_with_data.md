# Working with Data

## The fastest and riskier way use Db::getInstance();

In your node, you'll nee to declare

```php
 use Sintattica\Atk\Db\Db;
```

Then you can obtain a database instance with:

```php
$db = Db::getInstance();
```

The Database object **$db** have an method called **getRows** that returns an array of rows i.e.:

```php
use Sintattica\Atk\Db\Db;

class MyNode extends Node
{

    function myaction_action()
    {
          $db = Db::getInstance();
          $query = "SELECT * FROM TABLE";    
          $result = $db->getRows($query);
          foreach($result as $row)
          {
              //Do Something with $row here
          }
     }
}
```

This way you can access the database, BUT, be aware of sanitize any input if you are building the query string inb order
to avoid SQL INJECTION i.e. DO NOT do this:

```php
$unsafe_variable = $_POST['user_input']; 
$query = "SELECT * FROM TABLE WHERE field = ". $unsafe_variable;    
$result = $db->getRows($query);
```

Do at least (if using mysql):

```php
$safe_variable = Mysql::escape_string($_POST['user_input']); 
$query = "SELECT * FROM TABLE WHERE field = ". $safe_variable;    
$result = $db->getRows($query);
```
The rationale of the previous precaution is becasue if the user_input variable contains **'Sarah'; DELETE FROM employees** the result would 
simply be a search for the string "'Sarah'; DELETE FROM employees", and you will not end up with an empty table.

Having said that, do not use **Db::getInstance()** Unless you are really sure about the query string  (i.e. a "fixed" query select to recover a "list" from a table)

## Recovering the Rows for your node

In your node you can recover the records with the node method **select** i.e.:

```php
use Sintattica\Atk\Db\Db;

class MyNode extends Node
{
    function myaction_action()
    {
          $rows = $this->select()->getAllRows();
          print_r($rows):
          die();          
    }
}
```

The method select can receive a filter parameter, i.e.:

```php
class MyNode extends Node
{
    function myaction_action()
    {
          $filter = "sex ='M'";
          $rows = $this->select($fiter)->getAllRows();
          print_r($rows):
          die();          
    }
}
```

You can recover just the first row with: 

```php
$rows = $this->select($fiter)->getFirsrRow();
```

If you need to recover records based on an "unsafe" entry you can use the **where** method:

```php
class MyNode extends Node
{
    function myaction_action()
    {
          $filter = "sex ='M'";
          $rows = $this->select()
                       ->where ('sex = ? ', ['M'])
                       ->getAllRows();
          print_r($rows):
          die();          
    }
}
```

Note that the question mark signals where the parameter will be replaced, and that the parameteres are sent into an array, thus

```php
 ->where ('sex = ? ', ['M']) //Correct
 ->where ('sex = ? ', 'M' )  //Incorrect                       
```

"Named" parameters can be used to, and these improves the readability of your code a lot, i.e:


```php
class MyNode extends Node
{
    function myaction_action()
    {
          $filter = "sex ='M'";
          $rows = $this->select()
                       ->where ('sex = :customer_sex and age >= :customer_age ', ['customer_sex'=>'M','customer_age' => 40])
                       ->getAllRows();
          print_r($rows):
          die();          
    }
}
```

## The query builder 
...
