---
title: 金額の計算に関する質問（一）
date: 2019-03-04 18:57:15
tags: [ Fintech, Backend, System Design ]
---

# Questions about amount calculation (1)
## SCENE

![](meal.jpg)

私は辻山さんとかつやさんと一緒に中華料理を食べに行きました
If I go to eat Chinese food with Mr.Tsujiyama and Mr.Katsuya.
* 牛肉 1000円
* 麻婆豆腐 500円
* 餃子 500円
* 炒飯 450円

DigiCashの割り勘機能5%割引
5% discount on DigiCash's Dutch-Treat function

---

一般に、ロジックは次のようになります
In general, the logic is as follows.
* 2450 * 0.95 = 2327.5円
* 2327.5 / 3 = 775.83333...円 / 人

## BUT
### 浮点数はお勧めできません
Float point numbers (float/double) are not recommended.
#### よく知られている精度の問題
Well-known precision problem

```php
// precision problem example
$a = (int)( 0.58 * 100 );
$b = 0.58 * 100;
echo $a.' '.$b;

// result
57 58
```
#### データベース設定の問題
Database options problem

```SQL
CREATE TABLE `panda`.`test` (
  `id` INT NOT NULL,
  `a` FLOAT(6,2) NULL,
  `b` DOUBLE(6,2) NULL,
  `c` DECIMAL(6,2) NULL,
  PRIMARY KEY (`id`));

INSERT INTO `panda`.`test` (`id`, `a`, `b`, `c`) VALUES ('1', '1.111', '1.117', '1.117');

INSERT INTO `panda`.`test` (`id`, `a`, `b`, `c`) VALUES ('2', '1234567', '1234.892', '1234.892');
```
Result in DB
| id | a | b | c |
| --- | --- | --- | --- |
| 1 | 1.11 | 1.12 | 1.12 |
| 2 | 9999.99 | 1234.89 | 1234.89 |

### 決済における参照値と実績値
Reference and actual values in settlement

* 実際値のみが決済に参加することができます
* Only the actual value can participate in the settlement.
* 参照値の意味は実際値を決定することです
* The meaning of the reference value is to determine the actual value.

> RV = Reference Value 参照値
> AV = Actual Value 実績値

#### 割引 Discount
##### 5%割引の本質 The essence of 5% discount
AV 5%, AV 2450
-> ( ( 1 - 5% ) * 2450 )
-> RV 2327.5
-> AV 2327
-> ( 2327 - 2450 )
-> AV -123

> 123 > 5% * 2450

#### 配布 Distribution
* 配布の焦点は、その完全性を確実にすることです
* The focus of distribution is to ensure its integrity.

##### 割り勘 Dutch treat
> 総料 / 人数 = RV
> ceil(RV) = AV1
> p(1) = p(2) = ... = p(n-1) = AV1
> p(n) = 総料 - (n-1) * AV1 = AV2

##### 払い戻し Refund
> r(1) = 1000 * 95% = RV 950 -> AV 950
> r(2) = r(3) = 500 * 95% = RV 475 -> AV 475
> r(4) = 2327 - r(1) - r(2) - r(3) = AV 427

##### 割引の詳細 Discount detail
> d(1) = 1000 * 5% = RV 50 -> AV 50
> d(2) = d(3) = 500 * 5% = RV 25 -> AV 25
> d(1) + d(2) + d(3) + d(4) = AV 123

## RECOMMEND
### 整数のみを使用 Only integer
* 現在の主流の金融システムは、ブロックチェーンを含む金額を処理するために整数を使用しています
* The current mainstream financial system uses integers to process amounts, including blockchain.

#### 最小単位を決定し、整数を使用して金額を処理します
Determine the minimum unit and use the integer to process the amount.

```php
// Pseudocode
class Currency {
    private $id;
    private $name;
    private $standard_unit;
    private $standard_unit_symbol;
    ...
}

class Money {
    private $currency_id;
    private $amount;
    ...
}

// define JPY
$jpy = new Currency();
$jpy->id = 1;
$jpy->name = 'JPY';
$jpy->standard_unit = 1; // 1 standard_unit = 100 minimum_unit
$jpy->standard_unit_symbol = '円';

// define USD
$usd = new Currency();
$usd->id = 2;
$usd->name = 'USD';
$usd->standard_unit = 100; // 1 standard_unit = 100 minimum_unit
$usd->standard_unit_symbol = 'ドル';

// taxi fee 4500円
$taxi_fee_jpy = new Money();
$taxi_fee_jpy->currency_id = $jpy->id;
$taxi_fee_jpy->amount = 4500;

// movie ticket 2.43USD
$movie_ticket_usd = new Money();
$movie_ticket_usd->currency_id = $usd->id;
$movie_ticket_usd->amount = 243;
```

#### ブロックチェーントークン Blockchain Token
* 最小単位で量を測定する
* Measure the quantity in minimum unit.
##### BTCの最小単位 The minimum unit of BTC
> 0.00000001 BTC = 1 聡
> 1 BTC in block info is 10,000,000

##### イーサリアムERC20 Ethereum ERC20
> 1 ETH = 10^18 Wei
> 1 ETH in block info is 1,000,000,000,000,000,000

### 事実に基づいてシステムを設計する
Design a system based on facts.

#### 事実プロセス Factual process
1. 料理を注文する Restaurant order
    AV 1000 + AV 500 + AV 500 + AV 450
2. 決済 Settlement
    AV 2450
3. [1] Restaurant order -> [2] DigiCash product order
4. DigiCash settlement
    1. AV 5% * AV 2450 = RV 2327.5
    2. AV 2327
5. [3] DigiCash main payment order: AV 2327
    AV 2327 / AV 3人 = RV 775.666...7
    1. [3-1] Panda sub payment order: AV 776
    2. [3-2] Tsujiyama sub payment order: AV 776
    3. [3-3] Katsuya sub payment order: AV 775
6. [3-all] All sub payment orders finished
7. [3] DigiCash main payment order finished
8. [2] DigiCash product order finished
9. [1] Restaurant order finished

#### 参照値
1. 2327.5 -> 2327
2. 775.66...7 -> 776

#### 実績値
1. 3人
2. 1000円 / 500円 / 500円 / 450円
3. 2450円
4. 5%
5. 2327円
6. 776円
7. 776円
8. 775円