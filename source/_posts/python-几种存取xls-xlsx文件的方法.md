---
title: python-几种存取xls/xlsx文件的方法
date: 2020-01-02 14:30:34
tags: [python]
---

# python-几种存取xls/xlsx文件的方法

## [demo地址](https://github.com/AsummerCat/other_demo/tree/master/xlsx)

## **xlwt/xlrd库** 

### 写入

```python
# -*- coding:utf-8 -*-
import xlwt

'''
写入excel
'''


def write_excel():
    workbook = xlwt.Workbook(encoding='utf-8')
    booksheet = workbook.add_sheet('Sheet 1', cell_overwrite_ok=True)
    # 存第一行cell(1,1)和cell(1,2)
    booksheet.write(0, 0, 'Python开发')
    booksheet.write(0, 1, 'Web前端')
    # 存第二行cell(2,1)和cell(2,2)
    booksheet.write(1, 0, '100人')
    booksheet.write(1, 1, '200人')
    # 存一行数据
    rowdata = [43, 56]
    for i in range(len(rowdata)):
        booksheet.write(2, i, rowdata[i])
    workbook.save('test_xlwt.xlsx')


if __name__ == '__main__':
    write_excel()
    print("xlwt已将数据写入Excel")

```



### 读取

<!--more-->

```python
# -*- coding:utf-8 -*-
import xlrd

'''
读取excel
'''


def read_excel():
    workbook = xlrd.open_workbook('test_xlwt.xlsx')
    print(workbook.sheet_names())  # 查看所有sheet
    booksheet = workbook.sheet_by_index(0)  # 用索引取第一个sheet
    booksheet = workbook.sheet_by_name('Sheet 1')  # 或用名称取sheet
    # 读单元格数据
    cell_11 = booksheet.cell_value(0, 0)
    cell_21 = booksheet.cell_value(1, 0)
    cell_12 = booksheet.cell_value(0, 1)
    cell_22 = booksheet.cell_value(1, 1)
    # 读一行数据
    row_3 = booksheet.row_values(2)
    print(cell_11, cell_21, cell_12, cell_22, row_3)


if __name__ == '__main__':
    read_excel()
    print("xlwt已将Excel读取完毕")

```

## **openpyxl 库**

### 写入 

```python
# -*- coding:utf-8 -*-
from openpyxl import Workbook

'''
写入excel
'''


def write_excel():
    wb = Workbook()  # 打开 excel 工作簿
    ws1 = wb.active  # 激活
    # ws1.title = ""  # 给予文件标题
    # 存第一行单元格cell(1,1)
    ws1.cell(1, 1).value = 6  # 这个方法索引从1开始
    ws1.cell(2, 1).value = 6  # 这个方法索引从1开始
    ws1.cell(3, 1).value = 6  # 这个方法索引从1开始
    ws1['A4'] = 4

    # 存一行数据
    ws1.append([11, 87])
    wb.save("openpyxl.xlsx")


if __name__ == '__main__':
    write_excel()
    print("openyxl已将数据写入Excel")

```

### 读取

```python
# -*- coding:utf-8 -*-
from openpyxl import load_workbook

def read_excel():
    import openpyxl

    wb = openpyxl.load_workbook('openpyxl.xlsx')
    # 获取所有工作表名
    names = wb.sheetnames
    # wb.get_sheet_by_name(name) 已经废弃,使用wb[name] 获取指定工作表
    sheet = wb[names[0]]
    # 获取最大行数
    maxRow = sheet.max_row
    # 获取最大列数
    maxColumn = sheet.max_column
    # 获取当前活动表
    current_sheet = wb.active
    # 获取当前活动表名称
    current_name = sheet.title
    # 通过名字访问Cell对象, 通过value属性获取值
    a1 = sheet['A1'].value
    print("a1的数据:{}".format(a1))
    # 通过行和列确定数据
    a52 = sheet.cell(row=5, column=2).value
    print("a5 2的数据:{}".format(a52))
    # 获取列字母
    column_name = openpyxl.utils.cell.get_column_letter(1)
    # 将列字母转为数字, 参数忽略大小写
    column_name_num = openpyxl.utils.cell.column_index_from_string('a')
    # 获取一列数据, sheet.iter_rows() 获取所有的行
    for one_column_data in sheet.iter_rows():
        print(one_column_data[0].value)
    # for one_row_data in sheet.iter_cols():
    #     print(one_row_data[0].value, end="\t")

    print("row = {}, column = {}".format(maxRow, maxColumn))



    # workbook = load_workbook('openpyxl.xlsx')
    # # booksheet = workbook.active                #获取当前活跃的sheet,默认是第一个sheet
    # sheets = workbook.get_sheet_names()  # 从名称获取sheet
    # booksheet = workbook.get_sheet_by_name(sheets[0])
    #
    # rows = booksheet.rows
    # columns = booksheet.columns
    # # 迭代所有的行
    # for row in rows:
    #     line = [col.value for col in row]
    #
    # # 通过坐标读取值
    # cell_11 = booksheet.cell('A1')
    # cell_11 = booksheet.cell(row=1, column=1).value

if __name__ == '__main__':
    read_excel()
    print("openpyxl已将Excel读取完毕")
```

## 导出CSV文件

如果对存储格式没有要求的话 **csv文件** 

### 导入1

```python
import pandas as pd
import numpy as np

'''
写入CSV
'''
def write_csv1():
    csv_mat = np.empty((0, 2), float)
    csv_mat = np.append(csv_mat, [[43, 55]], axis=0)
    csv_mat = np.append(csv_mat, [[65, 67]], axis=0)
    csv_pd = pd.DataFrame(csv_mat)
    csv_pd.to_csv("test_pd.csv", sep=',', header=False, index=False)
```

## 写入2

```python
# -*- coding:utf-8 -*-
def write_csv2():
    import unicodecsv as ucsv
    data = [[u"列1", u"列2"], [u"内容1", u"内容2"]]
    with open('test.csv', 'wb') as f:
        w = ucsv.writer(f, encoding='utf-8')
        w.writerows(data)

```



### 输出

```python
import pandas as pd
 
filename = "D:\\Py_exercise\\test_pd.csv"
csv_data = pd.read_csv(filename, header=None)
csv_data = np.array(csv_data, dtype=float)

```

