---
layout:       post
title:        "java.lang.IllegalStateException: Cannot get a text value from a numeric cell"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - working
    - poi
    - java
---

# 原因
- Excel中数值类型值在程序中按字符串类型获取
# 解决
```java
    /**
     * 拿到不同类型单元格中的值
     * 1. 字符串: 字符串
     * 2. 布尔: toString
     * 3. 数值(double): 格式化后的字符串
     * @param cell 获取的单元格
     * @return 单元格中的值
     */
    private static String getCellValue(Cell cell) {
        String resultValue = "";
        // 判空
        if (Objects.isNull(cell)) {
            return resultValue;
        }

        // 拿到单元格类型
        int cellType = cell.getCellType();
        switch (cellType) {
            // 字符串类型
            case Cell.CELL_TYPE_STRING:
                resultValue = StringUtils.isEmpty(cell.getStringCellValue()) ? "" : cell.getStringCellValue().trim();
                break;
            // 布尔类型
            case Cell.CELL_TYPE_BOOLEAN:
                resultValue = String.valueOf(cell.getBooleanCellValue());
                break;
            // 数值类型
            case Cell.CELL_TYPE_NUMERIC:
                resultValue = new DecimalFormat("#.######").format(cell.getNumericCellValue());
                break;
            // 取空串
            default:
                break;
        }
        return resultValue;
    }
```