title: JAVA树型结构导出excel，并合并单元格
author: Mario
date: 2021-01-21 17:47:55
tags:
---
最近写了一个树型结构的目录导出Excel的代码逻辑，这里记录一下实现思路

<!--more-->

###### 查询目录列表，并转换为树型结构的数据
``` bash
1、数据库查询数据，获得数据类型为List
 List<Map<String, Object>> categorys = service.list(map);
 
2、转换为树型结构
 /**
     * 根据目录列表生成导出数据
     * @param categoryList 原始数据列表
     * @return
     */
    public List<CategoryTree> makeCategoryTree(List<Map<String, Object>> categoryList){
        List<CategoryTree> categoryTree = new ArrayList<>();
        if (categoryList != null) {
            categoryList.forEach(f -> {
                if (Objects.equals(FIRST_CATEGORY_PARENYID,f.get("PARENTID"))){
                    categoryTree.add(new CategoryTree(f.get("ID").toString(),f.get("NAME").toString(),FIRST_CATEGORY_PARENYID));
                }
            });

            //为一级目录设置子目录准备递归
            for (CategoryTree category:categoryTree) {
                //获取父菜单下所有子菜单调用getChildList
                List<CategoryTree> childList = getChildList(category.getId(),categoryList);
                category.setDatas(childList);
            }

        }
        return categoryTree;
    }
    
 /**
     * 递归获得下级目录
     * @param resId
     * @param categoryList
     * @return
     */
    private List<CategoryTree> getChildList(String resId, List<Map<String, Object>> categoryList) {
        //构建子目录
        List<CategoryTree> childList = new ArrayList<>();
        //遍历传入的list
        categoryList.forEach(f->{
            //所有目录的父id与传入的根节点id比较，若相等则为该级目录的子目录
            if (Objects.equals(f.get("PARENTID"),resId)){
                childList.add(new CategoryTree(f.get("ID").toString(),f.get("NAME").toString(),f.get("PARENTID").toString()));
            }
        });

        //递归
        childList.forEach(c->{
            c.setDatas(getChildList(c.getId(),categoryList));
        });

        if (childList.size() == 0){
            return null;
        }
        return childList;
    }
    

```
###### 获得所有目录的最后一个节点
``` bash
 /**
     * 获得所有目录的最后一个节点
     * @param categoryTree
     * @return
     */
    public List<String> getCategoryTreeLastNode(List<CategoryTree> categoryTree){
        List<String> nodes = new ArrayList<>();
        if (categoryTree != null) {
            categoryTree.forEach(f -> {
                if (Objects.isNull(f.getDatas())){
                    nodes.add(f.getId());
                }else {
                    getLastNode(f.getDatas(),nodes);
                }
            });

        }
        return nodes;
    }

    /**
     * 递归获取最后一个节点
     * @param categoryTrees 下级节点数据
     * @param nodes 最后一集节点列表
     */
    private void getLastNode(List<CategoryTree> categoryTrees, List<String> nodes) {
        for (CategoryTree categoryTree: categoryTrees) {
            if (Objects.isNull(categoryTree.getDatas())) {
               nodes.add(categoryTree.getId());
            }else {
                getLastNode(categoryTree.getDatas(),nodes);
            }
        }
    }
```

