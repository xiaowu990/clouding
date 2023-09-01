# 教程链接
[PromQL快速入门_涂作权的博客的博客-CSDN博客](https://blog.csdn.net/tototuzuoquan/article/details/119719591?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169234585716800225587577%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169234585716800225587577&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-119719591-null-null.142^v93^insert_down1&utm_term=promQL&spm=1018.2226.3001.4187)
更详细的如下
[来自weiyiweek.top的详细文档](https://blog.weiyigeek.top/2021/4-25-554.html)

在 Prometheus 中，指标（Metric）是被监控的某个具体指标的名称，例如 `http_requests_total` 表示 HTTP 请求的总数。每个指标都可以包含一个或多个标签（Label），标签提供了关于指标的附加信息。例如，针对 `http_requests_total` 指标，可能有标签 `job` 表示所属的作业（例如 `webserver`），标签 `status` 表示请求的状态码（例如 `200`）。

基本表达式：==指标名称{标签选择器}==
基本表达式由指标名称和一组标签组成，用于选择特定的时间序列。例如，`http_requests_total{job="webserver", status="200"}` 是一个基本表达式，选择了 `http_requests_total` 指标中 `job` 标签为 "webserver"、`status` 标签为 "200" 的时间序列。

>在 Prometheus 中，指标（Metric）是被监控的某个具体指标的名称，例如 `http_requests_total` 表示 HTTP 请求的总数。每个指标都可以包含一个或多个标签（Label），标签提供了关于指标的附加信息。例如，针对 `http_requests_total` 指标，可能有标签 `job` 表示所属的作业（例如 `webserver`），标签 `status` 表示请求的状态码（例如 `200`）。

# PromQL介绍
## 基础简述
## 基础标准

## 数据类型

### 瞬时数据 (Instant vector)

### 区间数据 (Range vector)

### 纯量数据 (Scalar)

### 字符串数据 (String)

## 选择器 - (Selector)

## 匹配器 - (Matcher)

## 偏移修改器 - (Offset)

## 修饰符 - @

## 子查询

## 查询类型 
### Counter 类型

### Gauge 类型

### Summary 类型

### Histogram 类型
## API查询指标数据

### Query

### Query_range
# 表达式语言运算符

## 二元运算符

## 修饰运算符

### ignoring 修饰符

### on 修饰符

### group_left 修饰符

### group_right 修饰符

## 聚合运算符

### 分组

### 聚合

## 运算符优先级
# 实际案例

## promtheus

## node_exporter

## Windows (win_exporter)

#  PromQL 内置函数介绍

 