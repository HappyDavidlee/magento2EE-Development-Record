# Magento2EE功能修改之报表去掉按钮
新建`app/code/CustomBackend/OrderviewButton/Component`目录
新建ExportButton.php

```
<?php
namespace CustomBackend\OrderviewButton\Component;

class ExportButton extends \Magento\Ui\Component\ExportButton
{
    /**
     * @return void
     */
    public function prepare()
    {
        $context = $this->getContext();
        $config = $this->getData('config');
        if (isset($config['options'])) {
            $options = [];
            foreach ($config['options'] as $option) {
                if($option['value'] != 'xml') {
                    $additionalParams = $this->getAdditionalParams($config, $context);
                    $option['url'] = $this->urlBuilder->getUrl($option['url'], $additionalParams);
                    $options[] = $option;
                }
            }
            $config['options'] = $options;
            $this->setData('config', $config);
        }
        parent::prepare();
    }

}
```
后台展示效果
![](images/docs/Magento2EE-Button.jpg)
