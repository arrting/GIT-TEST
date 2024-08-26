#	API前置作業
## 從Git拉檔案下來
* 從版控取得連結:
  http://129.34.70.108:8080/Repository/Index
  
![image](https://hackmd.io/_uploads/HkcklQXjR.png)

* 取得Git的Url(<font color = red>兩個檔案都取下來!</font>)
![image](https://hackmd.io/_uploads/r1SwJ_NoR.png)
* 對要放取下檔按的資料夾空白處按右鍵，選取"Git Clone"
* 設定要拉下資料的Url，設定完後點"ok"把資料Pull下來

![image](https://hackmd.io/_uploads/B1k1B9toC.png)
![image](https://hackmd.io/_uploads/Bk3Mr9ti0.png)




---

## API測試


* 開啟API專案
![image](https://hackmd.io/_uploads/Hy_LGXQoC.png)
* 點"開始"測試API是否可以執行、確認是否有跳出Swagger頁面
![image](https://hackmd.io/_uploads/SkeF87moA.png)
![image](https://hackmd.io/_uploads/B1sXFMNoC.png)

---

## 取得Token(目前測試的話使用這個)
### Postman JWT設定
* 有兩種取得Token方式
	* 透過執行TokenAPI專案來get token
	* 直接輸入網址執行129.34.69.70上的API取得(<font color = red>目前測試的話使用這個</font>)
1. 填入GetToken的API網址(HTTP Method選擇Post)
	```
	http://129.34.69.70/TokenAPI/v1/api/GetToken
	```
3. Body>raw:填入Json格式userid跟pwd
	```
	{
		"userid":"xxxxx",
		"pwd":"xxxxx"
	}
	```
4. Scripts>Post-response 填入:
`pm.environment.set(JWT',pm.response.json().access_token);`
4. 按"Send" 
![image](https://hackmd.io/_uploads/rJPyEVNoA.png)
5. 取得Token(如下圖紅色區塊)
![image](https://hackmd.io/_uploads/Byi-puVs0.png)

6.將取得的token填入要使用的API Auth
7. 設定完token後測試，以Station API為測試範例:
	* 按下Send後取得JSON
	![image](https://hackmd.io/_uploads/rk051iFiA.png)



<!-- 6. 新增Environment
 `step1:`
![image](https://hackmd.io/_uploads/ryocr44o0.png)
`step2:`
![image](https://hackmd.io/_uploads/ryKTrVNsA.png)
`step3:新增完後回到GetToken，選擇剛剛新增的Environment，然後再點"Send"一次`
`點選右上角圖示確認是否有JWT`
![image](https://hackmd.io/_uploads/BJG7uE4iC.png)
7. 設定完JWT後在使用其他API時的設定
* 如以Station API為測試範例:
	* Auth type -> Bearer Token，Token填入 "<font color = red>{{JWT}}</font>"
	* 按下Send後取得JSON
	* ![image](https://hackmd.io/_uploads/BJJ-FBEsC.png) -->

### 使用Swagger進行測試
同前面方式從Postman取得token
```
Bearer+空格+token
```
![image](https://hackmd.io/_uploads/S1zoNPHj0.png)


---

# API編寫
WebApi2 教學 : http://129.34.69.70/TokenAPI/V1/
> * Controllers=>每支API的執行起點, 主要控制程式流程.
> * Models=>DAL 負責每支API的邏輯.
> * Models=>Public 共用的Function
> * Models=>SQLStr 存放SQL語法, 因應不同的資料庫的SQL字串

##  編寫範例:
* 分別使用兩種API來做範例:
	* Users Api
	* CdcCodeFil Api
* 以API_Example來作為編寫的專案
* 1. 開啟API_Example(開啟後專案如下)
![image](https://hackmd.io/_uploads/Hy2VTPSjR.png)



---

### Users Api
> API說明:
> > 輸入職編
> > 取得是否為院內同工
```
//Response的JSON格式:
{
"isEmployee": true,
"emp_id": "string",
"emp_name": "string"
}
```
1. 新增UserController:
* 	對Controllers資料夾點右鍵->加入->控制器->Web API2控制器->新增
* 	輸入Controller名稱->加入

![image](https://hackmd.io/_uploads/B180xotiC.png)
![image](https://hackmd.io/_uploads/By9OejKo0.png)
![image](https://hackmd.io/_uploads/H1dFboFi0.png)



2. 新增User的Model 
* 	對Models.DAL資料夾點右鍵->加入->類別
* 	輸入Model.DAL名稱->新增
![image](https://hackmd.io/_uploads/SJepJoSsA.png)
3. 新增User的SQLStr
* 	對Models.SQLStr資料夾點右鍵->加入->類別
* 	輸入Model.SQLStr名稱->新增
![image](https://hackmd.io/_uploads/SynngiSjR.png)


---

**4.<font color = blue>Models.DAL.Users(編寫&解釋):</font>**
 Users 的類別，該類別<font color =orange>繼承自 BaseSQL。</font>這個類別主要用來與資料庫進行互動，從資料庫中查詢並返回員工資訊。
 `public class Users : BaseSQL`
 ![image](https://hackmd.io/_uploads/rygW6aBsC.png)


* 先在Models.DAL.Users設定屬性(Properties):
	* 是否為員工
	* 員工編號
	* 員工姓名
	* ` <summary>`:這段 XML 註解的目的是描述各屬性的用途。
	```
	/// <summary>
	/// Y_同工/N_非同工
	/// </summary>
	public bool isEmployee { set; get; }

	/// <summary>
	/// 5碼員編
	/// </summary>
	public string emp_id { set; get; }

	/// <summary>
	/// 同工姓名
	/// </summary>
	public string emp_name { set; get; }
	```
	![image](https://hackmd.io/_uploads/H1QUaTSoA.png)


* 初始化 Users 類別的實例
` public Users() { }`
* 宣告Users 類別的構造函數，用來存儲從數據庫中查詢得到的記錄。
`public Users(DataRow dr) : base(dr) { }`
![image](https://hackmd.io/_uploads/S1HC6TBjC.png)

* FunGetData 方法:
	* 接收一個員工編號 empID 作為參數，並傳回一個 Users 類別的實例
	* 方法內部包含了對資料庫的查詢邏輯，根據傳入的empID尋找對應的使用者資訊。
	```
	public static Users FunGetData(string empID)
	```
	* 	FunGetData方法內部邏輯:
		* 	實例化和資料庫連接設定:
		```
		Users _data = new Users();//用來儲存查詢結果
		BaseDB _db = new BaseDB();//用來進行資料庫操作
		```
		* 	SQL 查詢和參數設定	:

		---

		<font color = red>**SQL原始寫法**</font>
			* 	lst_where 和 lst_para 用來儲存查詢的條件和對應的參數。
		```
		string strSQL = "...";SQL 查詢的字串
		List<string> lst_where = new List<string>();
		List<DbParameter> lst_para = new List<DbParameter>();
		```
		* 	 設定查詢條件	
		```
        lst_where.Add("a.doc_code = ? ");
        st_para.Add(new IfxParameter() { ParameterName = "doc_code", Value = empID });
        ```
		
		* 	查詢資料庫	
		```
		using (DbConnection conn = _db.FunGetConn(PublicLib.HIS))
		{
			conn.Open();
			dt = _db.FunGetData(conn, strSQL, lst_para);
		}
		```
		
		--- 
		<font color = red>**SQL未來寫法**</font>
		```
		strSQL = new UsersStr(_db.database).strSQL; //預備
		BaseDB.DbPara _dbPara = new BaseDB.DbPara(_db.database); //設定資料庫

		if (empID.Trim().Count() == 4)
		{
			_dbPara.FunAdd("a", "doc_code", empID);
		}
		else if (empID.Trim().Count() == 5)
		{
			_dbPara.FunAdd("a", "emp_id", empID);
		}

		strSQL += " and " + string.Join(" and ", _dbPara.listWhere);

		dt = _db.FunGetData(PublicLib.HIS, strSQL, _dbPara.listPara);
		```
		
		* 	資料處理：	
			* 	根據查詢結果填充 Users 類別的屬性。
		```
		if (dt.Rows.Count > 0)
		{
			_data.isEmployee = true;
			_data.emp_id = dt.Rows[0]["emp_id"].ToString().Trim();
			_data.emp_name = dt.Rows[0]["emp_name"].ToString().Trim();
		}
		else
		{
			_data.isEmployee = false;
			_data.emp_id = "";
			_data.emp_name = "";
		}
		```

	
	

**5.<font color = blue>Controllers.UsersController(編寫&解釋)</font>**
![image](https://hackmd.io/_uploads/S1UmJASsR.png)
* 命名空間和引用
```
using API_Example.Models.DAL;
using Dev.API;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Http;

```
* Get 方法
```
[NLogFilter]
[Route("api/Users/{_input}")]
[HttpGet]
[DevSwaggerResponse(typeof(Users))]
public IHttpActionResult Get(string _input)
```
* 	`[NLogFilter]`:使用了 NLog 來記錄日誌
* 	`[Route("api/Users/{_input}")]`:指定了這個 API 的 URL，當請求的路徑為 api/Users/某個輸入值 時，這個方法就會被呼叫。
* 	`[HttpGet]`:表示這是一個 GET 方法，將會處理 GET 請求。
* 	`[DevSwaggerResponse(typeof(Users))]`:用來在 Swagger 文件中標註返回的資料類型 Users
* 	Get 方法內部邏輯:
```
Users _data = Users.FunGetData(_input.Trim());
return Ok(_data);
```
* Users.FunGetData(_input.Trim()) 這行程式碼會根據輸入參數 _input 呼叫 Users 類別的
* return Ok(_data); 將結果 _data 包裝在一個 HTTP 200 OK 的回應中返回

**6.<font color = blue>Models.SQLStr.UsersStr(編寫&解釋)</font>**
![image](https://hackmd.io/_uploads/H1YzKDtiC.png)






---

### CdcCodeFil Api
<font color=red>//未完成</font>

## Tips💡
> 1.程式碼提示和建議功能（IntelliSense）
> :::info
> 可以幫助你寫程式時自動提示、補全、和建議修改程式碼。
> (如下圖)，可以點選燈泡💡，會有建議的修改方式/提示字元可以協助修正。
> :::
> ![image](https://hackmd.io/_uploads/rJ4fjoSjR.png)
> 
> ![image](https://hackmd.io/_uploads/HkeZioHsR.png)

> 2. visual studio 格式化文件:
> :::info
> 可以幫助你自動整理代碼的縮進、空格和其他格式，使代碼更易讀。
> :::
> ![image](https://hackmd.io/_uploads/HyKXVpSj0.png)

> 3. 註解 : 
> :::info
> 編寫程式碼時，在方法、屬性或類別前面輸入三個斜線<font color = red> " /// "</font>時，會自動生成 XML 註解的模板，如下所示：
> :::
> 
> 	```
> 	/// <summary>
> 	/// 
> 	/// </summary>
> 	```
> :::info
> 這樣的註解有助於讓其他開發者理解程式碼，還可以在自動生成的文件（例如 IntelliSense 說明(swagger所引用)）中提供說明文字。
> :::








