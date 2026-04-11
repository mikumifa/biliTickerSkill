# 搜索票务

当用户不是直接给活动链接，而是只给一个关键词、活动名、角色名、IP 名时，先不要要求他自己去找链接。

应优先直接调用搜索接口，先把候选活动搜出来，再继续后面的选票流程。

## 适用场景

例如用户这样说：

- `帮我抢原神的票`
- `搜一下原神 only`
- `我想买 ilem 的南京巡演`
- `帮我看看有没有林震的票`

这时先调用：

```python
import bilitickerbuy

result = bilitickerbuy.search_tickets("原神")
text = bilitickerbuy.format_ticket_search_results_text(result)
```

## 接口

### `search_tickets`

```python
search_tickets(
    keyword: str,
    *,
    page: int = 1,
    pagesize: int = 16,
    platform: str = "web",
    cookies=None,
    cookies_path=None,
) -> dict
```

返回结构里最重要的是：

- `keyword`
- `page`
- `pagesize`
- `total`
- `results`

每个 `results` 项里通常会有：

- `id`
- `title`
- `project_name`
- `city`
- `venue_name`
- `tlabel`
- `price_low`
- `price_high`
- `sale_flag`
- `url`

### `format_ticket_search_results_text`

```python
format_ticket_search_results_text(search_result: dict, *, limit: int = 10) -> str
```

这个函数会把搜索结果整理成适合直接发给用户的文字格式。

## 推荐交互方式

第一次把结果发给用户时，优先列出前几条最相关结果，不要让用户自己翻原始 JSON。

例如：

```text
我先帮你搜到了这些结果，你看看更像哪一个：

1. 北京·原神only5.0同人展·璃光宵彩宴
   城市：北京市  场地：北京大红门国际会展中心
   时间：2025.02.15
   价格：￥75.0 - ￥100.0  状态：已结束

2. 上海·原神×崩坏×星铁only旅行盛宴2.0
   城市：上海市  场地：上海大世界
   时间：2024.05.18 - 05.19
   价格：￥65.0 - ￥178.0  状态：已结束
```

然后用自然语言继续问：

```text
你想要的是哪一个？如果都不对，我可以继续换关键词搜。
```

## 规则

1. 用户只给关键词时，先搜索，不要先逼用户补链接。
2. 搜索结果优先整理成文字给用户看，不要直接扔原始 JSON。
3. 如果结果太多，只先展示最相关的几条。
4. 如果用户给的是模糊名称，允许先搜一次，再根据用户反馈换关键词继续搜。
