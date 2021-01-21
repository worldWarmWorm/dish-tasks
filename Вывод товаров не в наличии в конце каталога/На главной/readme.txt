1. \protected\controllers\ShopController.php

в actionIndex добавить $dataProvider->criteria->order='IF(`t`.`notexist`=1,1,0) '; после объявления переменной $dataProvider (строка 31)