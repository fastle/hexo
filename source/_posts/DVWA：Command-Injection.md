---
title: DVWA：Command Injection
date: 2024-09-26 10:05:40
tags:
 - 网络攻防
 - DVWA
---

命令注入指在应当输入数据的地方输入执行了代码。

# Low
输入你要 Ping 的 IP 地址， 但是对其没有进行任何检查，  可以在后面接上任何命令

# Medium
观察代码， 发现对于 `&&` 和 `;` 添加了转义， 所以使用 & 即可。

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
	// Get input
	$target = $_REQUEST[ 'ip' ];

	// Set blacklist  添加了转义
	$substitutions = array(
		'&&' => '',
		';'  => '',
	);

	// Remove any of the characters in the array (blacklist).
	$target = str_replace( array_keys( $substitutions ), $substitutions, $target );

	// Determine OS and execute the ping command.
	if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
		// Windows
		$cmd = shell_exec( 'ping  ' . $target );
	}
	else {
		// *nix
		$cmd = shell_exec( 'ping  -c 4 ' . $target );
	}

	// Feedback for the end user
	$html .= "<pre>{$cmd}</pre>";
}

?>

```

# High
发现被转义的更多了， 但是由于第三行编写错误导致有了可乘之机。
```php
$substitutions = array(
    '&'  => '',
    ';'  => '',
    '| ' => '',
    '-'  => '',
    '$'  => '',
    '('  => '',
    ')'  => '',
    '`'  => '',
    '||' => '',
);

```
# Impossible
使用解码限制了 IP 的格式， 并引入了 token ，基本不可解。 