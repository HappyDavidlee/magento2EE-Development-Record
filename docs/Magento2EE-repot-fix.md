# Magento2EE功能修改之报表导出自定修改

1. 首先在项目目录下新建文件夹
`/app/code/CustomBackend/OrderviewButton/view/adminhtml/ui_component`
并复制`sales_order_invoice_grid.xml`文件到此目录下。

2. 找到exportButton并修改为：
`<exportButton name="export_button" class="CustomBackend\OrderviewButton\Component\ExportButton">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="options" xsi:type="array">
                        <item name="csv" xsi:type="array">
                            <item name="value" xsi:type="string">csv</item>
                            <item name="label" xsi:type="string" translate="true">Details Report</item>
                            <item name="url" xsi:type="string">custombackend_orderviewbutton/export/gridToXlsDetails</item>
                        </item>
                        <item name="xls" xsi:type="array">
                            <item name="value" xsi:type="string">xls</item>
                            <item name="label" xsi:type="string" translate="true">Headers Report</item>
                            <item name="url" xsi:type="string">custombackend_orderviewbutton/export/gridToXls</item>
                        </item>
                    </item>
                </item>
            </argument>
        </exportButton>`
        
1. 在`app/code/CustomBackend/OrderviewButton/etc` 下修改di.xml。添加一下代码
`    <type name="CustomBackend\OrderviewButton\Model\Export\ConvertToXlsDetails">
        <arguments>
            <argument name="metadataProvider" xsi:type="object">CustomBackend\OrderviewButton\Model\Export\MetadataProvider</argument>
        </arguments>
    </type>
    <type name="CustomBackend\OrderviewButton\Model\Export\ConvertToXls">
        <arguments>
            <argument name="metadataProviderXls" xsi:type="object">CustomBackend\OrderviewButton\Model\Export\MetadataProviderXls</argument>
        </arguments>
    </type>`
3. 创建`app/code/CustomBackend/OrderviewButton/Model/Export`目录。新建ConvertToXls.php和MetadataProvider.php文件。
##### MetadataProvider.php 文件内容

