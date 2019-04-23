---
title: コードスタイルに関する提案（一）
date: 2019-04-23 19:07:50
tags: [ Backend, Frontend ]
---

# The suggestion about code style (1)
## Foreword
![](coder.png)

Today, I want to talk about the topic of code style.

First of all, it needs to be said that there is no real good or bad code style. Just establish a standard that you will insist on is enough. Sometimes,  it is no reason and might be very casual, but it will be a very big gain if you insist on it!

If all codes can maintain the same code style in your team, it must be a great benefit to understanding and maintenance.

Since the language what I write the most is PHP, so let me use PHP as an example here, other languages are similar.

Language requirements + Community or Team habits + Own code style = Own code specification.

The purpose of the code style is trying to let the code follow the rules so that people can focus on " what you are doing? " rather than " how you are doing? ".

## Naming

1. The most important thing about naming is DO NOT use abbreviations casually!

The abbreviations are easy to repeat, and the different people might use different abbreviations at the same words, then, the meaning of the naming itself will be lost.

```php
# Error sample
public class Usr
{
    public $tmp_name;
    public $addr;
}

# Correct sample
public class User
{
    public $temp_name;
    public $address;
}
```

2. Don't use the tense and plural forms casually.
```php
# Error sample
$books_count = 10;
$deleted = TRUE;

# Correct sample
$book_count = 10;
$is_delete = TRUE;
```

3. Don't use _ and __ at the beginning casually.

Unless you are writing a framework or an underlying tool, don't use _ at the beginning. And in many languages and project habits, _ stands for internal, reserved, __ stands for language-level usages, such as __construct in PHP.

```php
# Error sample
$_temp = "a";

# Correct sample
$temp = "a";
```

4. Use is_ at the beginning when you are naming a bool variable.
```php
# Error sample
$closed = FALSE;
$disable = FALSE;
$deleted = TRUE;

# Correct sample
$is_open = TRUE;
$is_enable = TRUE;
$is_delete = FALSE;
```

5. Reserved keywords

If you have a team, or if your project is getting more complicated.
Then please make sure to keep some reserved words, such as id, key, state, created_timestamp, etc., don't re-name them every time.

6. Name variables in snake-mode

It is not recommended to use camel-mode because, in the usual habit, camel-mode is used to name a member method of the class.

```php
# Error sample
$FirstName = "Panda";
$lastName = "Lee";

# Correct sample
$first_name = "Panda";
$last_name = "Lee";
```

7. Name the global function with snake-mode.

To distinguish it from the member method of the class.

```php
# Global function
function get_name(): string
{
    $name = $first_name . '.' . $last_name;
    $res = $name;
    return $res;
}

# Member function
public class User
{
    public $first_name;
    public $last_name;
    
    public function getName(): string
    {
        $name = $first_name . '.' . $last_name;
        $res = $name;
        return $res;
    }
}
```

8. Name the class with upper-camel-mode.
```php
public class User
{
    public $first_name;
    public $last_name;
}

public abstract class Util
{
    public static function sayHello(): void
    {
        $string = "hello";
        echo $string;
    }
}

Util::sayHello();
```

## Header
1. DO NOT use "?>" at the end of the PHP file.

If there is still a character after "?>", it may be a blank or a newline, which will cause the program to crash.

```php
<?php declare( strict_types = 1 );
# no ?>    
```

2. declare( strict_types = 1 )

This is very important. Adding type constraints first will greatly improve the reliability of the code.
Secondly, JIT's friendliness will greatly improve the efficiency of code execution.

Please note that in strict mode, some behaviors are no longer allowed.

```php
function sum( $a, $b ): int
{
    $res = $a + $b;
    return $res;
}
# 2 + 1 = 3, correct
# 2.1 + 1 = 3.1, type error
```

## Global

From the perspective of the namespace, the global is also a domain, although it needs to use with caution, not means completely avoid using global variables.

### Variables
In general, please use global variables as less as possible.
Apart from the problem of poor management. The "global" of global variables is actually hard to define.
In the same project but different situations, the "global" may be the whole project, a single service, a process, a thread, or even one life cycle.
Therefore, in the JavaScript project, if it is a b/s client, global variables such as window, document, location, or user-agent might be very useful. And some global variables like app, storage, and api_host can be extended by themselves.

### Functions
Global functions are becoming less and less in object-oriented programming, and are replaced by static member methods of abstract tool classes.
Based on object-oriented thinking, this is a good thing, but sometimes still not as convenient as global functions. And there are many native language-level functions.
I recommend we only use the global functions to extend the functions provided by the system and follow the original naming rules. For example, in PHP, the global functions provided by the system is named through snake-mode to distinguish it from the class member method.

### Constants
We just use global constants when extending system-level functions

## Exception and Error
First of all, please notice the difference between exceptions and errors. That's why I do not recommend to use "e" as an abbreviation since both exception and error can be written as "e".
"Exceptions" usually need to be logically processed in the try/catch/finally structure, and "Errors" need to be debugged.

