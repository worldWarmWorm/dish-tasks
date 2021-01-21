1. В контроллер \protected\modules\admin\controllers\ShopController.php добавить в actionIndex \ actionCategory

$productDataProvider->sort->defaultOrder = '`t`.`id` desc';

перед методом

$this->render()