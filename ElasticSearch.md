# ElasticSearch

## 安装 ElasticSearch

### Windows

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

#### 安装 elasticsearch-head 图形界面

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

#### 安装 Kibana

> 下载 Kibana

下载地址：https://mirrors.huaweicloud.com/kibana/?C=N&O=D（版本要和 ElasticSearch 一致）

> 运行

运行bin目录下 `kibana.bat`启动，默认端口：5601

####  安装IK分词器

> 下载IK分词器

下载地址：https://github.com/medcl/elasticsearch-analysis-ik（版本要和 ElasticSearch 一致）‘

> 安装IK分词器

将IK分词器压缩包解压到`ElasticSearch>plugins>ik`下，并重启ElasticSearch

> 使用方法

```sh
# 分词
Get _analyze
{
  "analyzer": "ik_smart",
  "text": "农村人居环境整治提升五年行动方案"
}
```



## ElasticSearch使用

### ElasticSearch 和数据库对比

| Relational DB      | ElasticSearch       |
| ------------------ | ------------------- |
| 数据库（database） | indices（索引）     |
| 表（tables）       | types（deprecated） |
| 行（rows）         | documents           |
| 字段（columns）    | fields              |

### 请求格式

| method | url地址                                  | 描述                   |
| ------ | ---------------------------------------- | ---------------------- |
| PUT    | ip:port/索引名称/类型名称/文档id         | 创建文档（指定文档id） |
| POST   | ip:port/索引名称/类型名称                | 创建文档（随机文档id） |
| POST   | ip:port/索引名称/类型名称/文档id/_update | 修改文档               |
| DELETE | ip:port/索引名称/类型名称/文档id         | 删除文档               |
| GET    | ip:port/索引名称/类型名称/文档id         | 通过文档id 查询文档id  |
| POST   | ip:port/索引名称/类型名称/_search        | 查询所有数据           |

### 使用kibana的Dev Tools 操作ES

```shell
# 创建索引规则
PUT /test2
{
  "mappings": {
    "properties": {
      "name" : {
        "type": "text"
      },
      "age": {
        "type": "long"
      },
      "birthday": {
        "type": "date"
      }
    }
  }
}

# 查询索引
GET test2

# 如果文档字段没有指定类型，那么es就会默认配置字段类型
PUT /test3/_doc/1
{
  "name": "张三",
  "age": 26,
  "birth": "1999-10-17"
}

# 查看ES状态
GET _cat/indices?vs

# 删除索引
DELETE test2
```

