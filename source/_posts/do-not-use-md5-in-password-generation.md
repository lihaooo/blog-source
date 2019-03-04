---
title: パスワード生成時にmd5を使用しない
date: 2019-03-03 22:58:38
tags: [ Security, Backend, Fintech ]
---

# Do not use md5 in password generation.

## NOW
- これは中国とアメリカのほとんどの古いプロジェクトに当てはまります
- In most old projects in China and America, the situation is like this.

```php
// Pseudocode
class User {
	...
	private $password;
	...
	...
	// get password salt
	private function getPasswordSalt(): string
	{
		...
	}
	// for generation
	public function generatePassword( string $password = 'default' )
	{
		$password_hash = md5( $password.$this->getPasswordSalt() );
		$this->password = $password_hash;
	}
	...
	// for validation
	public function validPassword( stirng $input_passowrd ): bool
	{
		$password_hash = md5( $this->password.$this->getPasswordSalt() );
		$input_password_hash = md5( $input_password.$this->getPasswordSalt() );
		$valid = $password_hash === $input_password_hash;
		$res = $valid;
		return $res;
	}
	...
}
```

- しかし、金融システムでmd5を使用するのは安全ではありません，同様に、sha1、sha128、sha256も危険です
- But, in financial systems, md5 is not safe, and sha1, sha128, sha256 are also dangerous.

## RECOMMEND
- md5をBCRYPTに置き換えます（またはPBKDF2）
- replace md5 with BCRYPT (or PBKDF2)

```php
// Pseudocode
class User {
	...
	private $password;
	...
	// for generation
	public function generatePassword( string $password = 'default' )
	{
		$password_hash = password_hash( $password, PASSWORD_BCRYPT );
		$this->password = $password_hash;
	}
	...
	// for validation
	public function validPassword( stirng $input_passowrd ): bool
	{
		$password_hash = password_hash( $password, PASSWORD_BCRYPT );
		$input_password_hash = password_hash( $input_passowrd, PASSWORD_BCRYPT );
		$valid = $password_hash === $input_password_hash;
		$res = $valid;
		return $res;
	}
	...
}
```

## WHY

### Q1: ドラッグライブラリのリスク
The risk of Drag-Library
- md5アルゴリズムは速すぎる
- The algorithm of md5 is too fast!

```php
// Speed test result (2core 4G Debian)
once result:
	BCRYPT 63 ms // when cost is 10 (default)
	md5 <0 ms
```

- ユーザーデータベースが漏えいした場合，md5暗号化パスワードはプレーンテキストのようになります
- If our user database has been leaked, the md5 encrypted password will be like plain text.

```php
// Violent cracking machine example
type: HashFast Sierra Batch 2
price: 7000USD
power: 780W
performance: 1200GH/s

// sha256 performance
1.2兆回每秒

// md5 performance
>2兆回每秒

// BCRYPT performance, when cost is 12 (recommend)
10万回每秒
```

- つまり、8バイトのmd5またはsha256パスワードはすべて40秒以内に解読される可能性があります、しかしBCRYPTは70年かかります
- It means all 8-byte md5 or sha256 passwords will be cracked within 40 seconds. But BCRYPT takes 70 years.

### Q2: md5アルゴリズムのハッシュ結果は繰り返される可能性があります
The hash result of the md5 algorithm may be repeated.

```php
// 128 bytes string_a
d131dd02c5e6eec4693d9a0698aff95c2fcab58712467eab4004583eb8fb7f89 
55ad340609f4b30283e488832571415a085125e8f7cdc99fd91dbdf280373c5b 
d8823e3156348f5bae6dacd436c919c6dd53e2b487da03fd02396306d248cda0 
e99f33420f577ee8ce54b67080a80d1ec69821bcb6a8839396f9652b6ff72a70

// 128 bytes string_b
d131dd02c5e6eec4693d9a0698aff95c2fcab50712467eab4004583eb8fb7f89 
55ad340609f4b30283e4888325f1415a085125e8f7cdc99fd91dbd7280373c5b 
d8823e3156348f5bae6dacd436c919c6dd53e23487da03fd02396306d248cda0 
e99f33420f577ee8ce54b67080280d1ec69821bcb6a8839396f965ab6ff72a70

// same md5 hash result
79054025255fb1a26e4bc422aef54eb4
```

- したがって、md5 ( md5 ( $password ) )も信頼できません
- So, md5 ( md5 ( $password ) ) is also not unreliable.

### Q3: md5などのアルゴリズムはパスワードの生成には使用されません
Algorithms such as md5 are not used to generate passwords.

- その超高性能により、md5アルゴリズムは大きなファイルの整合性を検証するのに優れています
- With its super high performance, the md5 algorithm is excellent for verifying the integrity of large files.

- sha256はセキュリティに関してmd5よりもかなり優れていますが、金融システムのコアパスワードには適しておらず、一般的な認証およびマッピングには適しています
- Although sha256 is much better than md5 in security, it is also not suitable for the core password of the financial system and it is suitable for general authentication and mapping.

## DETAIL

```php
// use cost option in BCRYPT
$option = [ 'cost' => $cost ];
$password_hash = password_hash( $password, PASSWORD_BCRYPT, $option );

// Speed test result (2core 4G Debian)
once result:
	BCRYPT_10 63 ms // when cost is 10 (default)
	BCRYPT_12 178 ms // when cost is 12 (recommend)
	BCRYPT_14 1003 ms // when cost is 14 (special)
```

お時間を割いていただきありがとうございます
ありがとうございました＾＾