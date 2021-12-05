# ElasticSearch

## 安装 ElasticSearch

> 下载ElasticSearch

下载地址： https://mirrors.huaweicloud.com/elasticsearch/?C=N&O=D

> 配置

在 config 目录下的`elasticsearch.yml`文件中添加下面配置解决跨域问题

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

> 启动

点击bin目录下的 `elasticsearch.bat` 启动elasticsearch，默认端口：9200

## 安装 elasticsearch-head 图形界面

> 下载 elasticsearch-head

```sh
git clone git://github.com/mobz/elasticsearch-head.git
```

> 运行 elasticsearch-head

```shell
npm install
npm run start
# 默认端口：9100
```

## 安装 Kibana

> 下载 Kibana

下载地址：https://mirrors.huaweicloud.com/kibana/?C=N&O=D（版本要和 ElasticSearch 一致）

> 运行

运行bin目录下 `kibana.bat`启动，默认端口：5601

## ElasticSearch 和数据库对比

| Relational DB      | ElasticSearch       |
| ------------------ | ------------------- |
| 数据库（database） | indices（索引）     |
| 表（tables）       | types（deprecated） |
| 行（rows）         | documents           |
| 字段（columns）    | fields              |

