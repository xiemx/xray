---
layout: post
title: 常见MIME类型设置方法
published: true
author: xiemx
comments: true
date: 2015-08-21 01:08:58
tags: [ http, mime-type ]
categories:
    - 'http'
    - 'mime-type'
# permalink: '/2015/08/21/mime-type'
---
### 常见MIME类型和设置方法
当前列举了常用的MIME类型，查询详细：[MIME Type](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)

#### MIME类型

| 扩展名 | 文档类型 | MIME 类型 |
| :----- | :------ | ----: |
| .aac | AAC audio | audio/aac |
| .abw | AbiWord document | application/x-abiword |
| .arc | Archive document (multiple files embedded) | application/x-freearc |
| .avi | AVI: Audio Video Interleave | video/x-msvideo |
| .azw | Amazon Kindle eBook format | application/vnd.amazon.ebook |
| .bin | Any kind of binary data | application/octet-stream |
| .bmp | Windows OS/2 Bitmap Graphics | image/bmp |
| .bz | BZip archive | application/x-bzip |
| .bz2 | BZip2 archive | application/x-bzip2 |
| .csh | C-Shell script | application/x-csh |
| .css | Cascading Style Sheets (CSS) | text/css |
| .csv | Comma-separated values (CSV) | text/csv |
| .doc | Microsoft Word | application/msword |
| .epub | Electronic publication (EPUB) | application/epub+zip |
| .gif | Graphics Interchange Format (GIF) | image/gif |
| .html | HyperText Markup Language (HTML) | text/html |
| .ico | Icon format | image/vnd.microsoft.icon |
| .ics | iCalendar format | text/calendar |
| .jar | Java Archive (JAR) | application/java-archive |
| .jpeg | JPEG images | image/jpeg |
| .js | JavaScript | text/javascript |
| .json | JSON format | application/json |
| .mjs | JavaScript module | text/javascript |
| .mp3 | MP3 audio | audio/mpeg |
| .mpeg | MPEG Video | video/mpeg |
| .oga | OGG audio | audio/ogg |
| .ogv | OGG video | video/ogg |
| .ogx | OGG | application/ogg |
| .otf | OpenType font | font/otf |
| .png | Portable Network Graphics | image/png |
| .pdf | Adobe Portable Document Format (PDF) | application/pdf |
| .rar | RAR archive | application/x-rar-compressed |
| .rtf | Rich Text Format (RTF) | application/rtf |
| .sh | Bourne shell script | application/x-sh |
| .svg | Scalable Vector Graphics (SVG) | image/svg+xml |
| .tar | Tape Archive (TAR) | application/x-tar |
| .tiff | Tagged Image File Format (TIFF) | image/tiff |
| .ttf | TrueType Font | font/ttf |
| .txt | Text, (generally ASCII or ISO 8859-n) | text/plain |
| .vsd | Microsoft Visio | application/vnd.visio |
| .wav | Waveform Audio Format | audio/wav |
| .woff | Web Open Font Format (WOFF) | font/woff |
| .woff2 | Web Open Font Format (WOFF) | font/woff2 |
| .xhtml | XHTML | application/xhtml+xml |
| .xls | Microsoft Excel | application/vnd.ms-excel |
| .xml | XML | application/xml|
| .zip | ZIP archive | application/zip |
| .3gp | 3GPP audio/video container | video/3gpp |
| .3g2 | 3GPP2 audio/video container | video/3gpp2 |
| .7z | 7-zip archive | application/x-7z-compressed |

#### 设置方法
```markdown
# IIS
默认网站属性-->http 头-->MIME映射-->文件类型-->新类型

# nginx
conf/mime.types
```