Let's use upload-file as an example.
If we require the file size to be less than 10MB, the size must be 800 * 600, and the file must be png type.

```php
Abstract
Class UploadFileTools
{
    public static
    function receiveUploadFile( File $upload_file ): bool
    {
        try {
            checkUploadFile( $upload_file );
            $res = saveFile( $upload_file );
        }
        catch ( UploadFileException $upload_file_exception ) {
            handleUploadFileException( $upload_file_exception );
            $res = false;
        }
        return $res;
    }

    public static
    function checkUploadFile( File $upload_file ): void
    {
        $image = getImage( $upload_file );

        $is_valid_file_size = checkFileSize( $upload_file );
        if ( $is_valid_file_size === false ) {
            throw new UploadFileException( FILE_SIZE_PROBLEM, "file size must be less than 10MB" );
        }

        $is_valid_image_type = checkImageType( $image );
        if ( $is_valid_image_type === false ) {
            throw new UploadFileException( IMAGE_TYPE_PROBLEM, "image type must be ". PNG_TYPE );
        }

        $is_valid_image_size = checkImageSize( $image, 800, 600 );
        if ( $is_valid_image_size === false ) {
            throw new UploadFileException( IMAGE_SIZE_PROBLEM, "image size must be 800 * 600" );
        }
    }

    public static
    function checkFileSize( File $file, int $limit = 10 * MB ): bool
    {
        ...
    }

    public static
    function checkImageType( Image $image, string $require_type = PNG_TYPE ): bool
    {
        ...
    }

    public static
    function checkImageSize( Image $image, int $require_height, int $require_weight ): bool
    {
        ...
    }
    
}
```

Errors are bugs in the source code. They should be discovered and fixed after tests, or caught by a higher level system, written to logs, alarmed, rolled back, restarted service, or other emergency handlings.

## Bool and Null
1. Please use === with !== instead of == and !=.
2. 
```php
# Error sample
if ( $a == 1 )

# Correct sample
if ( $a === 1 )
```

2. Please use TRUE, FALSE, NULL;

My personal reason is that "TRUE" looks a lot like a constant, in fact, bool and null are really constants in a sense.

```php
# Error sample
$is_confirm = true;
$description = null;

# Correct sample
$is_confirm = TRUE;
$description = NULL;
```

## Indent and Wrap
1. No tab, use 4 spaces

And I personally dislike 2 spaces, because it is very unclear, except in the dom nesting of the traditional HTML countless levels, I recommend everyone to use the indentation of 4 spaces.

2. Less than 80 characters in a line

Enter long and multi-line text by a template or stitched method.

```php
$text = <<<EOT
text text
text
text
EOT;

# or

$text = "abc" . PHP_EOF
    . "abc" . PHP_EOF
    . "abc" . PHP_EOF
```

## Comment
First, there are two purposes of comments, and doing things by purpose is always the most effective.
One purpose is to write to yourself to avoid forgetting the logic.
Another is to write to others and let them understand.
So generally don't comment on the code itself, you have to assume that the person who reads the comment is programming enough. So please just write what you are doing.
For the complex part, you can make a single line comment at the end. 
At here, I have to say that because JSON files do not allow comments, which makes the configuration such as package.json very unfriendly.

### TODO
TODO comments are a very handy thing, IDE and git can capture them, but in the team project, please write your email in TODO. Someone can find you in the future when countless TODO is unclaimed ^^

```php
# TODO [li@stageinc.jp] finish this article
```

### Line
Use # to make line comments instead of //, if it is tail, keep 2 spaces for easy reading

```php
$name = get_name();  # get name by a global function.
```

### Block
Most languages have own defined block comment tools, such as PHPDoc, just follow their rules and automatically generate the block comments by IDE might be a good choice.

```php
/**
 * @param int $index
 * @return string
 */
function get_name( int $index = 0 ): string
{
    $res = "name";
    return $res;
}
```

A large section of notes that describe functions, complex logic, or other information.

```php
/**
 * Problem Statement
 * Two strings are called anagram if and only if one string can be transformed into the other by changing the order of characters.
 * You are given n strings si.
 * Your task is to count the number of ( i, j ) such that si and sj are anagram and i < j.
 *
 * Constraints
 * 1 ≦ n ≦ 100000
 * 1 ≦ |si| ≦ 100000
 * Σ|si| ≦ 100000
 * si consists of lowercase latin letters
 */
```

## Final
Ok, that's the end today, thank you very much.
In the next issue, I will provide a completed PHPStorm configuration file for your reference. And we will discuss the project structure, package management and more examples of JavaScript.

I hope you can understand that which kind of code style you are using is really not important. But it is important to stick to use and to make codes easy to read.

So what is a good code style?
Consensus and compliance of the team or community > Your own COOOOOL > Appreciation from others.