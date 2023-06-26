title: EasyExcel生成带下拉和校验的导入模板
author: Mario
date: 2023-06-26 17:09:50
tags:
---
EasyExcel结合业务系统的字典管理生成excel导入模版，模版中包含：<font color=red>日期、数字格式校验、下拉数据、级联下拉数据（目前最多3级，可调整添加）</font>

<!--more-->
#### 自定义注解ExcelDictAnnotation
``` bash
/***
 * @classname ExcelDictAnnotation
 * @description 用于实体字段标识字典值
   使用时将该注解加到字典字段上如：@ExcelDictAnnotation("computer_type")
 * @Author: RenTie
 * @Date: 2023-06-08 10:06
 */

@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcelDictAnnotation {
    String value();
}
```
#### 自定义注解ExcelCascadeDictAnnotation
``` bash
/***
 * @classname ExcelCascadeDictAnnotation
 * @author rentie
 * @date 2023/6/12 14:23
 * @description 导入导出级联下拉字段注解 使用方法如下：
 * 下列分别是三级级联对应存储的三个字段的注解方式
 * @ExcelCascadeDictAnnotation(dictCode = "achievement_category")
 * @ExcelCascadeDictAnnotation(dictCode = "achievement_category", parentColumnIndex = 0,level = 2)
 * @ExcelCascadeDictAnnotation(dictCode = "achievement_category", parentColumnIndex = 1, level = 3)
 */

@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcelCascadeDictAnnotation {
    // 上级选项列下标
    int parentColumnIndex() default -1;

    // 字典code
    String dictCode() default "";

    // 列数据对应的字典层级
    int level() default 1;

    // 下拉菜单生效范围开始 (行数下标，默认第2行，第1行为表头)
    int firstRow() default 1;

    // 下拉菜单生效范围结束 (行数下标，默认为excel最大行数65536)
    int lastRow() default 0x10000;
}

```
#### 模版生成工具类
<font color=red>获取字典数据部分的代码需要根据项目实际情况做调整</font>
``` bash
/**
 * EasyExcel工具类
 *
 * @Author: RenTie
 * @Date: 2023-06-08 10:06
 **/
public class EasyExcelUtils {

    private final static Log logger = LoggerFactory.getLog(EasyExcelUtils.class);

    /**
     * excel导出单sheet页且sheet页中含有下拉框的excel文件（导入模板生成下载）
     *
     * @param response
     *            HttpServletResponse
     * @param data
     *            要导出的数据（导出模版时使用空数据即可）
     * @param head
     *            导出表头信息
     */
    @SuppressWarnings("rawtypes")
    public static <T> void writeTemplate(final HttpServletResponse response, final List<T> data, final Class<?> head) {
        try {
            // 解析表头类中的级联下拉注解 Map<下拉框列索引, 字典数据预处理> map
            final Map<Integer, ExcelSelectDataColumn> cascadeSelectedMap = resolveCascadeSelectedAnnotation(head);
            // 获取字典类型的字段信息 Map<下拉框列索引, 下拉对应的字典数据> map
            final Map<Integer, List<String>> selectedMap = resolveSelectedAnnotation(head);
            // 日期字段下标
            final List<Integer> dateFieldIndexs = resolveDateField(head);
            // 整型字段下标
            final List<Integer> intFieldIndexs = resolveIntField(head);
            // 声明excel导出builder
            final ExcelWriterBuilder excelBuilder = EasyExcel.write(response.getOutputStream(), head)
                // 自动设置列宽
                .registerWriteHandler(new ExcelWidthStyleStrategy())
                // 设置下拉值数据验证
                .registerWriteHandler(new DataSelectSheetWriteHandler(selectedMap))
                // 设置级联下拉值数据验证
                .registerWriteHandler(new DataCascadeSelectSheetWriteHandler(cascadeSelectedMap))
                // 设置日期数据验证
                .registerWriteHandler(new DataFormatSheetWriteHandler(dateFieldIndexs, intFieldIndexs));
            // excel写入
            excelBuilder.sheet("Sheet1").doWrite(data);
        } catch (final IOException e) {
            logger.error("导出excel文件异常", e);
        }
    }

    /**
     * 解析表头类中的级联下拉注解
     *
     * @param head
     *            表头类
     * @return Map<下拉框列索引, 字典数据预处理> map
     */
    @SuppressWarnings({"rawtypes", "unchecked"})
    private static <T> Map<Integer, ExcelSelectDataColumn> resolveCascadeSelectedAnnotation(final Class<T> head) {
        // 获取动态下拉框的内容的字典值
        final DictItemServiceImpl dictItemService = ApplicationContextHolder.getBeanByType(DictItemServiceImpl.class);
        final Map<Integer, ExcelSelectDataColumn> selectedMap = Maps.newHashMap();
        final Field[] fields = head.getDeclaredFields();
        for (int i = 0; i < fields.length; i++) {
            final Field field = fields[i];
            final ExcelCascadeDictAnnotation cascadeDictAnnotation =
                field.getAnnotation(ExcelCascadeDictAnnotation.class);
            final ExcelProperty property = field.getAnnotation(ExcelProperty.class);
            if (cascadeDictAnnotation != null) {
                if (property != null && property.index() >= 0) {
                    final int columnIndex = property.index();
                    // 字典code
                    final String dictCode = cascadeDictAnnotation.dictCode();
                    // 上级选项列下标
                    final int parentColumnIndex = cascadeDictAnnotation.parentColumnIndex();
                    // 列数据对应的字典层级
                    final int level = cascadeDictAnnotation.level();
                    // 下拉菜单生效范围开始 (行数下标，默认第2行，第1行为表头)
                    final int firstRow = cascadeDictAnnotation.firstRow();
                    // 下拉菜单生效范围结束 (行数下标，默认为excel最大行数65536)
                    final int lastRow = cascadeDictAnnotation.lastRow();
                    final ExcelSelectDataColumn excelSelectedResolve;
                    // 第一级数据
                    final List<DictItem> firstDictItems = dictItemService.listDictTreeFirstLeve(dictCode);
                    // 判断是否为第一级下拉
                    if (level > 1) {
                        // 不是第一级时根据层级获取数据
                        excelSelectedResolve = new ExcelSelectDataColumn<Map<String, List<String>>>();
                        // 获取所有层级的数据
                        final List<DictItem> allDictItems = dictItemService.listDictItemByCode(dictCode);
                        final Map<String, List<String>> source = new HashMap<>();
                        // 第二级数据
                        if (level == 2) {
                            firstDictItems.forEach(firstDictItem -> {
                                // 下级信息
                                final List<String> child = new ArrayList<>();
                                allDictItems.forEach(allDictItem -> {
                                    if (StringUtils.equals(allDictItem.getParentId(), firstDictItem.getItemId())) {
                                        child.add(allDictItem.getItemText());
                                    }
                                });
                                source.put(firstDictItem.getItemText(), child);
                            });
                        }
                        // 第三级数据
                        if (level == 3) {
                            final List<DictItem> secondDictItems = new ArrayList<>();
                            firstDictItems.forEach(firstDictItem -> {
                                allDictItems.forEach(allDictItem -> {
                                    if (StringUtils.equals(allDictItem.getParentId(), firstDictItem.getItemId())) {
                                        secondDictItems.add(allDictItem);
                                    }
                                });
                            });
                            secondDictItems.forEach(secondDictItem -> {
                                // 下级信息
                                final List<String> child = new ArrayList<>();
                                allDictItems.forEach(allDictItem -> {
                                    if (StringUtils.equals(allDictItem.getParentId(), secondDictItem.getItemId())) {
                                        child.add(allDictItem.getItemText());
                                    }
                                });
                                source.put(secondDictItem.getItemText(), child);
                            });
                        }
                        excelSelectedResolve.setSource(source);
                    } else {
                        // 获取第一级下拉显示数据
                        final List<String> dictItemTexts = new ArrayList<>();
                        excelSelectedResolve = new ExcelSelectDataColumn<List<String>>();
                        // 获取字典项的显示值
                        if (Objects.nonNull(firstDictItems) && !firstDictItems.isEmpty()) {
                            firstDictItems.forEach(dictItem -> {
                                dictItemTexts.add(dictItem.getItemText());
                            });
                        }
                        if (CollUtil.isNotEmpty(dictItemTexts)) {
                            excelSelectedResolve.setSource(dictItemTexts);
                        }
                    }
                    excelSelectedResolve.setColumnIndex(columnIndex);
                    excelSelectedResolve.setParentColumnIndex(parentColumnIndex);
                    excelSelectedResolve.setFirstRow(firstRow);
                    excelSelectedResolve.setLastRow(lastRow);
                    excelSelectedResolve.setLevel(level);
                    // 保存解析后的数据
                    selectedMap.put(columnIndex, excelSelectedResolve);
                }
            }
        }
        return selectedMap;
    }

    /**
     * 解析表头类中的单个下拉注解
     *
     * @param head
     *            表头类
     * @return Map<下拉框列索引, 下拉对应的字典数据> map
     */
    private static <T> Map<Integer, List<String>> resolveSelectedAnnotation(final Class<T> head) {
        // 获取动态下拉框的内容的字典值
        final DictItemServiceImpl dictItemService = ApplicationContextHolder.getBeanByType(DictItemServiceImpl.class);
        final Map<Integer, List<String>> selectedMap = new HashMap<>(16);
        final Field[] fields = head.getDeclaredFields();
        for (int i = 0; i < fields.length; i++) {
            final Field field = fields[i];
            final ExcelDictAnnotation dictAnnotation = field.getAnnotation(ExcelDictAnnotation.class);
            final ExcelProperty property = field.getAnnotation(ExcelProperty.class);
            if (dictAnnotation != null) {
                if (property != null && property.index() >= 0) {
                    final String dictCode = dictAnnotation.value();
                    final List<String> dictItemTexts = dictItemService.getDictItemTexts(dictCode);
                    if (CollUtil.isNotEmpty(dictItemTexts)) {
                        selectedMap.put(property.index(), dictItemTexts);
                    }
                }
            }
        }
        return selectedMap;
    }

    /**
     * 解析表头类中的日期字段
     *
     * @param head
     *            表头类
     * @return List<日期列索引> map
     */
    private static <T> List<Integer> resolveDateField(final Class<T> head) {
        final List<Integer> indexs = new ArrayList<>(16);
        final Field[] fields = head.getDeclaredFields();
        for (int i = 0; i < fields.length; i++) {
            final Field field = fields[i];
            final ExcelProperty property = field.getAnnotation(ExcelProperty.class);
            if (Objects.equals(field.getType(), LocalDate.class)) {
                indexs.add(property.index());
            }
        }
        return indexs;
    }

    /**
     * 解析表头类中的整型字段
     *
     * @param head
     *            表头类
     * @return List<整型列索引> map
     */
    private static <T> List<Integer> resolveIntField(final Class<T> head) {
        final List<Integer> indexs = new ArrayList<>(16);
        final Field[] fields = head.getDeclaredFields();
        for (int i = 0; i < fields.length; i++) {
            final Field field = fields[i];
            final ExcelProperty property = field.getAnnotation(ExcelProperty.class);
            if (Objects.equals(field.getType(), Integer.class)) {
                indexs.add(property.index());
            }
        }
        return indexs;
    }
}

```
#### 自定义整型、日期格式生成的SheetWriteHandler
``` bash
/**
 * @author rentie
 * @date 2023/6/8 15:25
 * @desc 日期类型设置单元格格式
 */
public class DataFormatSheetWriteHandler implements SheetWriteHandler {

    /**
     * 日期类型单元格的ColumnIndex, ColumnIndex 从0开始计数
     */
    private List<Integer> dateColumnIndexList;

    /**
     * 整型类型单元格的ColumnIndex, ColumnIndex 从0开始计数
     */
    private List<Integer> intColumnIndexList;

    public DataFormatSheetWriteHandler(final List<Integer> dateColumnIndexList,
        final List<Integer> intColumnIndexList) {
        this.dateColumnIndexList = dateColumnIndexList;
        this.intColumnIndexList = intColumnIndexList;
    }

    public DataFormatSheetWriteHandler() {}

    @Override
    public void afterSheetCreate(final WriteWorkbookHolder writeWorkbookHolder,
        final WriteSheetHolder writeSheetHolder) {

        // 创建日期数据规则
        this.dateColumnIndexList.forEach(index -> {
            // 检查的区域
            final CellRangeAddressList cellRangeAddressList = new CellRangeAddressList(1, 65536, index, index);
            final DataValidationHelper helper = writeSheetHolder.getSheet().getDataValidationHelper();
            // DVConstraint constrain1 = DVConstraint.CreateDateConstraint(条件,"最小时间","最大时间","时间格式"); //这是检查时间的方法
            final DataValidationConstraint constraint = helper.createDateConstraint(
                DataValidationConstraint.OperatorType.BETWEEN, "Date(1900, 1, 1)", "Date(2099, 12, 31)", "yyyy-MM-dd");
            final DataValidation dataValidation = helper.createValidation(constraint, cellRangeAddressList);
            dataValidation.setErrorStyle(DataValidation.ErrorStyle.STOP);
            // 输入无效值时是否显示错误框
            dataValidation.setShowErrorBox(true);
            // 验证输入数据是否真确
            dataValidation.setSuppressDropDownArrow(true);
            // 设置无效值时 是否弹出提示框
            dataValidation.setShowPromptBox(true);
            // 设置无效值时的提示框内容 createErrorBox
            dataValidation.createErrorBox("提示", "请输入[yyyy-MM-dd]格式日期！！！");
            dataValidation.createPromptBox("提示", "yyyy-MM-dd 格式日期");
            writeSheetHolder.getSheet().addValidationData(dataValidation);
        });

        // 创建整型数据规则
        this.intColumnIndexList.forEach(index -> {
            // 检查的区域
            final CellRangeAddressList cellRangeAddressList = new CellRangeAddressList(1, 65536, index, index);
            final DataValidationHelper helper = writeSheetHolder.getSheet().getDataValidationHelper();
            final DataValidationConstraint constraint = helper.createIntegerConstraint(
                DataValidationConstraint.OperatorType.BETWEEN, "0", String.valueOf(Integer.MAX_VALUE));
            final DataValidation dataValidation = helper.createValidation(constraint, cellRangeAddressList);
            dataValidation.setErrorStyle(DataValidation.ErrorStyle.STOP);
            // 输入无效值时是否显示错误框
            dataValidation.setShowErrorBox(true);
            // 验证输入数据是否真确
            dataValidation.setSuppressDropDownArrow(true);
            // 设置无效值时 是否弹出提示框
            dataValidation.setShowPromptBox(true);
            // 设置无效值时的提示框内容 createErrorBox
            dataValidation.createErrorBox("提示", "请输入正整数！！！");
            dataValidation.createPromptBox("提示", "请输入正整数");
            writeSheetHolder.getSheet().addValidationData(dataValidation);
        });

    }
}
```
#### 自定义单个下拉生成的SheetWriteHandler
``` bash
/**
 * @author rentie
 * @date 2023/6/12 13:50
 * @desc 设置单元格下拉
 */
public class DataSelectSheetWriteHandler implements SheetWriteHandler {

    /**
     * Map<下拉框列索引, 下拉对应的字典数据> map
     */
    private Map<Integer, List<String>> selectedMap;

    public DataSelectSheetWriteHandler(final Map<Integer, List<String>> selectedMap) {
        this.selectedMap = selectedMap;
    }

    public DataSelectSheetWriteHandler() {}

    @Override
    public void afterSheetCreate(final WriteWorkbookHolder writeWorkbookHolder,
        final WriteSheetHolder writeSheetHolder) {
        this.selectedMap.forEach((index, dictItemTexts) -> {
            // 设置下拉单元格的首行 末行 首列 末列 65536为excel最大行数
            ExcelUtils.addSelectValidationToSheet(writeSheetHolder, dictItemTexts, index, 1, 6553);
        });
    }
}
```
#### 自定义级联下拉生成的SheetWriteHandler
``` bash
/**
 * @author rentie
 * @date 2023/6/12 15:25
 * @desc 设置单元格级联下拉 使用excel的OFFSET结合MATCH的公式，可以解决名称管理器无法使用特殊字符的问题
 */
@SuppressWarnings({"rawtypes", "unchecked"})
public class DataCascadeSelectSheetWriteHandler implements SheetWriteHandler {

    /**
     * Map<下拉框列索引, 下拉对应的字典数据> map
     */
    private Map<Integer, ExcelSelectDataColumn> selectedMap;

    public DataCascadeSelectSheetWriteHandler(final Map<Integer, ExcelSelectDataColumn> selectedMap) {
        this.selectedMap = selectedMap;
    }

    public DataCascadeSelectSheetWriteHandler() {}

    @Override
    public void afterSheetCreate(final WriteWorkbookHolder writeWorkbookHolder,
        final WriteSheetHolder writeSheetHolder) {

        this.selectedMap.forEach((colIndex, model) -> {
            if (model.getParentColumnIndex() >= 0) {
                // 级联下拉生成
                ExcelUtils.addCascadeValidationToSheet(writeWorkbookHolder, writeSheetHolder,
                    (Map<String, List<String>>)model.getSource(), model.getLevel(), model.getParentColumnIndex(),
                    colIndex, model.getFirstRow(), model.getLastRow());
            } else {
                // 单个下拉
                ExcelUtils.addSelectValidationToSheet(writeSheetHolder, (List<String>)model.getSource(), colIndex,
                    model.getFirstRow(), model.getLastRow());
            }
        });
    }
}
```

