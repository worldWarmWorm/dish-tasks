1. В модели формы добавить поведение и класс A (вначале файла)


use common\components\helpers\HArray as A;

public function behaviors()
	{
		return A::m(parent::behaviors(), [
			'recaptchaBehavior'=>[
				'class'=>'\common\behaviors\ReCaptcha3Behavior',
				'on'=>'insert'
			]
		]);
	}
=====================================================================

2. Скопировать файл поведения ReCaptcha3Behavior.php в protected/modules/common/behaviors

<?php
/**
 * Минимальное подключение проверки Google ReCaptcha v3 для формы:
 * 
 * 1) Подключить в модель поведение 
 * 'recaptchaBehavior'=>[
 *      'class'=>'\common\behaviors\ReCaptcha3Behavior',
 *      'siteKey'=>'<ключ сайта>', // необязательно, если получается из настроек сайта
 *      'secretKey'=>'<секретный ключ>' // необязательно, если получается из настроек сайта
 *  ]
 * 
 * 2) Примеры подключения для формы в методах attachReCaptchaBy.
 */
namespace common\behaviors;

use common\components\helpers\HYii as Y;
use common\components\helpers\HArray as A;
use common\components\helpers\HRequest as R;
use common\components\helpers\HHash;

/**
 * Валидатор Google ReCaptcha v3
 */
class ReCaptcha3Validator extends \CValidator
{
    public $secretKey;
    public $score=0.9;
    public $recaptchaName='g-recaptcha-v3';
    public $errorAttribute='captcha';
    public $errorMessage='Не пройдена проверка';

    /**
     * Валидация Google ReCaptcha v3
     */
    public function validateAttribute($object, $attribute)
    {
        $hasError=true;

        if($token=R::post($this->recaptchaName, R::get($this->recaptchaName))) {
            $url='https://www.google.com/recaptcha/api/siteverify?' . http_build_query([
                'secret'=>$this->secretKey,
                'response'=>$token,
                'remoteip'=>$_SERVER['REMOTE_ADDR']
            ]);

            $ch=curl_init($url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
            curl_setopt($ch, CURLOPT_HEADER, 0);
            $response=json_decode(curl_exec($ch), true);
            curl_close($ch);

            if(!empty($response['success']) && !empty($response['score'])) {
                if((float)$response['score'] >= $this->score) {
                    $hasError=false;
                }
            }
        }

        if($hasError) {
            $object->addError($this->errorAttribute, $this->errorMessage);
        }
    }
}

/**
 * Поведение Google ReCaptcha v3
 * 
 * Наследоватся поведение должно для моделей
 * \common\components\base\FormModel или \common\components\base\ActiveRecord.
 * 
 * Подключение поведения
 * 'recaptchaBehavior'=>[
 *      'class'=>'\common\behaviors\ReCaptcha3Behavior',
 *      'siteKey'=>'<ключ сайта>', // необязательно, если получается из настроек сайта
 *      'secretKey'=>'<секретный ключ>' // необязательно, если получается из настроек сайта
 *  ]
 */
class ReCaptcha3Behavior extends \CBehavior
{    
    public $attribute='captcha';
    public $errorAttribute='captcha';
    public $errorMessage='Не пройдена проверка';
    
    public $siteKey=null;
    public $secretKey=null;
    public $score=null;

    public $cmsSiteKey='recaptcha3_sitekey';
    public $cmsSecretKey='recaptcha3_secretkey';
    public $cmsScore='recaptcha3_score';

    public $on=null;
    public $except=null;
    
    public $recaptchaName='g-recaptcha-v3';

    /**
     * Получить публичный ключ
     * @return string
     */
    public function getSiteKey()
    {
        if(!$this->siteKey && $this->cmsSiteKey) {
            return Y::param($this->cmsSiteKey, \D::cms($this->cmsSiteKey));
        }
        
        return $this->siteKey;
    }

