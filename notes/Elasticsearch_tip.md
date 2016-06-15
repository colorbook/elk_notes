# Elasticsearch Tip
---
## 目的
收錄 Elasticsearch 操作過程中瑣碎的經驗與心得，並附上參考文獻。

## Tip1	
##### 需求
制定template，供特定index使用。
##### 參考資料
[Indices APIs－Index Templates](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates.html)
##### 解決方法
ES有index template觀念，目的是設定供特定index套用該template內容，template主要可分為兩區塊：
```bash
{
	"template": "demo*",
    "order": 1,
    "setting: {...}",
    "mappings": {...}
}
```
1. 上述template格式為JSON，於第一層template指定未來index名稱開頭為demo時，則套用該template內容。
2. order代表template套用的優先順序，當index名稱符合多個template時，ES系統會先從order數值小的template開始套用，依序套用至order數值大的template，在套用過程中，假使template設定值改變，則以order數值大的內容複寫設定值。
3. setting代表可以設定meta-fields相關內容
4. mapping代表可以設定index內，type與各欄位的相關內容

##### 操作
1. 設定或更新template
```bash
PUT /_template/demo_1
{
	"template": "demo*",
    "order": 1,
    "setting: {...}",
    "mappings": {...}
}
```
利用PUT method與_template API定義template名稱為demo_1，並輸入template內容。此法也可做為template更新，但每次更新需完整輸入template內容，並非更新單一設定值就只輸入單一設定值內容。
2. 讀取template
```bash
GET /_template
```
ES template預設為logstash
3. 刪除template
```bash
DELETE /_template/demo_1
```

## Tip2
##### 需求
Logstash conf目前只支援欄位資料型態為integer、float、string與boolean，因此要進階更改index內欄位資料型態(例如：ip)必須從ES mapping下手。
##### 參考資料
[Filter Plugins－mutate－convert](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-convert)
[Mapping－field datatypes](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)

##### 解決方法
於template的mappings設定欄位資料型態
```bash
PUT /_template/demo_1
{
	"template": "demo*",
    "order": 1,
    "setting: {...}",
    "mappings": {
    	type_1: {
        	"properties": {
            	"src_ip": { "type": "ip" },
                "src_port": { "type": "interger" },
                "url": { "type": "string", "index": "not_analyzed" },
                "origin_datetime": { "type": "date" }
            }
        }
    }
}
```
上述template於mappings區段設定四個欄位內容，相關說明如下：
1. type_1表示type名稱為type_1下的設定
2. src_ip欄位資料形式設定為ip類型，預設ES會將各欄位預設為string，這會造成往後分析時(Kibana)，無法利用資料型態特性去做分析(例如：數字計算、日期區分、檢視ip範圍、地圖映射)。
3. src_port欄位資料形式設定為interger類型
4. url欄位資料形式設定為string類型，ES預設會將各string做斷詞斷句並做index，因此以url欄位值可能會被斷詞成許多字串做index，爾後會造成分析上的困擾(例如：於Kibana上分析會出現來自於同url的不同字串，原本資料為demo-data，晶斷詞後會分為demo與data，因此Kibana要統計url時會分別計算demo與data出現的次數，造成統計無意義。)因此在此利用index參數設定not_analyzed，讓ES不要針對該欄位值做index處理，所以index會保留原始字串。
5. origin_datetime欄位資料形式設定為date類型