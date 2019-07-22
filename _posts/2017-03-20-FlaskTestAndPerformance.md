---
layout: post
title:  Chapter 15 Testing and Performance
date:   2017-03-20 22:37:00 +0800
categories:
  - Reading
  - Flask
---

## Testing

### Obtaining Code Coverage Reports

```shell
pip install coverage
```

`coverage`可以用来检测测试覆盖率

### Flask Test Client

`flask client`可以模拟`request, session`等环境，方便测试`route`

### End-to-End Testing with Selenium

```shell
pip install selenium
```

`selenium`可以调用真实浏览器来进行测试

## Performance

### Slow Database Performance

可以直接使用`SQLAlchemy`的`get_debug_queries()`获取数据库使用性能数据，找到最毫时的`sql`语句

### Source Code Profiling

可以使用`werkzeug`的`ProfilerMiddleware`来检测每次`request`的性能数据
