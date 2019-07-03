[<img src="../../index.jpg" width = "80" height = "80"  />](../../index.md#index)

<h1 id="exceldb">SpringMVC实现上传Excel文件并读取至数据库</h1>

**1、添加依赖**

```xml
		<!-- Excel POI -->
		<dependency>
       		<groupId>org.apache.poi</groupId>
       		<artifactId>poi-ooxml</artifactId>
       		<version>3.9</version>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.3</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.6</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.17</version>
		</dependency>
		<dependency>
			<groupId>org.apache.xmlbeans</groupId>
			<artifactId>xmlbeans</artifactId>
			<version>2.6.0</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml-schemas</artifactId>
			<version>3.17</version>
		</dependency>

```

**2、添加WDWUtil工具类，用于判断Excel 版本**

```java
public class WDWUtil {
	/**
	 * 描述：是否是2003的Excel，返回true是2003
	 * 
	 * @param filePath
	 * @return
	 */
	public static boolean isExcel2003(String filePath) {
		return filePath.matches("^.+\\.(?i)(xls)$");
	}

	/**
	 * 描述：是否是2007的Excel，返回true是2007
	 * 
	 * @param filePath
	 * @return
	 */
	public static boolean isExcel2007(String filePath) {
		return filePath.matches("^.+\\.(?i)(xlsx)$");
	}
}
```

**3、创建BookEntity实体类**

```java
public class BookEntity {
	private Integer bookId;//图书编号
	private String bookName;//图书名称
	private String bookAuthor;//图书作者
	private Float bookPrice;//图书单价
	private Integer bookNum;//图书数量

	public BookEntity() {
		super();
		// TODO Auto-generated constructor stub
	}

	get/set省略。。。

	@Override
	public String toString() {
		return "BookEntity [bookId=" + bookId + ", bookName=" + bookName + ", bookAuthor=" + bookAuthor + ", bookPrice="
				+ bookPrice + ", bookNum=" + bookNum + "]";
	}

}
```

**4、创建ReadExcelUtil工具类，用于读取Excel**

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.springframework.web.multipart.MultipartFile;

import com.desire.entity.BookEntity;

/**
 * 用于读取Excel实体类
 * 
 * @author Administrator
 *
 */
public class ReadExcelUtil {

	// 总行数
	private int totalRows = 0;
	// 总条数
	private int totalCells = 0;
	// 错误信息接收器
	private String errorMsg;

	public int getTotalRows() {
		return totalRows;
	}

	public int getTotalCells() {
		return totalCells;
	}

	public String getErrorMsg() {
		return errorMsg;
	}

	/**
	 * 验证Excel文件
	 * 
	 * @param filePath
	 * @return
	 */
	public boolean validateExcel(String filePath) {
		if (filePath == null || !(WDWUtil.isExcel2003(filePath) || WDWUtil.isExcel2007(filePath))) {
			errorMsg = "文件名不是Excel格式";
			return false;
		}
		return true;
	}

	/**
	 * 读取Excel文件，获取书本信息集合
	 * 
	 * @param fileName
	 * @param mFile
	 * @return
	 */
	public List<BookEntity> getExcelRead(String fileName, MultipartFile mFile) {
		// 文件名
		fileName = mFile.getOriginalFilename();
		// 后缀名
		String suffxName = fileName.substring(fileName.lastIndexOf("."));
		// 上传后的路径
		String filePath = "D://fileupload//";
		// 新建一个文件
		String newFileName = new Date().getTime() + suffxName;
		File dest = new File(filePath + newFileName);
		// 创建一个目录（它的路径名由当前File对象指定，包括任一必须的父路径）
		if (!dest.getParentFile().exists())
			dest.getParentFile().mkdirs();
		// 将上传的文件写入新建的文件中
		try {
			mFile.transferTo(dest);
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}

		// 初始化书本信息的集合
		List<BookEntity> bookList = new ArrayList<>();
		// 初始化输入流
		InputStream is = null;
		try {
			// 验证文件名是否合格
			if (!validateExcel(fileName)) {
				return null;
			}
			// 根据文件名判断文件时2003版本还是2007版本
			boolean isExcel2003 = true;
			if (WDWUtil.isExcel2007(fileName)) {
				isExcel2003 = false;
			}
			// 根据新建的文件实例化输入流
			is = new FileInputStream(dest);
			// 根据Excel里面的内容读取书本信息
			bookList = getExcelInfo(is, isExcel2003);
			is.close();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (is != null) {
				try {
					is.close();
				} catch (IOException e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}
		}
		return bookList;
	}

	/**
	 * 根据Excel里面的内容读取客户信息
	 * 
	 * @param is          输入流
	 * @param isExcel2003 isExcel2003 Excel是2003还是2007版本
	 * @return
	 * @throws IOException
	 */
	private List<BookEntity> getExcelInfo(InputStream is, boolean isExcel2003) {
		List<BookEntity> bookList = null;
		try {
			// 根据版本选择创建WorkBook的方式
			Workbook wb = null;
			// 当Excel时2003时
			if (isExcel2003) {
				wb = new HSSFWorkbook(is);
			} else {
				// 当Excel是2007时
				wb = new XSSFWorkbook(is);
			}
			// 读取Excel里面书本的信息
			bookList = readExcelValue(wb);
		} catch (IOException e) {
			// TODO: handle exception
			e.printStackTrace();
		}
		return bookList;
	}

	/**
	 * 读取Excel里面客户的信息
	 * 
	 * @param wb
	 * @return
	 */
	private List<BookEntity> readExcelValue(Workbook wb) {

		// 得到第一个Sheel
		Sheet sheet = wb.getSheetAt(0);

		// 得到Excel的行数
		this.totalRows = sheet.getPhysicalNumberOfRows();

		// 得到Excel的列数（前提是有行数）
		if (totalRows >= 1 && sheet.getRow(0) != null) {
			this.totalCells = sheet.getRow(0).getPhysicalNumberOfCells();
		}

		List<BookEntity> bookList = new ArrayList<>();
		BookEntity bookEntity;
		// 循环Excel行数，从第二行开始。标题不入库
		for (int r = 1; r < totalRows; r++) {
			Row row = sheet.getRow(r);
			if (row == null) {
				continue;
			}
			bookEntity = new BookEntity();
			System.err.println("列数：" + this.totalCells);
			// 循环Excel的列
			for (int c = 0; c < this.totalCells; c++) {
				Cell cell = row.getCell(c);
				if (null != cell) {
					if (c == 0) {
						int bookId = (int) (cell.getNumericCellValue());
						System.out.println(bookId);
						bookEntity.setBookId(bookId);
					} else if (c == 1) {
						String bookName = cell.getStringCellValue();
						System.out.println(bookName);
						bookEntity.setBookName(bookName);
					} else if (c == 2) {
						String bookAuthor = cell.getStringCellValue();
						System.out.println(bookAuthor);
						bookEntity.setBookAuthor(bookAuthor);
					} else if (c == 3) {
						float bookPrice = (float) (cell.getNumericCellValue());
						System.out.println(bookPrice);
						bookEntity.setBookPrice(bookPrice);
					} else if (c == 4) {
						int bookNum = (int) (cell.getNumericCellValue());
						System.out.println(bookNum);
						bookEntity.setBookNum(bookNum);
					}
				}
			}
			// 添加书本
			bookList.add(bookEntity);
		}
		return bookList;
	}
}

```

**5、创建数据层**

*BookMapping.xml*

```xml
<!-- 单个添加 -->
<insert id="addBook" parameterType="com.desire.entity.BookEntity">
	insert into BOOK values(null,#{bookName},#{bookAuthor},#{bookPrice},#{bookNum})
</insert>
<!-- 批量添加 -->
<insert id="insertByForeachTag" parameterType="List">
	insert into BOOK
	VALUES
	<foreach collection="bookList" item="book" separator=",">
		(null,#{book.bookName},#{book.bookAuthor},#{book.bookPrice},#{book.bookNum})
	</foreach>
</insert>
```

*IBookMapper.java*

```java
@Repository
public interface IBookMapper {

	//添加
	void addBook(BookEntity bookEntity);

	//批量添加
	void insertByForeachTag(@Param("bookList") List<BookEntity> bookList);
}
```

**6、创建服务层**

*IBookService.java*

```java
public interface IBookService {

	// 添加
	void addBook(BookEntity bookEntity);

	// 批量添加
	void insertByForeachTag(List<BookEntity> bookList);
}
```

*BookServiceImpl.java*

```java
@Service
public class BookServiceImpl implements IBookService {

	@Autowired
	private IBookMapper bookMapper;

	@Override
	public void addBook(BookEntity bookEntity) {
		bookMapper.addBook(bookEntity);
	}

	@Override
	public void insertByForeachTag(List<BookEntity> bookList) {
		bookMapper.insertByForeachTag(bookList);
	}
}
```

*IUploadService.java*

```java
public interface IUploadService {

	public boolean batchImport(String name, MultipartFile file);
}
```

*UploadServiceImpl.java*

```java
@Service
public class UploadServiceImpl implements IUploadService {

	@Autowired
	private IBookService bookService;

	@Override
	public boolean batchImport(String name, MultipartFile file) {
		boolean b = false;
		// 创建处理Excel
		ReadExcelUtil readExcel = new ReadExcelUtil();
		// 解析Excel，获取书本信息集合
		List<BookEntity> bookList = readExcel.getExcelRead(name, file);
		System.out.println(bookList);
		if (bookList != null) {
			b = true;
		}
		// ①：迭代添加书本信息（注：实际上这里也可以直接将bookList集合作为参数，在MyBatis的相应映射文件中使用foreach标签进行批量添加）
//		for (BookEntity bookEntity : bookList) {
//			System.out.println(bookEntity.toString());
//			bookService.addBook(bookEntity);
//		}
		// ②：直接将bookList集合作为参数，在MyBatis的相应映射文件中使用foreach标签进行批量添加
		bookService.insertByForeachTag(bookList);
		return b;
	}
}
```

**7、创建控制层**

*FileUploadController.java*

```java
import java.io.UnsupportedEncodingException;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import com.desire.service.IUploadService;

@RestController
@RequestMapping("/piliang")
public class FileUploadController {

	@Autowired
	private IUploadService uploadService;

	@RequestMapping(value = "batchimport", method = RequestMethod.POST)
	public String batchimport(@RequestParam("file") MultipartFile file, HttpServletRequest request,
			HttpServletResponse response) {
		try {
			request.setCharacterEncoding("UTF-8");
			response.setCharacterEncoding("UTF-8");
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		String msg = "";
		// 判断文件是否为空
		if (file == null) {
			return null;
		}
		// 获取文件名
		String fileName = file.getOriginalFilename();
		// 进一步判断文件是否为空（即判断其大小是否为0或其名称是否为null）
		long size = file.getSize();
		if (fileName == null || ("").equals(fileName) && size == 0) {
			return null;
		}
		// 批量导入。参数：文件名，文件
		boolean b = uploadService.batchImport(fileName, file);
		if (b) {
			msg = "import success!";
			request.getSession().setAttribute("msg", msg);
		} else {
			msg = "import failed!";
			request.getSession().setAttribute("msg", msg);
		}
		return msg;
	}
}
```

