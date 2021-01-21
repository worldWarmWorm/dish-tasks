1. Проверить актуальность resizehelper, если старая версия, поменять на новую.

<?php
/**
 * Класс помощник для создания миниатюр изображений
 * 
 */
class ResizeHelper
{
    const RESIZED_PATH='webroot.uploads.resized';
    const DEFAULT_WATERMARK='/images/watermark.png';

    /**
     * Resize image
     * @static
     * @param $sourceFileName
     * @param $width
     * @param $height
     * @param $top
     * @param $resize
     * @return mixed
     */
    public static function resize($sourceFileName, $width=0, $height=0, $top=false, $resize=false)
    {
        if($sourceFileName) {
            $sourceFileName=static::normalizeFileName($sourceFileName);
            $tmbFileName=static::getTmbFileName($sourceFileName, $width, $height);
            if(!file_exists($tmbFileName)) {
                $ih=new CImageHandler();
                $ih->load($sourceFileName);
                if(!$width || !$height || $resize) {
                    $ih->resize($width, $height);
                } else {
                    $ih->adaptiveThumb($width, $height, $top);
                }
                try {
                    static::createPath(dirname($tmbFileName));
                    $ih->save($tmbFileName, false, 95);
                }
                catch(\Exception $e) {
                    return static::getRelativeFileName($sourceFileName);
                }
            }

            return static::getRelativeFileName($tmbFileName);
        }

        return null;
    }

    /**
     * Resize image
     * @static
     * @param $sourceFileName
     * @param $zoom
     * @param $force
     * @param $watermark
     * @return mixed
     */
    public static function watermark($sourceFileName, $zoom=false, $force=false, $watermark=null)
    {
        if($sourceFileName) {
            $sourceFileName=static::normalizeFileName($sourceFileName);
            $waterFileName=static::getTmbFileName($sourceFileName, 0, 0, 'water_');
            if(!file_exists($waterFileName) || $force) {
                $ih=new CImageHandler();
                $ih->load($sourceFileName);

                $watermark=static::normalizeFileName($watermark ?: self::DEFAULT_WATERMARK);
                $ih->watermark($watermark, 0, 0, CImageHandler::CORNER_CENTER, $zoom);

                static::createPath(dirname($waterFileName));
                $ih->save($waterFileName, false, 95);
            }

            return static::getRelativeFileName($waterFileName);
        }

        return null;
    }

	/**
     * Resize with watermark
     */
    public static function wresize($sourceFileName, $width=null, $height=null, $zoom=0.75, $force=true)
    {
        return static::watermark(static::resize($sourceFileName, $width, $height), $zoom, $force);
    }

    public static function watermarkModifier($sourceFileName, $force=false, $watermark=null)
    {
        $originalPath=static::getRelativeFileName($sourceFileName);
        if($sourceFileName) {
            $sourceFileName=static::normalizeFileName($sourceFileName);
            $waterFileName=static::getTmbFileName($sourceFileName, 0, 0, 'w2_');
            $watermark=static::normalizeFileName($watermark ?: self::DEFAULT_WATERMARK);
            if(!file_exists($waterFileName) || $force) {
                if (!is_file($watermark) || !is_file($sourceFileName)) {
                    return $originalPath;
                }

                if(!\Yii::app()->hasComponent('imagemod')) {
                    \Yii::app()->setComponent('imagemod', [
                        'class'=>'application.extensions.imagemodifier.CImageModifier'
                    ]);
                }

                if(copy($sourceFileName, $waterFileName)) {
                    $image=\Yii::app()->imagemod->load($waterFileName);
                    $image->image_watermark = $watermark;
                    $image->image_watermark_position = 'CC';
                    $image->jpeg_quality = 100;
                    $image->file_new_name_body = $image->file_src_name_body;
                    $image->file_overwrite = true;
                    $image->image_watermark_no_zoom_in=false;
                    $image->image_watermark_no_zoom_out=false;
        
                    static::createPath(dirname($waterFileName));
                    $image->process(dirname($waterFileName));
                }
                else {
                    return $originalPath;
                }
            }

            return static::getRelativeFileName($waterFileName);
        }
    }

    protected static function normalizeFileName($filename)
    {
        if (strpos($filename, "?")) {
            $filename=substr($filename, 0, strpos($filename, "?"));
        }

        if(strpos($filename, static::getWebRoot()) !== 0) {
	        $filename=rtrim(static::getWebRoot(), '/\\\\') . '/' . $filename;
        }

        return static::normalizePath($filename);
    }

    protected static function getRelativePath($filename)
    {
        $path=dirname($filename);
        if(strpos($path, static::getWebRoot()) === 0) {
	        $relativePath='/' . trim(substr($path, strlen(static::getWebRoot())), '/\\\\');
        }
        else {
            $relativePath='/' . trim($path, '/\\\\');
        }

        return static::normalizeRelativePath($relativePath);
    }

    protected static function getRelativeFileName($filename)
    {
        return static::normalizeRelativePath(static::getRelativePath($filename) . '/' . basename($filename));
    }

    protected static function getTmbFileName($filename, $width=null, $height=null, $prefix='')
    {
        $width=(int)$width;
        $height=(int)$height;
        $filename=static::normalizeFileName($filename);
        $sourcePath=dirname($filename);
        $sourceBasename=basename($filename); 
        $relativePath=static::getRelativePath($filename);
        $salt=hash_file('md5', $filename);
        $tmbBasename=$prefix.$salt . (($width || $height) ? "_{$width}_{$height}" : '') . "_{$sourceBasename}";

        return static::normalizeRecusivePath(static::normalizePath(static::getBasePath() . $relativePath . '/' . $tmbBasename));
    }

