`BookmarkRepository`有一个类似finder的方法，但这个方法依赖`Bookmark`实体对应`Account`关系的`username`属性，最终它需要一种连接，它会生成对应的查询：

```sql
SELECT b from Bookmark b WHERE b.account.username = :username
```



