# Setup ELK on Windows 7
---

## 目的
透過此篇文章，您可以學到以下內容：
* 在 Windows 7 作業系統上安裝 ELK(Elasticsearch, Logstash, Kibana)
* 從 Logstash 匯入資料至 Elasticsearch，並於 Kibana 視覺化資料內容。

## 參考文獻
* [Github－Warning when running any shell commands](https://github.com/elastic/logstash/issues/3087)


## 版本資訊
* OS：Windows 7 Professional
* JAVA：JDK 8u77
* LS：2.2.2
* ES：2.3.1
* Kibana: 4.5.0

## 問題需求
近日發覺網路上 ELK 安裝於 Windows 作業系統相關文章較少，因此我嘗試安裝並簡單記錄安裝過程。

## 解決方法概念
先前已成功安裝 ELK 於 Linux 作業系統，系統基本運作過程順暢，因此我利用先前安裝經驗嘗試套用於 Windows 作業系統，安裝步驟幾乎一樣，可細分為以下步驟：
* JAVA：安裝 JAVA、設定 JAVA_HOME 環境變數
* Logstash 基本安裝：下載、解壓縮、運作測試
* Elasticsearch 基本安裝：下載、解壓縮、設定 host IP、運作測試
* Kibana 基本安裝：下載、解壓縮、設定 ES IP、運作測試

## 解決方法細節
### JAVA
1. 下載 [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，安裝完成後設定JAVA_HOME環境變數，請參考下圖。
	![1_JAVA_Setting.png](../pictures/1_JAVA_Setting.png)

### ELK 基本安裝
1. 下載 LS，解壓縮後做輸出與輸入測試，請參考下圖。
	![2_Logstash_success.png](../pictures/2_Logstash_success.png)

2. 下載 ES，解壓縮後於 elasticsearch.yml 設定檔設定 host IP，請參考下圖。
	![3_Elasticsearch_Setting.png](../pictures/3_Elasticsearch_Setting.png)

3. 安裝 Elasticsearch-head Plugin
	![4_Elasticsearch_HeadPlugin_Setting.png](../pictures/4_Elasticsearch_HeadPlugin_Setting.png)

4. 運行 ES 並確認
	![5_Elasticsearch_Running.png](../pictures/5_Elasticsearch_Running.png)
	
    開啟瀏覽器輸入 http://[Host IP]:9200/_plugin/head 確認 ES 是否正常運作
    ![6_Elasticsearch_HeadPlugin_Running.png](../pictures/6_Elasticsearch_HeadPlugin_Running.png)

5. 下載 Kibana，解壓縮後於 kibana.yml 設定檔設定 ES IP，請參考下圖。
	![7_Kibana_Setting.png](../pictures/7_Kibana_Setting.png)
    
    運行 Kibana
    ![8_Kibana_Running.png](../pictures/8_Kibana_Running.png)
    
    開啟瀏覽器進入輸入 http://[Host IP]:5601 確認 Kibana 是否正常運作
    ![9_Kibana_WebUI.png](../pictures/9_Kibana_WebUI.png)

### ELK 匯入資料
1. 執行 LS 設定檔
	撰寫設定檔內容並執行，相關內容請參考[投影片](../slides/ELK_share_public.html)，執行指令先透過 type 指令將檔案內容輸出(類似 Linux cat 指令)，並透過 pipe 將資料移轉至 LS 處理，因此 LS 設定檔 input 內容為 stdin{}，完整指令請參考下圖。
	![10_Logstash_inputdata.png](../pictures/10_Logstash_inputdata.png)
    
    設定檔 output 部分設定資料匯入 ES 中，並顯示螢幕上。
    ![11_Logstash_inputdata_successfal.png](../pictures/11_Logstash_inputdata_successfal.png)

2. 確認資料是否匯入 ES
	![12_Elasticsearch_HeadPlugin_Catchdata.png](../pictures/12_Elasticsearch_HeadPlugin_Catchdata.png)

3. Kibana 視覺化資料
	Kibana必須先設定 ES index 名稱，下圖為第一季視覺化資料。
    ![13_Kibana_Catchdata.png](../pictures/13_Kibana_Catchdata.png)

### 問題與討論
1. LS Ruby 警告訊息
	因本文預將 CSV 檔資料匯入 ES 中，因此 LS 設定檔 input 會使用 file plugin，但 LS 運作做過程會出現警告訊息 *io/console not supported; tty will not be manipulated*，造成 file plugin 無法運作。經了解，此為 jruby issue，詳細說明請見[官方討論串](https://github.com/elastic/logstash/issues/3087)，後續持續關注此 issue。
