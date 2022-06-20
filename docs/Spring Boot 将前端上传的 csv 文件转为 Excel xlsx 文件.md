## 一、概述
本地文件是 `csv`格式，要先将其转成 `xlsx`。
## 二、实现步骤
### 2.1 获取 csv 文件数据
由于前端的上传文件的数据类型是 `MultipartFile`，所以就通过获取输入流的方式读取到 `csv` 数据。
将所有 `csv`数据一行一行的放入到 `ArrayList` 动态数组中。
```java
    /**
     * 获取 csv 数据
     * @author 云胡
     * @param multipartFile
     * @throws IOException
     */
    public ArrayList<String[]> getCsvDataList(MultipartFile multipartFile) throws IOException{
        // 存放所有的 csv 文件数据
        ArrayList<String[]> csvDataList = new ArrayList<>();

        // 文件的编码，这里设为 utf-8
        CsvReader reader = new CsvReader(multipartFile.getInputStream(), ',', Charset.forName("utf-8"));

        // 获取数据
        while (reader.readRecord()) {
            csvDataList.add(reader.getValues());
        }
        // 关闭
        reader.close();
        return csvDataList;
    }
```
### 2.2 生成 Excel 文件
```java
	/**
     * 生成 Excel 文件
     * @param multipartFile
     * @return 生成的 Excel 文件名
     * @throws IOException
     */
    public String csvToExcelFile(MultipartFile multipartFile) throws IOException {
        // 获取 csv 数据
        ArrayList<String[]> csvDataList = getCsvDataList(multipartFile);
        if(csvDataList.isEmpty())
        {
            // 没有数据
            return "";
        }

        // 获取拥有后缀的文件名 「abc.csv」
        String csvFileNameHaveSuffix = multipartFile.getOriginalFilename();
        // 获取不带后缀的文件名 「abc」
        String csvFileName = csvFileNameHaveSuffix.substring(0, csvFileNameHaveSuffix.indexOf("."));

        // 创建一个 xlsx 工作簿
        XSSFWorkbook workbook = new XSSFWorkbook();

        String outputExcelFileName = csvFileName + ".xlsx";
        FileOutputStream out = new FileOutputStream(new File(outputExcelFileName));

        // 创建工作表
        XSSFSheet spreadsheet = workbook.createSheet("Sheet1");

        // 将 csv 数据存到 xlsx 文件中
        for (int rowNum = 0; rowNum < csvDataList.size(); rowNum++) {
            // 获取一行的数据
            String[] csvFileRowData = csvDataList.get(rowNum);
            XSSFRow row = spreadsheet.createRow(rowNum);
            for (int columnNum = 0; columnNum < csvFileRowData.length; columnNum++) {
                XSSFCell cell = row.createCell(columnNum);
                cell.setCellValue(csvFileRowData[columnNum]);
            }
        }

        workbook.write(out);
        out.close();
        return outputExcelFileName;
    }
```

有了 `xlsx` 的文件名，`EasyExcel` 就可以读取了。
## 