    /**
     * Получить секретный ключ
     * @return string
     */
    public function getSecretKey()
    {
        if(!$this->secretKey && $this->cmsSecretKey) {
            return Y::param($this->cmsSecretKey, \D::cms($this->cmsSecretKey));
        }

        return $this->secretKey;
    }

    /**
     * Получить порог оценки
     * @return float 
     */
    public function getScore()
    {
        if(!$this->score && $this->cmsScore) {
            return Y::param($this->cmsScore, \D::cms($this->cmsScore, 0.9));
        }

        return $this->score;
    }
    
    /**
     * Подключение правил валидации Google ReCaptcha v3
     */
    public function rules()
    {
    	$rules=[];
    	
    	if($secretKey=$this->getSecretKey()) {
	        $rule=[
    	        'captcha', 
        	    '\common\behaviors\ReCaptcha3Validator', 
	            'secretKey'=>$secretKey,
    	        'score'=>$this->getScore(),
        	    'recaptchaName'=>$this->recaptchaName,
	            'errorAttribute'=>$this->errorAttribute,
    	        'errorMessage'=>$this->errorMessage
        	];

	        if($this->on) {
    	        $rule['on']=$this->on;
        	}

	        if($this->except) {
    	        $rule['except']=$this->except;
	        }

    	    $rules[]=$rule;
    	}

		return $rules;
    }

    /**
     * Регистрация подключения Google ReCaptcha3 API
     * @return bool
     */
    protected function registerReCaptcha()
    {
        if($siteKey=$this->getSiteKey()) {
            Y::js('recaptcha3_api', 'window.recaptcha3ScriptLoaded=false;$(document).on("scroll mousemove",function(){'
                . 'if(!window.recaptcha3ScriptLoaded){window.recaptcha3ScriptLoaded=true;s=document.createElement("script");'
                . 's.setAttribute("src","https://www.google.com/recaptcha/api.js?render=' . $siteKey . '");'
                . '$("body").append(s);}});', \CClientScript::POS_READY);
            
            // Y::jsFile('https://www.google.com/recaptcha/api.js?render=' . $siteKey);
            
            return true;
        }
        
        return false;        
    }

    /**
     * Сжать код скрипта
     * @param string $jsCode код скрипта
     * @return string
     */
    protected function minifyJsCode($jsCode)
    {
        return preg_replace('/\s+/', ' ', str_replace(["\n", "\r"], '', $jsCode));
    }

    /**
     * Добавить проверку Google ReCaptcha v3 на кнопку отправки формы.
     * Пример подключения:
     * <?= \CHtml::submit('Отправить', ['id'=>$model->recaptchaBehavior->attachReCaptchaBySubmitButton()]); ?>
     *
     * @param string $buttonId идентификатор кнопки отправки формы.
     * Может быть не передан, тогда идентификатор будет сгенерирован 
     * авториматически и возвращен в результате.
     * @return string идентификатор кнопки отправки формы.
     */
    public function attachReCaptchaBySubmitButton($buttonId=null)
    {
        if(!$buttonId) {
            $buttonId=HHash::u('btn');
        }
        
        if($this->registerReCaptcha()) {
            $jsCode=<<<EOL
            $(document).on("click", "#{$buttonId}", function(e){
                let form=$(e.target).parents("form:first");
                if(!form.find("[name='{$this->recaptchaName}']").length) {
                    form.append('<input type="hidden" name="{$this->recaptchaName}" />');
                }
                if(!form.find("[name='{$this->recaptchaName}']").val()) {
                    e.preventDefault();
                    grecaptcha.ready(function() {
                        grecaptcha.execute("{$this->getSiteKey()}", {action: "submit"}).then(function(token) {
                            form.find("[name='{$this->recaptchaName}']").val(token);
                            $('#{$buttonId}').trigger('click');
                        });
                    });
                    return false;
                }
            });
EOL;
            Y::js('recaptchav3_' . $buttonId, $this->minifyJsCode($jsCode), \CClientScript::POS_READY);
        }

        return $buttonId;
    }

