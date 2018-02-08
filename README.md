# 我们是什么

abajsdhkajs
## ssssssssss

```java
@GetMapping("received_card")
public OkResponse receivedCard(@Context HttpServletRequest request, String openId, Integer cardId){

if(jfqService.hasEnd()){
return OkResponse.error(2, "活动已结束");
}
if(cardId == null || !cardService.checkCardId(cardId)) {
return OkResponse.error(-1, "参数不合法");
}

Integer userId = currentUserId(request);
if(userId == null || userId == 0) {
return OkResponse.error(1, "未登录");
}

return OkResponse.ok(jfqService.receivedCard(userId, cardId, openId));
}
```

This file file serves as your book's preface, a great place to describe your book's content and ideas.

```java
    @GetMapping("ask_card")
    public OkResponse askCard(@Context HttpServletRequest request, Integer cardId){
        if(jfqService.hasEnd()){
            return OkResponse.error(2, "活动已结束");
        }
        if(cardId == null || !cardService.checkCardId(cardId)) {
            return OkResponse.error(-1, "参数不合法");
        }

        Integer userId = currentUserId(request);
        if(userId == null || userId == 0) {
            return OkResponse.error(1, "未登录");
        }
        return OkResponse.ok(jfqService.askCard(userId, cardId));
    }
```