###### 获得目录所以分支的节点id与名字
循环所有目录的最后一个节点数组，获得其所有父级节点信息，形成一个List<List<CategoryExcel>>类型的数据，外层List是行数。内层List是列的数据
``` bash
 /**
     * 获得所有节点的名称
     * @param lastNodes 最后一级节点列表
     * @param categorys 原始列表数据
     * @return
     */
    public List<List<CategoryExcel>> queryParentNames(List<String> lastNodes,List<Map<String, Object>> categorys) {
        List<List<String>> categoryIds = new ArrayList<>();
        List<List<CategoryExcel>> categoryNames = new ArrayList<>();
        for(int i=0;i<lastNodes.size();i++){
            //节点名称数组
            List<String> parentIds = new ArrayList<>();
            parentIds.add(lastNodes.get(i));
            getParentId(categorys, lastNodes.get(i), parentIds);

            //现在获得的节点id列表是倒叙的，需要进行顺序反转
            Collections.reverse(parentIds);
            categoryIds.add(parentIds);
        }

        for (List<String> cIds: categoryIds) {
            List<CategoryExcel> nodeNames = new ArrayList<>();
            for (String cId: cIds) {
                for (Map<String, Object> f: categorys) {
                    if (Objects.equals(cId,f.get("ID"))){
                        nodeNames.add(new CategoryExcel(f.get("ID").toString(),f.get("NAME").toString()));
                    }
                }
            }
            categoryNames.add(nodeNames);
        }
        return categoryNames;
    }

    /**
     * 递归获取父级信息
     * @param categorys 原始列表数据
     * @param id 节点id
     * @param nodeIds 节点id数组
     */
    private void getParentId(List<Map<String, Object>> categorys, String id, List<String> nodeIds) {
        for (Map<String, Object> f: categorys) {
            if (Objects.equals(FIRST_CATEGORY_PARENYID,f.get("PARENTID"))){
                continue;
            }
            //判断是否有父节点
            if (id.equals(f.get("ID"))) {
                nodeIds.add(f.get("PARENTID").toString());
                getParentId(categorys, f.get("PARENTID").toString(), nodeIds);
            }
        }
    }
```

###### 生成excel文件
``` bash
 //生成excel
        XSSFWorkbook  wb = new XSSFWorkbook();
        wb = creatSheet(wb,nodeNames);

        //保存excel文件
        String savePath = excelSavePath + "directoryExport";
        File tempFilePath = new File(savePath);
        String fileName = "安全生产档案目录.xlsx";
        File tempFile = null;
        tempFile = new File(tempFilePath,fileName);
        //文件存在时先进行删除
        tempFile.deleteOnExit();

        try(FileOutputStream saveStream = new FileOutputStream (tempFile)){
            //保存文件
            //把编辑过后的工作薄重新保存为excel文件
            wb.write(saveStream);
        } catch (IOException e) {
            tempFile.deleteOnExit();
        }

        String resultPath = "/formParser?status=fileUpload&action=filedownload&fileName="+fileName+"&filePath=" + "directoryExport/" + tempFile
                .getName();

```

###### 填充数据，生成标题和表头
``` bash
  /**
     * excel数据填充
     * @param wb XSSFWorkbook对象
     * @param datas 填充数据
     * @return
     */
    public XSSFWorkbook creatSheet(XSSFWorkbook wb,List<List<CategoryExcel>> datas){
        //建立新的sheet对象（excel的表单）
        //新建sheet页
        XSSFSheet sheet = wb.createSheet();

        //新建单元格样式
        XSSFCellStyle jz = wb.createCellStyle();
        //水平居中
        jz.setAlignment(HorizontalAlignment.CENTER);
        //垂直居中
        jz.setVerticalAlignment(VerticalAlignment.CENTER);
        //自动换行
        jz.setWrapText(true);
        //第一行
        XSSFRow row0 = sheet.createRow(0);
        //设置行高
        row0.setHeight((short) ((short) 30*20));
        //创建单元格标题行
        XSSFCell cellTitle = row0.createCell(0);
        //设置单元格样式
        XSSFCellStyle styleTitle = creatStyle(wb);
        cellTitle.setCellStyle(styleTitle);
        //设置单元格内容
        cellTitle.setCellValue("安全生产档案目录");
        //合并单元格CellRangeAddress构造参数依次表示起始行，截至行，起始列， 截至列
        sheet.addMergedRegion(new CellRangeAddress(0,0,0,6));

        //创建内容样式
        XSSFCellStyle styleContent = wb.createCellStyle();
        styleContent.setWrapText(true);
        styleContent.setAlignment(HorizontalAlignment.CENTER);
        styleContent.setVerticalAlignment(VerticalAlignment.CENTER);

        //创建单元格表头行
        XSSFRow row1 = sheet.createRow(1);
        //设置行高
        row1.setHeight((short) 300);
        //列
        XSSFCell cell1_0 = row1.createCell(0);
        cell1_0.setCellStyle(styleContent);
        cell1_0.setCellValue("序号");

        XSSFCell cell1_1 = row1.createCell(1);
        cell1_1.setCellStyle(styleContent);
        cell1_1.setCellValue("大项");

        XSSFCell cell1_2 = row1.createCell(2);
        cell1_2.setCellStyle(styleContent);
        cell1_2.setCellValue("小项");

        XSSFCell cell1_3 = row1.createCell(3);
        cell1_3.setCellStyle(styleContent);
        cell1_3.setCellValue("小项");

        XSSFCell cell1_4 = row1.createCell(4);
        cell1_4.setCellStyle(styleContent);
        cell1_4.setCellValue("小项");

        XSSFCell cell1_5 = row1.createCell(5);
        cell1_5.setCellStyle(styleContent);
        cell1_5.setCellValue("小项");

        XSSFCell cell1_6 = row1.createCell(6);
        cell1_6.setCellStyle(styleContent);
        cell1_6.setCellValue("小项");

        //数据填充
        setValue(sheet,styleContent,datas);

        return wb;
    }

```