```
<?php

namespace CustomBackend\OrderviewButton\Model\Export;


use Magento\Framework\Api\Search\DocumentInterface;
use Magento\Framework\View\Element\UiComponentInterface;

class MetadataProvider extends \Magento\Ui\Model\Export\MetadataProvider
{

    /**
     * Returns columns list
     *
     * @param UiComponentInterface $component
     * @return UiComponentInterface[]
     */
    protected function getColumns(UiComponentInterface $component)
    {
        if (!isset($this->columns[$component->getName()])) {
            $columns = $this->getColumnsComponent($component);
            foreach ($columns->getChildComponents() as $column) {
                if ($column->getData('config/label') && $column->getData('config/dataType') !== 'actions') {
                    $this->columns[$component->getName()][$column->getName()] = $column;
                }
            }
        }
        unset($this->columns[$component->getName()]["customer_email"]);
        unset($this->columns[$component->getName()]["billing_name"]);
        unset($this->columns[$component->getName()]["base_grand_total"]);
        unset($this->columns[$component->getName()]["shipping_address"]);
        unset($this->columns[$component->getName()]["billing_address"]);
        unset($this->columns[$component->getName()]["grand_total"]);
        unset($this->columns[$component->getName()]["customer_group_id"]);
        unset($this->columns[$component->getName()]["Customer Group"]);
        unset($this->columns[$component->getName()]["payment_method"]);
        unset($this->columns[$component->getName()]["shipping_information"]);
        unset($this->columns[$component->getName()]["subtotal"]);
        unset($this->columns[$component->getName()]["shipping_and_handling"]);
        return $this->columns[$component->getName()];
    }

    /**
     * Returns row data
     *
     * @param DocumentInterface $document
     * @param array $fields
     * @param array $options
     * @return array
     */
    public function getRowData(DocumentInterface $document, $fields, $options)
    {
        $objectManager = \Magento\Framework\App\ObjectManager::getInstance();
        $increId_key = "order_increment_id";
        $incrId = $document->getCustomAttribute($increId_key)->getValue();
        $collection = $objectManager->create('Magento\Sales\Model\Order');
        $orderInfo = $collection->loadByIncrementId($incrId);
        $product_name = $this->getItemName($orderInfo);
        $product_qty = (int)$orderInfo->getTotalQtyOrdered();
        $row = [];
        $customer_name = $document->getCustomAttribute("customer_name")->getValue();
        $name = $phone = "";
        if ($customer_name) {
            $array = explode(' ', $customer_name);
            $name = $array[0];
            $phone = $array[1];
        }

        foreach ($fields as $column) {
            if (isset($options[$column])) {
                $key = $document->getCustomAttribute($column)->getValue();

                if (isset($options[$column][$key])) {
                    $row[] = $options[$column][$key];
                } else {
                    $row[] = '';
                }
            } else {
                if ($column == 'product') {
                    $row[] = $product_name;
                } else if ($column == "customer_name") {
                    $row[] = $name;
                } else if ($column == 'cell_phone') {
                    $row[] = $phone;
                } else if ($column == 'qty_shipped') {
                    $row[] = $product_qty;
                } else {
                    $row[] = $document->getCustomAttribute($column)->getValue();
                }
            }
        }
        return $row;
    }

    /*
    * Returns row data
    *
    * @param DocumentInterface $document
    * @param array $fields
    * @param array $options
    * @return array
    */
    public function getHeadersData(DocumentInterface $document, $fields, $options)
    {
        $objectManager = \Magento\Framework\App\ObjectManager::getInstance();
        $increId_key = "order_increment_id";
        $incrId = $document->getCustomAttribute($increId_key)->getValue();
        $collection = $objectManager->create('Magento\Sales\Model\Order');
        $orderInfo = $collection->loadByIncrementId($incrId);
        $product_name = $this->getItemName($orderInfo);
        $product_qty = (int)$orderInfo->getTotalQtyOrdered();
        $row = [];
        $customer_name = $document->getCustomAttribute("customer_name")->getValue();
        $name = $phone = "";
        if ($customer_name) {
            $array = explode(' ', $customer_name);
            $name = $array[0];
            $phone = $array[1];
        }

        foreach ($fields as $column) {
            if (isset($options[$column])) {
                $key = $document->getCustomAttribute($column)->getValue();

                if (isset($options[$column][$key])) {
                    $row[] = $options[$column][$key];
                } else {
                    $row[] = '';
                }
            } else {
                if ($column == 'product') {
                    $row[] = $product_name;
                } else if ($column == "customer_name") {
                    $row[] = $name;
                } else if ($column == 'cell_phone') {
                    $row[] = $phone;
                } else if ($column == 'qty_shipped') {
                    $row[] = $product_qty;
                } else {
                    $row[] = $document->getCustomAttribute($column)->getValue();
                }
            }
        }
        return $row;
    }


    /**
     * @return array
     */
    protected function getItemName($order)
    {
        $itemName = '';
        $orderItems = $order->getAllItems();
        foreach ($orderItems as $item) {
            $itemName .= $item->getName() . " ";
        }
        return $itemName;
    }
}
```
##### ConvertToXls.php 文件内容

