---
title: 记在Mono 4.2 上EF使用Select操作时会发生的错误
tags:
- mono
- .NET
- EF
permalink: yun-xing-zai-monoshang-de-efyun-xing-cuo-wu
id: 18
updated: '2016-07-20 18:18:29'
date: 2016-07-12 10:38:39
---

错误提示：The classes in the module cannot be loaded.

代码还原：

```aspnet
 var data = await db.Set<FormulaCarType>()
.Where(c => c.Status == (int)CommonStatus.启用)
.Select(c => new FormulaCarModel
            {
                Id = c.Id,
                Type = c.CardType,
                Seats = c.Seats,
                FuelConsumption = c.FuelConsumption,
                MaintainFee = c.MaintainFee,
                CreateTime = c.CreateTime,
                Status = (CommonStatus)c.Status
            }).ToListAsync();
```

在查询中，在查询方法（ToListAsync）执行前进行了select的操作，则在执行的时候就会出错（在Windows上运行时不会发生），因此，需要在select前执行查询，再对查询结果进行select的操作。如下则不会出错：

```aspnet
var result = await db.Set<FormulaCarType>()
.Where(c => c.Status == (int)CommonStatus.启用).ToListAsync();
 var data = result.Select(c => new FormulaCarModel
            {
                Id = c.Id,
                Type = c.CardType,
                Seats = c.Seats,
                FuelConsumption = c.FuelConsumption,
                MaintainFee = c.MaintainFee,
                CreateTime = c.CreateTime,
                Status = (CommonStatus)c.Status
            });
```