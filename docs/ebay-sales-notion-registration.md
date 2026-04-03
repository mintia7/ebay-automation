# eBay売上情報 Notion DB登録手順書（公開用・マスキング版）

## 概要
メルカリでの仕入れ（直近N件）とeBayでの販売（直近N件）を確認し、Notionの在庫管理DBと海外販売管理DBに登録する定期タスク。

- N はユーザーが毎回指定する（例:「直近3件を登録して」）
- 指定がない場合はユーザーに件数を確認する

---

## Notion DB 定義

### データソース
※公開用のためIDはマスキング

| DB名 | データソース |
|------|-------------|
| トレカ在庫管理DB | <NotionデータソースID> |
| トレカ海外販売管理DB | <NotionデータソースID> |

---

## SKU命名規則

形式
pYYMMDD-NNN

日付
eBay販売日（バイヤーが購入した日）

NNN
当日内連番（001開始、販売時刻が早い順）

採番前に既存SKUを検索して重複がないことを確認する。

※
eBay Custom Label SKU（例: 260308001m）とは別物
eBay側は出品日ベース

---

## 実行フロー

### Step1 メルカリ情報取得

メルカリ購入履歴から直近N件を確認し、以下の情報を取得する

取得項目

- 商品名
- 商品代金
- 購入日時
- 商品ID（取引URL）

取得後、ユーザーに以下を確認する

仕入決済方法

例

- PayPayゴールド
- セゾンAMEXプラチナ
- その他

---

### Step2 eBay情報取得

eBayの注文履歴から直近N件を確認し、以下の情報を取得する

取得項目

- Order番号
- Custom Label（SKU）
- Subtotal
- Shipping
- Sales tax
- Transaction fees
- Ad Fee
- Order earnings

さらにOrder詳細から以下を取得

- Buyer名
- ユーザーID
- 評価数
- 発送先国
- Shipping service
- eBay Item ID
- Sold日

---

### Step3 SKU採番

eBay販売日ベースでSKUを作成

例
p260322-001

既存SKUがある場合
最大番号 +1
で採番

---

## Notion登録

### 在庫管理DB

登録項目

- SKU
- カード名
- 仕入先
- 仕入価格
- 仕入URL
- 仕入決済日
- 仕入決済方法
- ステータス
- 鑑定機関
- 鑑定結果

仕入先

メルカリ(みんてぃあ)

---

### 海外販売管理DB

登録項目

- SKU
- 販売先
- 販売日
- 出品日
- Subtotal
- Shipping
- Sales tax
- Transaction fees
- Ad Fee
- Discount
- 為替レート
- Buyer情報
- 注文URL
- 出品URL
- 在庫DBリレーション
- 消費税還付

---

## カード画像登録

メルカリの商品画像1枚目を登録する

画像URL形式

`static.mercdn.net/item/detail/orig/photos/<商品ID>_1.jpg`

登録方法

1. Notionページを開く
2. カード画像フィールドをクリック
3. リンク貼付

---

## 鑑定証明番号登録

表示形式

`[cert番号](鑑定会社URL)`

例

| 鑑定機関 | URL形式 |
|---------|---------|
| PSA | `psacard.com/cert/<番号>` |
| ARS | `ars-grading.com/grading/searchNumber?id=<番号>` |
| CGC | `cgccards.com/certlookup/<番号>` |
| BGS | `beckett.com/grading/card-lookup?item_id=<番号>&item_type=BGS` |

---

## ビジネスルール

### 実質送料の計算

| 発送先 | 条件 | 計算式 |
|--------|------|--------|
| アメリカ | Subtotal < $2,500（DDP方式） | Subtotal × 15% × 為替レート（切り捨て） |
| アメリカ | Subtotal ≧ $2,500（DDU方式） | ¥3,000（関税は購入者負担のため定額） |
| アメリカ以外 | — | ¥3,000（仮置き。発送後に確定値に更新） |

### 送料実費＋関税の計算

| 発送先 | 条件 | 計算式 |
|--------|------|--------|
| アメリカ | Subtotal < $2,500（DDP方式） | ¥3,000 + (Subtotal × 15% × 為替レート)（小数点以下切り捨て） |
| アメリカ | Subtotal ≧ $2,500（DDU方式） | 実質送料と同額 |
| アメリカ以外 | — | 実質送料と同額 |

---

### デフォルト値

| 項目 | 値 |
|----|----|
| 為替レート | 155 |
| 実質送料 | 3000 |
| Ad Fee | 0 |
| Discount | 0 |
| 消費税還付 | 還付対象 |

---

## 鑑定機関判定

カード名から判定

例

- PSA10
- PSA9
- ARS10
- CGC10
- Gemix
- BGS Black Label 10
- BGS10
- BGS9

---

## eBay出品日の取得

Custom Label例

`260308001m`

意味

- 26 → 2026
- 0308 → 3月8日
- 001 → 連番

出品日

2026-03-08

---

## ユーザー確認事項

毎回確認

1. 仕入決済方法
2. 鑑定結果が存在しない場合、選択肢追加の要否

---

## 注意事項

- 在庫DBページを先に作成する
- 海外販売DBとリレーション接続する
- eBay Ad Fee は末尾スペースあり
- Sales tax が表示されない場合は 0
- メルカリShops購入の場合、取引URLの形式が通常メルカリと異なる
