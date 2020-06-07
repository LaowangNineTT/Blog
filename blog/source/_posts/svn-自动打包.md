---
title: svn 自动打包
date: 2018-08-19 16:22:07
tags: svn
---

> 找了个时间写了一个 SVN 自动寻找某人打包的工具
> 
> 最近是比较痛苦的一段时间，git 和 svn 一起运行。
> 
> 然后这里面每次 svn 打包的时候都会少上文件，花了一会儿，写了一个 svn 自动打包的工具。

<!--more-->

```php
<?php

fwrite(STDOUT, "Please enter your working copt path[ep: /www/htdocs/xxx ]");
$path = trim(fgets(STDIN));

fwrite(STDOUT, "Please enter your svn version for last make package [ep: 16600]:");
$lastVersion = trim(fgets(STDIN));

fwrite(STDOUT, "Please enter your svn name [ep: xx.zhang]:");
$name = trim(fgets(STDIN));

echo "\n Path is :{$path}\n";
echo "\n Version is :{$lastVersion}\n";
echo "\n Name is :{$name}\n";
echo "\n";
sleep(1);
echo ' 3...' . "\n";
sleep(1);
echo ' 2...' . "\n";
sleep(1);
echo ' 1...' . "\n";
echo "-------------------------------------------------------------\n";
exec("svn info {$path}", $info);

$revision = $info[6];
if (empty($revision)) {
    echo "Please Check Your Input Path? \n";
    die;
}
$localPath = str_replace('Working Copy Root Path: ', '', $info[1]);
$svnUrl    = str_replace('URL: ', '', $info[2]);
preg_match_all('/\d{4,6}/', $revision, $match);
$version = $match[0][0];
exec('svn diff -r ' . $lastVersion . ':' . $version . ' --summarize ' . $svnUrl, $svn);
if (empty($svn)) {
    echo "SVN Error...Can't to make package!\n";
    die;
} else {
    echo "Ready to collect svn update diff files....\n";
}
$filePath = array();
foreach ($svn as $value) {
    $fileStatus  = substr($value, 0, 1);
    $svnFilePath = str_replace($fileStatus . '       ', '', $value);
    $svnLog      = array();
    exec('svn log -r ' . $lastVersion . ':' . $version . ' ' . $svnFilePath . ' | grep ' . $name, $svnLog);
    if (empty($svnLog)) {
        continue;
    }
    $changePath = str_replace($fileStatus . '       ' . $svnUrl, $localPath, $value);
    $filePath[] = $changePath;
}

if (empty($filePath)) {
    echo "Can't Find Update File\n";
} else {
    $count = count($filePath);
    echo "Find {$count} File To Update!\n";

    // mkdir
    $date        = date("Y-m-d");
    $packagePath = $localPath . '/svn_package/' . $date . '/';
    if (is_dir($packagePath)) {
        exec('rm -rf ' . $packagePath . "*");
        echo "Find a Exist Package FilePath\n";
    } else {
        mkdir($packagePath, 0777, true);
    }

    foreach ($filePath as $path) {
        $targetFile = str_replace($localPath . '/', $packagePath, $path);
        $dirName    = dirname($targetFile);
        if (!is_dir($dirName)) {
            exec('mkdir -p ' . $dirName);
        }
        exec('cp ' . $path . ' ' . $targetFile);
    }

    echo "Finish....\n";
}
```