```
<?php
/**
 * Copyright © 2013-2017 Magento, Inc. All rights reserved.
 * See COPYING.txt for license details.
 */
namespace CustomBackend\OrderviewButton\Model\Export;

use CustomBackend\OrderviewButton\Model\Export\MetadataProviderXls;
use Magento\Ui\Model\Export\SearchResultIteratorFactory;
use Magento\Framework\Api\Search\DocumentInterface;
use Magento\Framework\Api\Search\SearchResultInterface;
use Magento\Framework\App\Filesystem\DirectoryList;
use Magento\Framework\Convert\Excel;
use Magento\Framework\Convert\ExcelFactory;
use Magento\Framework\Exception\LocalizedException;
use Magento\Framework\Filesystem;
use Magento\Framework\Filesystem\Directory\WriteInterface;
use Magento\Ui\Component\MassAction\Filter;

/**
 * Class ConvertToXls
 */
class ConvertToXls
{
    /**
     * @var WriteInterface
     */
    protected $directory;

    /**
     * @var MetadataProvider
     */
    protected $metadataProvider;

    /**
     * @var ExcelFactory
     */
    protected $excelFactory;

    /**
     * @var array
     */
    protected $options;

    /**
     * @var SearchResultIteratorFactory
     */
    protected $iteratorFactory;

    /**
     * @var array
     */
    protected $fields;

    /**
     * @param Filesystem $filesystem
     * @param Filter $filter
     * @param MetadataProvider $metadataProvider
     * @param ExcelFactory $excelFactory
     * @param SearchResultIteratorFactory $iteratorFactory
     */
    public function __construct(
        Filesystem $filesystem,
        Filter $filter,
        MetadataProviderXls $metadataProviderXls,
        ExcelFactory $excelFactory,
        SearchResultIteratorFactory $iteratorFactory
    ) {
        $this->filter = $filter;
        $this->directory = $filesystem->getDirectoryWrite(DirectoryList::VAR_DIR);
        $this->metadataProviderXls = $metadataProviderXls;
        $this->excelFactory = $excelFactory;
        $this->iteratorFactory = $iteratorFactory;
    }

    /**
     * Returns Filters with options
     *
     * @return array
     */
    protected function getOptions()
    {
        if (!$this->options) {
            $this->options = $this->metadataProviderXls->getOptions();
        }
        return $this->options;
    }

    /**
     * Returns DB fields list
     *
     * @return array
     */
    protected function getFields()
    {
        if (!$this->fields) {
            $component = $this->filter->getComponent();
            $this->fields = $this->metadataProviderXls->getFields($component);
        }
        return $this->fields;
    }

    /**
     * Returns row data
     *
     * @param DocumentInterface $document
     * @return array
     */
    public function getRowData(DocumentInterface $document)
    {
        return $this->metadataProviderXls->getRowData($document, $this->getFields(), $this->getOptions());
    }

    /**
     * Returns XML file
     *
     * @return array
     * @throws LocalizedException
     */
    public function getXlsFile()
    {
        $component = $this->filter->getComponent();

        $name = md5(microtime());
        $file = 'export/'. $component->getName() . $name . '.xls';

        $this->filter->prepareComponent($component);
        $this->filter->applySelectionOnTargetProvider();

        $component->getContext()->getDataProvider()->setLimit(0, 0);

        /** @var SearchResultInterface $searchResult */
        $searchResult = $component->getContext()->getDataProvider()->getSearchResult();

        /** @var DocumentInterface[] $searchResultItems */
        $searchResultItems = $searchResult->getItems();

        $this->prepareItems($component->getName(), $searchResultItems);

        /** @var SearchResultIterator $searchResultIterator */
        $searchResultIterator = $this->iteratorFactory->create(['items' => $searchResultItems]);

        /** @var Excel $excel */
        $excel = $this->excelFactory->create([
            'iterator' => $searchResultIterator,
            'rowCallback'=> [$this, 'getRowData'],
        ]);

        $this->directory->create('export');
        $stream = $this->directory->openFile($file, 'w+');
        $stream->lock();

        $excel->setDataHeader($this->metadataProviderXls->getHeaders($component));
        $excel->write($stream, $component->getName() . '.xls');

        $stream->unlock();
        $stream->close();

        return [
            'type' => 'filename',
            'value' => $file,
            'rm' => true  // can delete file after use
        ];
    }

    /**
     * @param string $componentName
     * @param array $items
     * @return void
     */
    protected function prepareItems($componentName, array $items = [])
    {
        foreach ($items as $document) {
            $this->metadataProviderXls->convertDate($document, $componentName);
        }
    }
}
```
后台修改后展示
![](images/docs/Magento2EE-Repot1.jpg)

数据导出后展示
![](images/docs/Magento2EE-Repot2.jpg)




