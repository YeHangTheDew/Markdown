

1. 类型转换

   不支持column：：text的方式，使用 cast（column as type）

2. 时间 计算

   不支持，先转换到时间戳格式进行计算

   ```
   to_unix_timestamp({0}, 'yyyy-MM-dd HH:mm:ss)
   ```

3. 不支持 interval 求前几分钟、前几天操作

   ```
   from_unixtime(unix_timestamp(time) - 1*60*60*24,''yyyy-MM-dd HH:mm:ss'')
   ```

4. 不支持 string_agg聚合列函数

```
concat_ws(',',collect_set(column));
```

5. 中位数计算

   ```
   percentile_approx(coloumn,0.5)
   ```

   


