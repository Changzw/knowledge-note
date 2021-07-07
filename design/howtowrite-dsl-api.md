
# 如何写一个声明式的库

1. 组合模式

```swift
component0(
	[component11,
	 component12,
	 component13(
		 component21,
		 component22,
		 component23,
		 component24,
	 ),]
)
```

2. 如果每个 component传递的参数相同（eg: context）

```swift
那么可以把 component 构建方法返回值写成闭包 (context)->()
component0(
	[component11,
	 component12,
	 component13(
		 component21,
		 component22,
		 component23,
		 component24,
	 ),]
).render(by: context)
```