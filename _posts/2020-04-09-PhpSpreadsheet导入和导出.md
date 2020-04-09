---
layout: post
title: 'PhpSpreadsheet导入和导出'
date: 2020-04-09
author: smallsat
categories: PHP PHPOffice
tags: PHP Excel PHPOffice
---

# 安装
```php
composer reuqire phpoffice/phpspreadsheet
```

# 读取excel

```php
<?php

function read($file,  $fields = [],$start_line=1,$line=2000,$sheet_index=0)
{
    require_once __DIR__ . '/vendor/autoload.php';
    $file = dirname(__FILE__) . '/01simple.xlsx';
    $reader = IOFactory::createReaderForFile($file);
    $reader->setReadDataOnly(true); //不需要更改，加快速度
    if ($line > 0) {
        $filter = new ReadFilter();//增加读取规则，避免无用的空白行导致的内存占用高
        $filter->start_row = $start_line;
        $filter->end_row = $line;
    }
    $obj = $reader->load($file);    //加载文件
    $currSheet = $obj->getSheet($sheet_index);      
    // 获取所有列
    $cell = $currSheet->getHighestColumn(); 
            /* 读取内容 */
            for ($_row = intval($start_line); $_row <= $line; $_row++) {
                $isNull = true;

                for ($_column = 1; $_column <= $cell; $_column++) {
                    $orCellName = $cellName = Coordinate::stringFromColumnIndex($_column);
                    $cellId   = $cellName . $_row;
                    $cellValue =  trim($currSheet->getCell($cellId)->getValue());
                    //第一个无值直接不拿该行
                    if ($_column == 1 && ($cellValue == '' || is_null($cellValue))) {
                        break;
                    }
                    if (!empty($fields) && !isset($fields[$cellName])) continue ;
                    $data[$_row][$cellName] =$cellValue;
                    if (!empty($data[$_row][$cellName])) {
                        $isNull = false;
                    }
                }

                /* 判断是否整行数据为空，是的话删除该行数据 */
                if ($isNull) {
                    unset($data[$_row]);
                }
       
            }
            $obj->disconnectWorksheets();   //释放资源
            return ['error'=>0,'data'=>$data];
    

```
```php
<?php
namespace ;


use PhpOffice\PhpSpreadsheet\Cell\Coordinate;
use PhpOffice\PhpSpreadsheet\Reader\IReadFilter;

class ReadFilter implements IReadFilter
{
    public $end_row;
    public $start_row = 1;
    public $start_column = 'A';
    public $end_column ;

    /**
     * @param string $column
     * @param int $row
     * @param string $worksheetName
     * @return bool
     * @throws \PhpOffice\PhpSpreadsheet\Exception
     */
    public function readCell($column, $row, $worksheetName = '')
    {
        //读取全部
        if (!$this->end_row ) {
            return true;
        }
      //只读取指定的行
        if ($row >= $this->start_row && $row <= $this->end_row) {
            return true;
        }

        return false;
    }
}
```
# 导出excel
```php
<?php
/**
 * Excel导出，TODO 可继续优化
 *
 * @param array  $datas      导出数据，格式['A1' => 'XXXX公司报表', 'B1' => '序号']
 * @param string $fileName   导出文件名称
 * @param array  $options    操作选项，例如：
 *                           bool   print       设置打印格式
 *                           string freezePane  锁定行数，例如表头为第一行，则锁定表头输入A2
 *                           array  setARGB     设置背景色，例如['A1', 'C1']
 *                           array  setWidth    设置宽度，例如['A' => 30, 'C' => 20]
 *                           bool   setBorder   设置单元格边框
 *                           array  mergeCells  设置合并单元格，例如['A1:J1' => 'A1:J1']
 *                           array  formula     设置公式，例如['F2' => '=IF(D2>0,E42/D2,0)']
 *                           array  format      设置格式，整列设置，例如['A' => 'General']
 *                           array  alignCenter 设置居中样式，例如['A1', 'A2']
 *                           array  bold        设置加粗样式，例如['A1', 'A2']
 *                           string savePath    保存路径，设置后则文件保存到服务器，不通过浏览器下载
    @link 参考文档 https://blog.csdn.net/DestinyLordC/article/details/84071456 
 */
function exportExcel(array $datas, string $fileName = '', array $options = []): bool
{
    try {
        if (empty($datas)) {
            return false;
        }

        set_time_limit(0);
        /** @var Spreadsheet $objSpreadsheet */
        $objSpreadsheet = app(Spreadsheet::class);
        /* 设置默认文字居左，上下居中 */
        $styleArray = [
            'alignment' => [
                'horizontal' => Alignment::HORIZONTAL_LEFT,
                'vertical'   => Alignment::VERTICAL_CENTER,
            ],
        ];
        $objSpreadsheet->getDefaultStyle()->applyFromArray($styleArray);
        /* 设置Excel Sheet */
        $activeSheet = $objSpreadsheet->setActiveSheetIndex(0);

        /* 打印设置 */
        if (isset($options['print']) && $options['print']) {
            /* 设置打印为A4效果 */
            $activeSheet->getPageSetup()->setPaperSize(PageSetup:: PAPERSIZE_A4);
            /* 设置打印时边距 */
            $pValue = 1 / 2.54;
            $activeSheet->getPageMargins()->setTop($pValue / 2);
            $activeSheet->getPageMargins()->setBottom($pValue * 2);
            $activeSheet->getPageMargins()->setLeft($pValue / 2);
            $activeSheet->getPageMargins()->setRight($pValue / 2);
        }

        /* 行数据处理 */
        foreach ($datas as $sKey => $sItem) {
            /* 默认文本格式 */
            $pDataType = DataType::TYPE_STRING;

            /* 设置单元格格式 */
            if (isset($options['format']) && !empty($options['format'])) {
                $colRow = Coordinate::coordinateFromString($sKey);

                /* 存在该列格式并且有特殊格式 */
                if (isset($options['format'][$colRow[0]]) &&
                    NumberFormat::FORMAT_GENERAL != $options['format'][$colRow[0]]) {
                    $activeSheet->getStyle($sKey)->getNumberFormat()
                        ->setFormatCode($options['format'][$colRow[0]]);

                    if (false !== strpos($options['format'][$colRow[0]], '0.00') &&
                        is_numeric(str_replace(['￥', ','], '', $sItem))) {
                        /* 数字格式转换为数字单元格 */
                        $pDataType = DataType::TYPE_NUMERIC;
                        $sItem     = str_replace(['￥', ','], '', $sItem);
                    }
                } elseif (is_int($sItem)) {
                    $pDataType = DataType::TYPE_NUMERIC;
                }
            }

            $activeSheet->setCellValueExplicit($sKey, $sItem, $pDataType);

            /* 存在:形式的合并行列，列入A1:B2，则对应合并 */
            if (false !== strstr($sKey, ":")) {
                $options['mergeCells'][$sKey] = $sKey;
            }
        }

        unset($datas);

        /* 设置锁定行 */
        if (isset($options['freezePane']) && !empty($options['freezePane'])) {
            $activeSheet->freezePane($options['freezePane']);
            unset($options['freezePane']);
        }

        /* 设置宽度 */
        if (isset($options['setWidth']) && !empty($options['setWidth'])) {
            foreach ($options['setWidth'] as $swKey => $swItem) {
                $activeSheet->getColumnDimension($swKey)->setWidth($swItem);
            }

            unset($options['setWidth']);
        }

        /* 设置背景色 */
        if (isset($options['setARGB']) && !empty($options['setARGB'])) {
            foreach ($options['setARGB'] as $sItem) {
                $activeSheet->getStyle($sItem)
                    ->getFill()->setFillType(Fill::FILL_SOLID)
                    ->getStartColor()->setARGB(Color::COLOR_YELLOW);
            }

            unset($options['setARGB']);
        }

        /* 设置公式 */
        if (isset($options['formula']) && !empty($options['formula'])) {
            foreach ($options['formula'] as $fKey => $fItem) {
                $activeSheet->setCellValue($fKey, $fItem);
            }

            unset($options['formula']);
        }

        /* 合并行列处理 */
        if (isset($options['mergeCells']) && !empty($options['mergeCells'])) {
            $activeSheet->setMergeCells($options['mergeCells']);
            unset($options['mergeCells']);
        }

        /* 设置居中 */
        if (isset($options['alignCenter']) && !empty($options['alignCenter'])) {
            $styleArray = [
                'alignment' => [
                    'horizontal' => Alignment::HORIZONTAL_CENTER,
                    'vertical'   => Alignment::VERTICAL_CENTER,
                ],
            ];

            foreach ($options['alignCenter'] as $acItem) {
                $activeSheet->getStyle($acItem)->applyFromArray($styleArray);
            }

            unset($options['alignCenter']);
        }

        /* 设置加粗 */
        if (isset($options['bold']) && !empty($options['bold'])) {
            foreach ($options['bold'] as $bItem) {
                $activeSheet->getStyle($bItem)->getFont()->setBold(true);
            }

            unset($options['bold']);
        }

        /* 设置单元格边框，整个表格设置即可，必须在数据填充后才可以获取到最大行列 */
        if (isset($options['setBorder']) && $options['setBorder']) {
            $border    = [
                'borders' => [
                    'allBorders' => [
                        'borderStyle' => Border::BORDER_THIN, // 设置border样式
                        'color'       => ['argb' => 'FF000000'], // 设置border颜色
                    ],
                ],
            ];
            $setBorder = 'A1:' . $activeSheet->getHighestColumn() . $activeSheet->getHighestRow();
            $activeSheet->getStyle($setBorder)->applyFromArray($border);
            unset($options['setBorder']);
        }

        $fileName = !empty($fileName) ? $fileName : (date('YmdHis') . '.xlsx');

        if (!isset($options['savePath'])) {
            /* 直接导出Excel，无需保存到本地，输出07Excel文件 */
            header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
            header(
                "Content-Disposition:attachment;filename=" . iconv(
                    "utf-8", "GB2312//TRANSLIT", $fileName
                )
            );
            header('Cache-Control: max-age=0');//禁止缓存
            $savePath = 'php://output';
        } else {
            $savePath = $options['savePath'];
        }

        ob_clean();
        ob_start();
        $objWriter = IOFactory::createWriter($objSpreadsheet, 'Xlsx');
        $objWriter->save($savePath);
        /* 释放内存 */
        $objSpreadsheet->disconnectWorksheets();
        unset($objSpreadsheet);
        ob_end_flush();

        return true;
    } catch (Exception $e) {
        return false;
    }
}

```
