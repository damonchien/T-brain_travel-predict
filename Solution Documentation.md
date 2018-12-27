# 旅遊訂單成行預測

* 隊伍：試一下
* Best Submission (AUC)：
  * Public Score：0.7446609257
  * Private Score：0.752072301
* Private Leaderboard Rank：6 / 733

## 摘要

基本上使用的只有 order.csv , group.csv 兩個資料集， airline.csv , day_schedule.csv 則沒有使用。
其中 group.csv 的部分對 promotion_prog , product_name兩個文字特徵做 Label Encoder，其餘的也大概是這樣的處理方式。
剩餘的 order.csv 只對其中跟時間有關的特徵個別做了很多萃取，如 order_date , begin_date , return_date (年、月、日、季節等)
	
接下來就直接跑一遍lightgbm尋找較有效的特徵，且也剛好都是數值的特徵 predays , price 找到之後做了很多 group ，group其他類別類型的特徵，
算上述兩個的 mean , median , diff ，並且再跑一次lightgbm再刪除不重要的特徵，並以此重複做個兩三次，跑到 valid set 的AUC不再有明顯上升。
最後特徵數大概有1421個，其中一些 missing value 只做了使用相同類別的平均填入
	
最後為了再上升一點分數，做了 Staking，所使用的為六個lightgbm再接上一個lightgbm，其中的參數部分只有做手動的網格搜索一下。

## 環境

* [系統平台] Windows 10 (i5 CPU, 8GB RAM)
* [程式語言] Python 3.6.5 
* [函式庫] numpy , pandas , sklearn, lightgbm

## 特徵

所使用的特徵就是原有的幾個特徵 

	['group_id', 'sub_line', 'area', 'days', 'unit', 'people_amount', 'source_1', 'source_2', 'promotion_prog', 'product_name', 'order_date_month', 'order_date_dayofyear', 'order_date_weekofyear', 'order_date_dayofmonth', 'order_date_dayofweek', 'begin_date_dayofweek', 'begin_date_dayofyear', 'begin_date_dayofmonth', 'begin_date_weekofyear', 'begin_date_month', 'return_date_dayofweek', 'return_date_dayofyear', 'return_date_dayofmonth', 'return_date_weekofyear', 'return_date_month', 'begin_date_from2016', 'order_date_from2016', 'ticketsischeap_days', 'ticketsischeap_week', 'ticketsisexpen_week', 'order_holiday', 'begin_date_quarter' ]
	
對predays，price 做統計分析，並且再嘗試以上的組合，兩兩一組，三三一組，四四一組

## 訓練模型

### LightGBM Model 參數 (最後一層的參數)

	 'boosting_type': 'dart'
	 'objective': 'binary'
	 'metric': 'auc'
	 'learning_rate': 0.05
	 'num_leaves': 15
	 'max_depth': 4
	 'feature_fraction': .9
	 'bagging_fraction':0.6
	 'bagging_freq':10
	 'min_data_in_leaf': 100
	 'reg_alpha': 10
	 'reg_lambda': 10
* 其他六個模型參數皆在 Staking.ipynb

## 訓練方式及原始碼

請參考 [README](https://github.com/damonchien/T-brain-predictions/blob/master/README.md)

## 結論

特徵工程部分可以再修正許多，加入更多idea產生更多特徵。
資料集部分沒有完全完整的利用，
顧客的個人資料再加進來也會相信也會有許多幫助。
硬體上限制導致中間刪除特徵不確定有沒有影響最終結果。
綜上因素，認為準確分數集所有人的結果可能可以達八成。
欠缺EDA部分，日後可以再加強。

### 以下是個人模型認為較重要之單位特徵因素(組合)

* 'predays’'
* 'order_date_dayofweek',
* 'unit',
* 'source_1',
* 'people_amount',
* 'source_2',
* 'group_id',
* 'return_date_dayofweek'
