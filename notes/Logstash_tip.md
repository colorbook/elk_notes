# Logstash Tip
---
## 目的
收錄 Logstash 操作過程中瑣碎的經驗與心得，並附上參考文獻。

## Tip1	
##### 需求
針對單一筆資料需要拆成兩筆資料輸出至Elasticsearch。
例如：
```bash
原始資料：src_ip,src_port,dst_ip,dst_port,protocol
欲拆資料一：src_ip,src_port,protocol
欲拆資料二：dst_ip,dst_port,protocol
```
##### 參考資料
[Filter plugins－clone](https://www.elastic.co/guide/en/logstash/current/plugins-filters-clone.html)
##### 解決方法
LS conf處理資料過程會將每筆資料視為一個event，event處理過程可能會經過input、filter與output，因此必須在這三階段中讓event拆成兩個event做處理。因此使用clone plugin，並利用type做區分。
```bash
[root@ELK]# vim clone.conf
input{
	stdin{ type => "src_data" }
}
filter{
	if [type] == "src_data"{
    	clone{ clones => ["dst_data"] }
        ...
	}
    if [type] == "dst_data"{ ... }
}
```
上述程式碼於input階段事先定義該event的type為src_data，並於filter過程先判斷type為src_data則利用clone plugin去產生新的event，其type為dst_data，後續並針對不同event去做不同的處理。

## Tip2
##### 需求
匯入CSV檔資料，因CSV檔資料第一行為欄位名稱，欲輸出至ES過程中不輸出第一行資料。
例如下述原始CSV檔資料內容，不希望將第一行欄位名稱輸出至ES。
```bash
src_ip,src_port,dst_ip,dst_port,protocol
1.1.1.1,12,2.2.2.2,23,TCP
3.3.3.3,34,4.4.4.4,45,UTP
```
##### 參考資料
[Filter plugins－drop](https://www.elastic.co/guide/en/logstash/current/plugins-filters-drop.html)
##### 解決方法
找出第一行特徵，並使用drop plugin忽略該event。
```bash
[root@ELK]# vim drop.conf
filter{
	csv{
    	columns => [ "src_ip","src_port","dst_ip","dst_port","protocol" ]
    }
	if [src_ip] == "src_ip"{ drop{} }
}
```
上述程式碼於filter階段先利用csv plugin處理各欄位資料，並判斷src_ip欄位值為src_ip時，則忽略處理該event。

## Tip3
##### 需求
指定特定欄位為空值(null)，例如：
```bash
原始資料欄位：src_ip,src_port,dst_ip,dst_port,protocol
資料內容：1.1.1.1,1122,2.2.2.2,80,TCP
```
預將src_port指定為空值，但指定為null輸入至ES後，ES會辨認為字串，實際上並非空值。
##### 參考資料
[Check existence of a field and checking null value in field](https://discuss.elastic.co/t/check-existence-of-a-field-and-checking-null-value-in-field/24968/3)
##### 解決方法
使用ruby plugin指定event欄位資料指定為空值(nil)
```bash
[root@ELK]# null_ruby.conf
...
filter{
	ruby{
    	code => "
        	event['src_port'] = nil
            event['protocol'] = nil
        "
    }
}
```