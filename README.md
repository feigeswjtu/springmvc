写了快两年的Java代码了，发现自己居然不会搭建一套Spring mvc环境出来，只会添砖加瓦，说来就惭愧了，为了弥补这点的缺陷，也为了以后更好的借助已有的SpringMVC项目学习更多的知识，我决定从零搭建一套自己的SpringMVC。

网上搜索大部分都是基于XML配置的环境搭建，基于XML大部分是因为历史项目的原因无法切换成基于JavaConfig来进行配置，但是基于JavaConfig是未来的趋势，所以本系列的文章是零XML配置的，喜欢XML请自行搜索其他文章。

使用的基础环境如下:
 - 系统: MacOS（也就是Linux系统）
 - java版本: 1.8.0_151，[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
 - Tomcat版本: 8.5.20， [下载地址](https://tomcat.apache.org/download-80.cgi)
 - Maven版本: 3.5.0，[下载地址](http://maven.apache.org/download.cgi)
 - 开发工具: IntelliJ Idea

废话不多说，开始整起。