#### 注解在DTO中使用例子
``` bash
@Setter
@Getter
@ToString
@EqualsAndHashCode
public class DataDTO implements Serializable {

    private static final long serialVersionUID = 1L;

    /** 级联一级 */
    @ExcelProperty(value = "级联一级", index = 0)
    @ApiModelProperty(value = "级联一级")
    @ExcelCascadeDictAnnotation(dictCode = "achievement_category")
    private String deptName;

    /** 级联二级 */
    @ExcelProperty(value = "级联二级", index = 1)
    @ApiModelProperty(value = "级联二级")
    @ExcelCascadeDictAnnotation(dictCode = "achievement_category", parentColumnIndex = 0,level = 2)
    private String operName;
    
     /** 级联三级 */
    @ExcelProperty(value = "级联三级", index = 2)
    @ApiModelProperty(value = "级联三级")
    @ExcelCascadeDictAnnotation(dictCode = "achievement_category", parentColumnIndex = 1,level = 3)
    private String operName;

    /** 单个下拉 */
    @ExcelDictAnnotation("computer_type")
    @ExcelProperty(value = "单个下拉", index = 3, converter = DictConverter.class)
    @ApiModelProperty(value = "单个下拉", notes = "computer_type")
    private String computerType;
}
```
#### 模版生成方法调用
``` bash
public void download(final HttpServletResponse response) throws IOException {
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding("utf-8");
        // 设置文件名和response参数
        final String fileName = URLEncoder.encode("文件_" + LocalDate.now(), "UTF-8").replaceAll("\\+","%20");
        response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + fileName + ".xlsx");
        // 模板下载
        final List<DataDTO> rows = Lists.newArrayList();
        EasyExcelUtils.writeTemplate(response, rows, DataDTO.class);
    }
```