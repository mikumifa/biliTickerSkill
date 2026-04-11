# 搜索票务

当用户不是直接给活动链接，而是只给一个关键词、活动名、角色名、IP 名，或者直接说“帮我找漫展”时，先不要要求他自己去找链接。

应优先直接调用搜索接口，先把候选活动搜出来，再继续后面的选票流程。

## 适用场景

例如用户这样说：

- `帮我找上海最近的漫展`
- `看看北京 5 月有什么二次元展`
- `帮我抢原神的票`
- `搜一下原神 only`
- `我想买 ilem 的南京巡演`
- `帮我看看有没有林震的票`

这时先调用：

```python
import interface as btb

result = btb.search_tickets("原神")
text = btb.format_ticket_search_results_text(result)
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

注意：

1. 这个接口走的是会员购搜索接口，通常需要可用登录态。
2. 这个函数内部会先检查登录态；如果已经登录，会自动复用当前 cookies 发起搜索。
3. 如果当前未登录，不应该继续搜索，而是直接进入登录流程。
4. 返回结果里会包含 `ok`、`requires_login`、`next_action` 这些字段，方便 skill 直接分支。

返回结构里最重要的是：

- `ok`
- `keyword`
- `page`
- `pagesize`
- `total`
- `results`
- `requires_login`
- `next_action`

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

如果传入的是“未登录，需要先登录”的搜索结果，它会直接返回一段提示登录的自然语言，不需要调用方自己额外拼文案。

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
5. 如果 `search_tickets(...)` 返回 `requires_login=True`，就停止搜索并进入登录流程，不允许改走公开网页。
6. “找漫展”本质上也是搜索会员购活动，应当由当前 skill 直接处理。