    protected static function getBasePath()
    {
        static $basePath=null;
        
        if(!$basePath) {
            $basePath=static::normalizePath(\Yii::getPathOfAlias(self::RESIZED_PATH));
            static::createPath($basePath);
        }
        
        return $basePath;
    }

    protected static function createPath($path)
    {
        if(!is_dir($path)) {
			mkdir($path, 0755, true);
            chmod($path, 0755);
        }
    }

    protected static function getWebRoot()
    {
        static $webroot=null;

        if(!$webroot) {
            $webroot=static::normalizePath(\Yii::getPathOfAlias('webroot'));
        }

        return $webroot;
    }

    protected static function normalizePath($path)
    {
        return preg_replace('#[/\\\\]+#', DIRECTORY_SEPARATOR, $path);
    }

    protected static function normalizeRelativePath($path)
    {
        return preg_replace('#[/\\\\]+#', '/', $path);
    }

    protected static function normalizeRecusivePath($filename)
    {
        $relativeBasePath=static::normalizePath(static::getRelativeFileName(static::getBasePath()));
        return preg_replace('#(' . preg_quote($relativeBasePath) . ')+#', $relativeBasePath, $filename);
    }
}


===================================

2. Заменить 

public function getPath()
	{
		return \Yii::getPathOfAlias('webroot') . Y::DS . $this->filePath .  Y::DS . $this->owner->id;
	}

на

public function getPath()
	{
		return \Yii::getPathOfAlias('webroot') . Y::DS . $this->filePath .  Y::DS . $this->owner->id;
	}
====================================

3. Заменить в функции public function beforeSave()

$this->owner->{$this->attribute}=$this->getBasename() . '.' . strtolower($file->getExtensionName());

на 

$this->owner->{$this->attribute}=$file->getName();

затем

HFile::mkDir($this->getPath(), '0755', true);

на

HFile::mkDir($this->getPath(), 0755, true);

=====================================

4. Заменить в функции public function afterSave()

$this->owner->{$this->attribute}=$this->getBasename() . '.' . strtolower($file->getExtensionName());

на

$this->owner->{$this->attribute}=$file->getName();

затем

HFile::mkDir($this->getPath(), '0755', true);

на

HFile::mkDir($this->getPath(), 0755, true);

======================================

5. В функции public function getTmbSrc заменить

$src=preg_replace('#[/\\\\]+#', '/', $this->filePath) . '/' . $tmbFilename;

на

$src=preg_replace('#[/\\\\]+#', '/', $this->filePath) . '/' . $this->owner->id . '/' . $tmbFilename;

=======================================

6. В функции public function getSrc заменить

$src=preg_replace('#[/\\\\]+#', '/', $this->filePath) . '/' . $this->getFilename();

на

$src=preg_replace('#[/\\\\]+#', '/', $this->filePath) . '/' . $this->owner->id . '/' . $tmbFilename;

=======================================

7. В функции public function getTmbSrc заменить

$isReturnOrigin=((int)$width < 0) || ((int)$height < 0);
			if($isReturnOrigin) {
				$tmbFilename=$this->tmbPrefix . '_' . (((int)$width < 0) ? '_' : $width) . 'x' . (((int)$height < 0) ? '_' : $height) . '_' . $this->getFilename();				
			}
			else {
				$tmbFilename=$this->tmbPrefix . '_' . (($width === false) ? 'n' : $width) . 'x' . (($height === false) ? 'n' : $height) . '_' . $this->getFilename();
            }
            
            if(!is_file($this->getPath() . Y::DS . $tmbFilename) || YII_DEBUG) {
				
                $image=\Yii::app()->ih->load($this->getFilename(true));
                if(!$isReturnOrigin) {
                    if($adaptive) {
                        $image=$image->adaptiveThumb($width, $height);
                    }
                    else {
                        $image=$image->thumb($width, $height, $proportional);
                    }
                }
                $image->save($this->getPath() . Y::DS . $tmbFilename);                
            }
			
			$src=preg_replace('#[/\\\\]+#', '/', $this->filePath) . '/' . $this->owner->id . '/' . $tmbFilename;
			if($absolute) {
				return \Yii::app()->createAbsoluteUrl($src, [], $schema);
			}
			return '/' . $src;

на

return \ResizeHelper::resize($this->getFilename(true), $width, $height, false, true);



После всех изменений в этой папке /uploads/resized/images/product/97 будет храниться картинка, загруженная в в карточку товара. 97 - индекс фотографии и название папки, что дает уникальность пути. Такие папки будут создаваться в папке /product. При загрузке картинке будет сохраняться оригинальное имя файла.

Пример вывода фотографии с оригинальным именем:

<div style="display:none">
		<div id="popup-<?=$data->id?>" class="product-popup">
			<div class="more-images">
					<div class="more-images-item">
						<img src="<?=$data->getSrc()?>" alt="">
						<div class="lg"><?= $data->main_image ?></div>
					</div>
					<?foreach($data->moreImages as $id=>$img):?>
					<div class="more-images-item">
						<img src="<?=$img->url?>" alt="">
					</div>
					<?endforeach?>
				</div>
		</div>
	</div>



<?= $data->main_image ?> - здесь хранится оригинальное имя

Такое поведение будет на всем сайте, т.е. при загрузке файлов в админке.