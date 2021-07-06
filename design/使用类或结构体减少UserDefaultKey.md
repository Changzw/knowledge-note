
# 使用结构管理 UserDefault，减少 UserDefaultKey

## 场景

当有一类功能想要使用 UserDefault 来缓存操作记录的时候，会产生大量 UserDefaultKey，并且没有集中管理

## 解决方案

使用一个结构集中管理需要缓存的 UserDefaultKey，可内部进行逻辑操作

样例代码：

```swift
struct PermissionApplicantName: RawRepresentable {
  let rawValue: String
}

extension PermissionApplicantName {
  static let location = PermissionApplicantName(rawValue: "location")
  static let editHeadCamera = PermissionApplicantName(rawValue: "editHeadCamera")
  static let editBackgroundCamera = PermissionApplicantName(rawValue: "editBackgroundCamera")
  static let feedbackCamera = PermissionApplicantName(rawValue: "feedbackCamera")
  
  static let editHeadAblum = PermissionApplicantName(rawValue: "editHeadAblum")
  static let editBackgroundAblum = PermissionApplicantName(rawValue: "editBackgroundAblum")
  static let feedbackAblum = PermissionApplicantName(rawValue: "feedbackAblum")
}

struct PermissionAppliant {
  let id: PermissionApplicantName
  let tickets: Int
  
  init(id: PermissionApplicantName, tickets: Int = 3) {
    self.id = id
    self.tickets = tickets
  }
  
  private var refuseCountKey: String { id.rawValue + "refuseCountKey" }
  private var start48DateKey: String { id.rawValue + "start48DateKey" }

  private var refuseCount: Int {
    get { UserDefaults.standard.integer(forKey: refuseCountKey) }
    set { UserDefaults.standard.set(newValue, forKey: refuseCountKey) }
  }
  
  private var start48Date: Date? {
    get { UserDefaults.standard.object(forKey: start48DateKey) as? Date }
    set { UserDefaults.standard.set(newValue, forKey: start48DateKey) }
  }
  
  mutating func refuse() {
    refuseCount += 1
  }
  
  mutating func reset() {
    refuseCount = 0
    start48Date = nil
  }
}
```

使用方式：

fileA.swift

```swift
  private var A = PermissionAppliant(id: .editProfileLocation)
 
  func xxx() {
     A.refuse()
  }
```

fileB.swift

```swift
  private var B = PermissionBppliant(id: .editProfileLocation)
 
  func xxx() {
     B.refuse()
  }
```


