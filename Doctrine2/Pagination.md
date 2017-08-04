## Pagination

```code
<?php
require_once __DIR__."/vendor/bootstrap.php";

use Doctrine\ORM\Tools\Pagination\Paginator;

$dql = "SELECT b, e, r, p FROM Bug b JOIN b.engineer e ".
       "JOIN b.reporter r JOIN b.products p ORDER BY b.created DESC";
$query = $entityManager->createQuery($dql)
               ->setFirstResult(0)
               ->setMaxResults(1);

$paginator = new Paginator($query, $fetchJoinCollection = true);

$count = count($paginator);         // output total record
$bugs = $query->getArrayResult();   // output array result

foreach ($bugs as $item) {
    print_r($item);
}
```
The Paginator has implements the SPL interfaces <code>Countable</code> and <code>IteratorAggregate</code>. so, you can use the <code>foreach()</code> method directly.

Maybe
```code
foreach ($paginator as $bug) {
    print_r($bug);
}
```

## Reference

* [Pagination](http://docs.doctrine-project.org/en/latest/tutorials/pagination.html)
