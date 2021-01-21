1. В \protected\models\Product.php закомментировать строка 398

// $modelProduct=new Product;
        // $criteria->mergeWith($modelProduct->orderById()->getDbCriteria());
        // $modelProduct=new Product;
        // $criteria->mergeWith($modelProduct->categoryOnTop($categoryId)->getDbCriteria());
        // $modelProduct=new Product;
        // $criteria->mergeWith($modelProduct->scopeSort('shop_category', $categoryId)->getDbCriteria());
        // $modelProduct=new Product;
        // $criteria->mergeWith($modelProduct->hitOnTop()->getDbCriteria());
		// $order=$criteria->order;

2. строка 413 заменить

return new \CActiveDataProvider($this, [
            'criteria'=>$criteria,
            'pagination'=>[
                'pageSize' =>$pageSize,
                'pageVar'=>'p'
            ],
            'sort'=>[
                'sortVar'=>'s', 
                'descTag'=>'d',
                'defaultOrder'=> $order,
            ],
        ]);

на 

return new \CActiveDataProvider($this, [
            'criteria'=>$criteria,
            'pagination'=>[
                'pageSize' =>$pageSize,
                'pageVar'=>'p'
            ],
            'sort'=>[
                'sortVar'=>'s', 
                'descTag'=>'d',
                'defaultOrder'=>'t.`id` desc' // $order,
            ],
        ]);