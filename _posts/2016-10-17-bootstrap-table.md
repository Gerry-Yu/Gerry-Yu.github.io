---
layout: post
title:  "Bootstrap-table"
categories: 前端
tag: 前端
---

## 导入文件

> * bootstrap.css
> * bootstrap-table.css
> * jquery.js
> * bootstrap.js
> * bootstrap-table.js
> * bootstrap-table-zh-CN.js

## html
> 这里使用JavaScript 的方式，还可以使用data 属性的方式

``` html
 <table id="table"></table>
```
## JS

> 这里使用Server分页方式，想后台传入pageSize和PageNumber。使用post方式必须设置`contentType`

``` JavaScript
$('#table').bootstrapTable({
    method: 'post',
    contentType: "application/x-www-form-urlencoded",
    striped: true,
    pagination: true,
    pageSize: 5,
    pageNumber:1,
    search: false,
    showColumns: false,
    sidePagination: "server",
    queryParamsType : "undefined",
    queryParams: function queryParams(params) {
        var param = {
            pageNumber: params.pageNumber,
            pageSize: params.pageSize,
        };
        return param;
    },
    cache: false,
    sortable: false,
    sortOrder: "asc",
    height: 400,
    pageList: [5, 10, 25],
    url: 'getUsers',
    columns: [{
        field: 'userId',
        title: 'userId'
        }, {
        field: 'userName',
        title: 'userName'
    }]
});
```

## 后台Controller

> 这里需要想前端传入`total`和`rows`，不能是其他命名！！！这里为了简便total没有查询数据库

``` java
@RequestMapping("getUsers")
@ResponseBody
Map getUsers(int pageSize, int pageNumber) {
    System.out.println(pageSize + " " + pageNumber);
    HashMap<String, Object> hashMap = new HashMap<String, Object>();
    int first = (pageNumber - 1 ) * pageSize;
    hashMap.put("rows", userService.getUser(first, pageSize));
    hashMap.put("total", 21);
    return hashMap;
}
```

## 效果

![](/styles/images/2016-10-17-bootstrap-table.jpg)

## 文档
[bootstrap-table](http://bootstrap-table.wenzhixin.net.cn/zh-cn/getting-started/)