    /**
     * Добавить проверку Google ReCaptcha v3 на отправку формы.
     * Пример подключения:
     * <? $model->recaptchaBehavior->attachReCaptchaBySubmit('#my-form'); ?>
     *
     * @param string $jFormSelector jQuery условие выборки формы, 
     * для которой подключается каптча.
     */
    public function attachReCaptchaBySubmit($jFormSelector)
    {
        if($this->registerReCaptcha()) {
            $jsCode=<<<EOL
            $(document).on("submit", "{$jFormSelector}", function(e){
                let form=$(e.target);
                if(!form.find("[name='{$this->recaptchaName}']").length) {
                    form.append('<input type="hidden" name="{$this->recaptchaName}" />');
                }
                if(!form.find("[name='{$this->recaptchaName}']").val()) {
                    e.preventDefault();
                    grecaptcha.ready(function() {
                        grecaptcha.execute("{$this->getSiteKey()}", {action: "submit"}).then(function(token) {
                            form.find("[name='{$this->recaptchaName}']").val(token);
                            $('{$jFormSelector}').trigger('submit');
                        });
                    });
                    return false;
                }
            });
EOL;
            Y::js('recaptchav3_' . md5($jFormSelector), $this->minifyJsCode($jsCode), \CClientScript::POS_READY);
        }
    }

    /**
     * Добавить проверку Google ReCaptcha v3 как функцию. 
     * Пример подключения:
     * <?= $model->recaptchaBehavior->attachReCaptchaByFunction('#my-form', '$("#my-form").submit();'); ?>
     * 
     * @param string $jFormSelector jQuery условие выборки формы, 
     * для которой подключается каптча.
     * @param string $successCallback javascript код, который будет 
     * выполнен после успешной установки токена каптчи.
     * @param string $functionName имя функции. Если не задано,
     * будет сгенерировано авториматически.
     * @return string javascript код вызова функции проверки
     */
    public function attachReCaptchaByFunction($jFormSelector, $successCallback=null, $functionName=null)
    {
        if($this->registerReCaptcha()) {
            if(!$functionName) {
                $functionName=HHash::u('func');
            }

            $jsCode=<<<EOL
            ;window.{$functionName}=function(){
                let form=$("{$jFormSelector}");
                if(!form.find("[name='{$this->recaptchaName}']").length) {
                    form.append('<input type="hidden" name="{$this->recaptchaName}" />');
                }
                if(!form.find("[name='{$this->recaptchaName}']").val()) {
                    grecaptcha.ready(function() {
                        grecaptcha.execute("{$this->getSiteKey()}", {action: "submit"}).then(function(token) {
                            form.find("[name='{$this->recaptchaName}']").val(token);
                            window.{$functionName}();
                        });
                    });
                    return false;
                }
                else {
                    {$successCallback}
                }
            };
EOL;
            Y::js('recaptchav3_' . md5($functionName), $this->minifyJsCode($jsCode), \CClientScript::POS_READY);

            return "window.{$functionName}();";
        }
        
        return '';
    }
}

================================================================================
3. В protected/config/params.php дабавить параметры

'recaptcha3_secretkey'=>'6LdhpuEZAAAAABWS7AdALeJBDpCi94e612tA2lI-',
    'recaptcha3_sitekey'=>'6LdhpuEZAAAAADjzFhCWBPu47ziuMqnAXMabFyfR',
    'recaptcha3_score'=>0.9

================================================================================
4. Вывод кнопки отправить в форме 

<?php echo CHtml::submitButton($factory->getOption('controls.send.title', 'Отправить'), array(
				'class'=>'callback-form__submit btn shine__link',
				'id'=>$factory->getModelFactory()->getModel()->recaptchaBehavior->attachReCaptchaBySubmitButton()
			)); ?>

--------------------

Для теста работы капчи можно изменить один символ в публичном ключе и по идее кнопка будет неактивной