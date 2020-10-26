---
title:  Codable
date:  2018-08-17 17:06
categories:
- Swift
tags: 
- Swift
---

![](https://upload-images.jianshu.io/upload_images/3340896-f2a235b5b168f084.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 前言
*Objective-C*  中我们有好多 *JSON* 转 *Model* 第三方,比如 *JSONModel*, *MJExtension*,*YYModel*等好用的第三方库,在 **Swift 4** 推出了 *Codable* 协议,只要遵循 **Codable** 协议的模型就可以用*JSONEncoder* 和 *JSONDecoder* 两个类来实现 *JSON* 和 *Model* 的相互转换.


**Codable** 协议分为两部分,**Encodable**和**Decodable**;如果只需要解析JSON数据,那么只遵循 **Decodable** 协议就行了,如果 *JSON* 和 *Model* 都需要相互转换,那么遵循 **Codable** 协议

## **JSON** & **Model** 

##### 支持类型
下面代码实现了 *JSON* 字符串转为 *Student* , **Swift**  标准库里的基本类型已经都遵循 **Codable** 协议,所以,只要用遵循 **Codable** 协议的类型来创建模型,就可以正常读取json中的内容.

示例代码中的 **Student** 模型里面包含 *Int*, *Float*, *Bool*, *String* 和自定义的枚举类型.可选类型也是支持的,如果json中读取的key对应value可能为空,那么我们可以声明该类型为可选类型.

##### 嵌套Model
如果模型有嵌套自定义的类型, 自定义的枚举类型如果也遵循 **Codable** 协议,那么添加到模型中也可以正常读取,枚举的 *rawValue* 需要指定为遵循 **Codable** 协议的基本类型.

##### 自定义Model的key
如果需要自定义字段来解析 *JSON* 数据,实现 **CodingKey** 那个协议就OK,有一点不太友好的是,不管你是否自定义某个key,都要把模型中所有的key填写进去,否则会报编译错误.

## 示例代码

```
enum Gender: String, Codable {
    case male = "male"
    case female = "female"
}

struct Student: Codable {
    var name       : String
    var gender     : Gender
    var age        : Int
    var weight     : Float
    var isRegisted : Bool
    var score      : Double?
    
    enum CodingKeys: String, CodingKey {
        case gender
        case name
        case age
        case score
        case weight
        case isRegisted = "is_registed"
    }
}

let json = """
                {
                    "name"  : "Durian",
                    "gender": "male",
                    "age"   : 12,
                    "weight": 56.4,
                    "is_registed": true
                    "score" : null
                }
                """.data(using: .utf8)!
        
        let decoder = JSONDecoder()
        let encoder = JSONEncoder()
        encoder.outputFormatting = .prettyPrinted
        
        do {
            //json转Model
            let student =  try decoder.decode(Student.self, from: json)
            print(student)
            do {
                //model转JSON
                let json =  try encoder.encode(student)
                print(String(data: json, encoding: .utf8)!)
            } catch {
                print(error)
            }
        } catch {
            print(error)
        }

```

## One More Thing
*Codable*  不仅仅对 *JSON* 模型转换有很好的支持,对于 *PropertyList* 与模型的转换也很友好,我们只需要使用 *PropertyListEncoder*  和  *PropertyListDecoder*  替换 *JSONEncoder* 和 *JSONDecoder* 就行了,用法是一样的.

当然, *Codable* 的强大之处不止这些,阅读 *Apple* 官方文档,[了解更多...](https://developer.apple.com/documentation/swift/codable)

## 参考资料

[Encoding and Decoding Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types#overview)
[JSONEcoder](https://developer.apple.com/documentation/foundation/jsonencoder)
[JSONDecoder](https://developer.apple.com/documentation/foundation/jsondecoder)
[Swift 4 踩坑之 Codable 协议](https://www.jianshu.com/p/bdd9c012df15)
