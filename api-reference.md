# Topics



## Authentication

- 無し
- 現段階で、質問の投稿・取得は認証を掛ける必要は無いと考えています



## Base URL

```
https://api.qicoo.xx/
```



**todo**

- EKSのLoadBalancerでhttpsにする方法
- EKSのLoadBalancerで任意のDomainを指定する方法
- ドメイン名の取得 (AWSで取得するとよいかな)
- アプリ名の決定
- SSL証明書の準備 (AWSを利用すると無料)



## Cookie



![Cookie Flow](https://github.com/Sugi275/qa-apps-docs/blob/master/1346_7081.png "サンプル")

Client側にsession idを保存するため、Cookieを利用します。Cookieの中に「user_session:hogehoge」という形で session id を保存します。

session id は主にユーザーの識別に利用します。



## Errors

一般的なREST APIのステータスコードを仮採用。問題がありそうなら検討

| Code                               | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| 200 - OK                           | Everything worked as expected.                               |
| 400 - Bad Request                  | The request was unacceptable, often due to missing a required parameter. |
| 401 - Unauthorized                 | No valid API key provided.                                   |
| 402 - Request Failed               | The parameters were valid but the request failed.            |
| 404 - Not Found                    | The requested resource doesn't exist.                        |
| 409 - Conflict                     | The request conflicts with another request (perhaps due to using the same idempotent key). |
| 429 - Too Many Requests            | Too many requests hit the API too quickly. We recommend an exponential backoff of your requests. |
| 500, 502, 503, 504 - Server Errors | Something went wrong on Stripe's end. (These are rare.)      |



## Pagination

list取得系のAPIは、基本的にページ指定を有効にします。ページ指定に以下の3つのパラメータを使用します。

取得したObject(questionsなど)には必ず ID が割り当てられています。



パラメータ

| Parameter      | Description                                                  | default | value                 |
| -------------- | ------------------------------------------------------------ | ------- | --------------------- |
| limit          | 取得するObjectの上限数を指定します。defaultで10です。1~100の間で指定することが出来ます。 | 10      | 1 ~ 100               |
| starting_after | ページ指定のカーソルとして使用します。starting_afterにIDを指定すると、指定した IDの次のObjectから始まるデータを取得します。 | none    | id                    |
| ending_before  | ページ指定のカーソルとして使用します。ending_beforeにIDを指定すると、指定した IDの前のObjectで終わるデータを取得します。 | none    | id                    |
| sort           | ソートで使用する値を指定します。ソートで利用できる値は、Objectに依存します。 | none    | value                 |
| order          | ソートで使用する順序を指定します。                           | asc     | 昇順(asc), 降順(desc) |



### Example Request 

curlの例

```
curl https://api.qicoo.xx/v1/questions?event=jkd1804&program=1&limit=3&starting_after=hoge&sort=create_at&order=asc
```



### Example Response

```
todo update
```







# Core Objects

## Questions 

QuestionsというObjectは、イベントや勉強などでユーザーから投稿される質問内容を表しています。



### The Question Object

パラメータ

| Parameter  | Description                                 | Type                  |
| ---------- | ------------------------------------------- | --------------------- |
| id         | Objectのid                                  | String                |
| object     | Objectの種類。"question"                    | String                |
| username   | 質問した人の名前                            | String                |
| comment    | 質問内容                                    | String                |
| created_at | 作成時間。 例 : "2018-09-08T09:07:09+09:00" | Timestamp (UTC+09:00) |
| updated_at | 更新時間。 例 : "2018-09-08T09:07:09+09:00" | Timestamp (UTC+09:00) |
| like       | イイネ！の数                                | integer               |



### Create a Question

質問を作成します。



URL

```
POST /v1/questions
```



HTTP Header

- Content-Type : application/json を指定します
- Cookie : Client側に保存されたCookieに user_session が保存されているので、これを指定します。Cookieが指定されていない場合は、username を Anonymous として扱います。

```
-H "Content-Type: application/json" -H 'Cookie: user_session=hogehoge'
```



パラメータ

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| event     | イベント(例 : jkd,  cndjp)のid                               |
| program   | プログラム(例 : spinnaker触ってみた, slafford触ってみた)のid |
| comment   | 質問内容                                                     |



Request 例

```
curl -H "Content-Type: application/json" -H 'Cookie: user_session=hogehoge' -X POST https://api.qicoo.xx/v1/questions -d '
{
  "event": "jkd1812",
  "program": "1",
  "comment": "kubernetesの〇〇について教えてください！"
}
'
```



Return 例

QuestionResourceがReturnされます

```
{
  "id": "BosWT9EsdzgjPn",
  "object": "question",
  "username": "sugimount",
  "event": "jkd1812",
  "program": "1",
  "comment": "kubernetesの〇〇について教えてください！",
  "created_at": "2018-09-08T09:07:09+09:00",
  "updated_at": "2018-09-08T09:07:09+09:00",
  "like": 0
}
```



### Retrieve a Question

1つの質問を取得します。



URL 例

```
GET /v1/questions/BosWT9EsdzgjPn
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter | Description |
| --------- | ----------- |
| なし      | なし        |



Request 例

```
curl -H "Content-Type: application/json" -X GET https://api.qicoo.xx/v1/questions/BosWT9EsdzgjPn
```



Return 例

QuestionResourceがReturnされます

```
{
  "id": "BosWT9EsdzgjPn",
  "object": "question",
  "username": "sugimount",
  "event": "jkd1812",
  "program": "1",
  "comment": "kubernetesの〇〇について教えてください！",
  "created_at": "2018-09-08T09:07:09+09:00",
  "updated_at": "2018-09-08T09:07:09+09:00",
  "like": 0
}
```







### Delete a Question

質問を削除します。

URL

```
DELETE /v1/questions/BosWT9EsdzgjPn
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter | Description |
| --------- | ----------- |
| なし      | なし        |



Request 例

```
curl -H "Content-Type: application/json" -X DELETE https://api.qicoo.xx/v1/questions/BosWT9EsdzgjPn
```



Return 例

削除された場合は、deleted が true となります

```
{
  "id": "BosWT9EsdzgjPn",
  "object": "question",
  "deleted": true
}
```





### List all Questions

全ての質問を取得します。



URL

```
GET /v1/questions
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter      | Description                                                  | default | Type     |
| -------------- | ------------------------------------------------------------ | ------- | -------- |
| limit          | 取得するObjectの上限数を指定します。defaultで10です。1~100の間で指定することが出来ます。 | 10      | interger |
| starting_after | ページ指定のカーソルとして使用します。starting_afterにIDを指定すると、指定した IDの次のObjectから始まるデータを取得します。 | none    | String   |
| ending_before  | ページ指定のカーソルとして使用します。ending_beforeにIDを指定すると、指定した IDの前のObjectで終わるデータを取得します。 | none    | String   |
| sort           | ソートで使用する値を指定します。ソートで利用できる値は、"created_at", "like" | none    | String   |
| order          | ソートで使用する順序を指定します。昇順(asc), 降順(desc)      | asc     | String   |
| event          | eventidを指定 "jkd1812"                                      | none    | String   |
| program        | programidを指定 "1"                                          | none    | String   |



Request 例

```
curl -H "Content-Type: application/json" -X GET https://api.qicoo.xx/v1/questions?event=jkd1812&program=1&limit=2&sort=created_at&order=asc
```



Return 例

QuestionResourceがReturnされます

```
{
  "object": "list",
  "data": [
    {
      "id": "BosWT9EsdzgjPn",
      "object": "question",
      "username": "sugimount",
      "event": "jkd1812",
      "program": "1",
      "comment": "kubernetesの〇〇について教えてください！",
      "created_at": "2018-09-08T09:07:09+09:00",
      "updated_at": "2018-09-08T09:07:09+09:00",
      "like": 0
    },
    {
      "id": "IOdeamop2243v",
      "object": "question",
      "username": "hhiroshell",
      "event": "jkd1812",
      "program": "1",
      "comment": "世界を平和にする方法を教えてください！",
      "created_at": "2018-09-08T09:09:09+09:00",
      "updated_at": "2018-09-08T09:09:09+09:00",
      "like": 0
    }
  ]
}
```







## Users

sessionidとuserを紐づけるObjectです。



### The User Object

パラメータ

| Parameter    | Description                                                 | Type                  |
| ------------ | ----------------------------------------------------------- | --------------------- |
| id           | Objectのid                                                  | String                |
| object       | Objectの種類。"user"                                        | String                |
| user_session | session id                                                  | String                |
| username     | userの名前                                                  | String                |
| type         | userの種類 "twitter"                                        | String                |
| created_at   | 作成時間。 例 : "2018-09-08T09:07:09+09:00"                 | Timestamp (UTC+09:00) |
| updated_at   | 更新時間。 例 : "2018-09-08T09:07:09+09:00"                 | Timestamp (UTC+09:00) |
| user_url     | userが "twitter"の場合、twitter userページのURLが格納される | String                |



### Create a User

sessionとuserを紐づけるObjectを作成します。



URL

```
POST /v1/users
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter | Description                                |
| --------- | ------------------------------------------ |
| username  | userの名前                                 |
| type      | userのtype "twitter" など                  |
| user_url  | userに関連するURL。twitterの個人ページなど |



Request 例

```
curl -H "Content-Type: application/json" -X POST https://api.qicoo.xx/v1/users -d '
{
  "username": "sugimount",
  "type": "twitter",
  "user_url": "https://twitter.com/sugimount"
}
'
```



Return 例

QuestionResourceがReturnされます

```
{
  "id": "IOodngp39890ba",
  "object": "user",
  "user_session": "hogehoge",  
  "username": "sugimount",
  "type": "twitter",
  "created_at": "2018-09-08T09:07:09+09:00",
  "updated_at": "2018-09-08T09:07:09+09:00",
  "user_url": "https://twitter.com/sugimount"
}
```





### Retrieve a User

1つのユーザー情報を取得します。



URL 例

```
GET /v1/users/IOodngp39890ba
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter | Description |
| --------- | ----------- |
| なし      | なし        |



Request 例

```
curl -H "Content-Type: application/json" -X GET https://api.qicoo.xx/v1/users/IOodngp39890ba
```



Return 例

QuestionResourceがReturnされます

```
{
  "id": "IOodngp39890ba",
  "object": "user",
  "user_session": "hogehoge",  
  "username": "sugimount",
  "type": "twitter",
  "created_at": "2018-09-08T09:07:09+09:00",
  "updated_at": "2018-09-08T09:07:09+09:00",
  "user_url": "https://twitter.com/sugimount"
}
```









### Delete a User

ユーザー情報を削除します。

URL

```
DELETE /v1/users/IOodngp39890ba
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter | Description |
| --------- | ----------- |
| なし      | なし        |



Request 例

```
curl -H "Content-Type: application/json" -X DELETE https://api.qicoo.xx/v1/users/IOodngp39890ba
```



Return 例

削除された場合は、deleted が true となります

```
{
  "id": "IOodngp39890ba",
  "object": "users",
  "deleted": true
}
```



### List all Users

全てのユーザーを取得します。



URL

```
GET /v1/users
```



HTTP Header

- Content-Type : application/json を指定します

```
-H "Content-Type: application/json"
```



パラメータ

| Parameter      | Description                                                  | default | Type     |
| -------------- | ------------------------------------------------------------ | ------- | -------- |
| limit          | 取得するObjectの上限数を指定します。defaultで10です。1~100の間で指定することが出来ます。 | 10      | interger |
| starting_after | ページ指定のカーソルとして使用します。starting_afterにIDを指定すると、指定した IDの次のObjectから始まるデータを取得します。 | none    | String   |
| ending_before  | ページ指定のカーソルとして使用します。ending_beforeにIDを指定すると、指定した IDの前のObjectで終わるデータを取得します。 | none    | String   |
| sort           | ソートで使用する値を指定します。ソートで利用できる値は、"created_at" | none    | String   |
| order          | ソートで使用する順序を指定します。昇順(asc), 降順(desc)      | asc     | String   |



Request 例

```
curl -H "Content-Type: application/json" -X GET https://api.qicoo.xx/v1/users?&limit=2&sort=created_at&order=asc
```



Return 例

QuestionResourceがReturnされます

```
{
  "object": "list",
  "data": [
    {
      "id": "IOodngp39890ba",
      "object": "user",
      "user_session": "hogehoge",  
      "username": "sugimount",
      "type": "twitter",
      "created_at": "2018-09-08T09:07:09+09:00",
      "updated_at": "2018-09-08T09:07:09+09:00",
      "user_url": "https://twitter.com/sugimount"
    },
    {
      "id": "Gdi4235niopan",
      "object": "user",
      "user_session": "gugehuge",  
      "username": "hhiroshell",
      "type": "twitter",
      "created_at": "2018-09-08T09:08:09+09:00",
      "updated_at": "2018-09-08T09:08:09+09:00",
      "user_url": "https://twitter.com/hhiroshell"
    }
  ]
}
```