###### 填充数据，填充内容
``` bash
  /**
     * 单元格赋值
     * @param sheet XSSFSheet对象
     * @param styleContent XSSFCellStyle对象
     * @param datas 填充数据
     */
    public void setValue(XSSFSheet sheet,XSSFCellStyle styleContent, List<List<CategoryExcel>> datas){

        for (int i=0;i<datas.size();i++) {
            List<CategoryExcel> cellDatas = datas.get(i);
            //创建单元格表行
            XSSFRow row1 = sheet.createRow(2+i);
            //设置行高
            row1.setHeight((short) 300);
            //列
            XSSFCell cell1 = row1.createCell(0);
            cell1.setCellStyle(styleContent);
            cell1.setCellValue(i+1);
            for (int j=0;j<cellDatas.size();j++) {
                //列
                XSSFCell cell = row1.createCell(1+j);
                cell.setCellStyle(styleContent);
                cell.setCellValue(cellDatas.get(j).getName());
            }
        }

        //单元格合并
        cellMerge(sheet,datas);
    }
```
###### 合并单元格
``` bash
  /**
     * 单元格合并
     * @param sheet XSSFSheet对象
     * @param datas 填充数据
     */
    public void cellMerge(XSSFSheet sheet, List<List<CategoryExcel>> datas){

        //循环6级分类列做合并
        for (int j=0;j<6;j++){
            //上一个目录
            String perCategory = "";
            //当前目录
            String category = "";

            //要合并的第一行
            int startMergeCol = 2;
            //合并的最后行
            int endMergeCol = 1;

            for (int i=0;i<datas.size();i++) {
                List<CategoryExcel> cellDatas = datas.get(i);
                if(cellDatas.size()>j){
                    //取每行的对应级目录id,使用id判断，避免名称相同的情况被合并
                    category = cellDatas.get(j).getId();
                    if ("".equals(perCategory)){
                        perCategory = category;
                    }
                    if(category.equals(perCategory)){
                        endMergeCol++;
                    }else {
                        if (endMergeCol>startMergeCol){
                            //两者不相同时，融合之前相同的行
                            //合并单元格CellRangeAddress构造参数依次表示起始行，截至行，起始列， 截至列
                            sheet.addMergedRegion(new CellRangeAddress(startMergeCol, endMergeCol, j+1, j+1));
                        }
                        startMergeCol = endMergeCol+1;
                        endMergeCol = startMergeCol;
                        perCategory = category;
                    }
                }else {
                    if (i==0){
                        //如果第一行为空，合并的开始结束行直接加1
                        startMergeCol++;
                        endMergeCol++;
                    }else {
                        if (endMergeCol>startMergeCol){
                            sheet.addMergedRegion(new CellRangeAddress(startMergeCol, endMergeCol, j+1, j+1));
                        }
                        startMergeCol = endMergeCol+1;
                        endMergeCol = startMergeCol;
                    }

                }

                //处理最后一个不合并的问题
                if (i == (datas.size()-1) ){
                    if (endMergeCol>startMergeCol){
                        //合并单元格CellRangeAddress构造参数依次表示起始行，截至行，起始列， 截至列
                        sheet.addMergedRegion(new CellRangeAddress(startMergeCol, endMergeCol, j+1, j+1));
                    }
                }
            }
        }
        
    }
```