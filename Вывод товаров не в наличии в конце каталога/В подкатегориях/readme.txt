1. \protected\models\Product.php
Создать переменную $order='IF(`t`.`notexist`=1, 1, 0) ASC, ' . $criteria->order;