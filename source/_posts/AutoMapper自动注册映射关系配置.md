---
title: AutoMapper自动注册映射关系配置
date: 2018-05-08 10:29:00
permalink: AutoMappers-automatic-registration-mapping-relationship-configuration
tags: 
- ASP.NET Core
- .NET Core
- WebApi
- MVC
- AutoMapper
categories:
- .NET 笔记
---

使用以下配置，即可免去AutoMapper每个对象映射关系都需要注册的麻烦。

```csharp
Mapper.Initialize(cfg =>
{
	cfg.ForAllMaps((typeMap, mappingExpr) =>
	{
		var ignoredPropMaps = typeMap.GetPropertyMaps();
		foreach (var map in ignoredPropMaps)
		{
			var sourcePropInfo = map.SourceMember as PropertyInfo;
			if (sourcePropInfo == null) continue;

			if (sourcePropInfo.PropertyType != map.DestinationPropertyType)
				map.Ignored = true;
		}
		mappingExpr.ForAllMembers(opt =>
		{
			if (ignoredPropMaps.All(p => opt.DestinationMember.Name != p.SourceMember.Name))
			{
				opt.Ignore();
			}

		});
	});
});

```