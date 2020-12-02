## 用于JSONDecoder
```swift
let appDecoder: JSONDecoder = {
    let decoder = JSONDecoder()
    // 将下划线的 Key 转换成 驼峰 Key
    // decoder.keyDecodingStrategy = .convertFromSnakeCase
    return decoder
}()
```

### 格式划 datetime 成为 RFC3339
```swift
class HelperDateTime {
    static func RFC339() ->String {
        // make time rfc3339
        let RFC3339DateFormatter = DateFormatter()
        RFC3339DateFormatter.locale = Locale(identifier: "en_US_POSIX")
        RFC3339DateFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZZZZZ"
        RFC3339DateFormatter.timeZone = TimeZone(identifier: "UTC")
        //let now = Date()
        //let date = RFC3339DateFormatter.string(from: now)
        
        // query
        let timestr = RFC3339DateFormatter.string(from: Date())
        return timestr
    }
}
```

## 获得 App Sanbox 的 Document Path
```swift
struct FileStore {
    
    var dirPath:String?
    
    var filePathMarketplaceList:String {
        "\(dirPath!)/marketplace.list.json"
    }
    
    var filePathProfileList:String {
        "\(dirPath!)/profile.list.json"
    }
    
    init(){
        
        // destination document path
        let documentDirectory = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)
        // transfer string to URL
        let ammDirectory = URL(string: documentDirectory[0])
        // append custom directory name
        let dataPath = ammDirectory?.appendingPathComponent("amm")
        
        // test
        if let dirPath = dataPath?.absoluteString {
            if !FileManager.default.fileExists(atPath: dirPath) {
                do{
                    try FileManager.default.createDirectory(atPath: dirPath, withIntermediateDirectories: true, attributes: nil)
                    self.dirPath = dirPath
                }catch{
                    print(error.localizedDescription)
                }
            }else{
                self.dirPath = dirPath
            }
        }
    }
}
```

## 将字符串复制进粘贴板
```swift
import Foundation
import AppKit

func copyToClipboard(url:String) {
    let pasteboard = NSPasteboard.general
    pasteboard.declareTypes([.string], owner: nil)
    pasteboard.setString(url, forType: .string)
    
//    var clipboardItems: [String] = []
//    for element in pasteboard.pasteboardItems! {
//        if let str = element.string(forType: NSPasteboard.PasteboardType(rawValue: "public.utf8-plain-text")) {
//            clipboardItems.append(str)
//        }
//    }
//
//    // Access the item in the clipboard
//    let firstClipboardItem = clipboardItems[0] // Good Morning
}
```

## 字符串加密 Hmac.sha256 & base64
```swift
// hmac.sha256
extension String {
     func sha256(key: String) -> (Data,String) {
        var digest = [UInt8](repeating: 0, count: Int(CC_SHA256_DIGEST_LENGTH))
        CCHmac(CCHmacAlgorithm(kCCHmacAlgSHA256), key, key.count, self, self.count, &digest)
        let data = Data(digest)
        return (data,data.map { String(format: "%02x", $0) }.joined())
    }
 }

// base64
func makeBase64Str(){
    self.signBase64Str = self.signSHA256Data.base64EncodedString()
}
```

## 一个 URLSession Publisher 与 sink
```swift
// publisher
func active(request:HttpRequest.Market)->AnyPublisher<HttpResponse.ResultString,Error> {
    // request
    var re = APIRequest.request(service: "serv_auth_manager", action: "mws/market/status")
    re.httpMethod = "PUT"
    let json = JSONEncoder()
    var body = [HttpRequest.Market]()
    body.append(request)
    let data = try? json.encode(body)
    re.httpBody = data!
    return URLSession.shared.dataTaskPublisher(for: re)
        .map{ data,header in
              return data
        }
        .decode(type: HttpResponse.ResultString.self, decoder: appDecoder)
           .receive(on: RunLoop.main)
           .eraseToAnyPublisher()
    }
    
// sink
func execute(in store: Store) {
    let token = SubscriptionToken()
    MwsPublisher.MwsMarket()
         .active(request: re)
         .sink { complete in
              print("\(complete)")
              token.unseal()
         } receiveValue: { value in
             if value.code != 0 {
//              store.appState.logModal = "\(value.msg ) | \(value.result ?? "")"
                store.dispatch(.marketplaceList)
             }
         }.seal(in: token)
    }
```