# SendingPOSTRequest
based on - KxCoding

- 서버로 데이터를 전송할 때는 주로 POST나 PUT을 사용한다.
- 전송할 데이터는 Body 섹션에 입력한다.

- 전송 방법은 서버 구현에 따라 달라지기 때문에 서버 개발자가 제공한 문서를 통해 API 사양을 확인해야 한다
- raw를 선택하고, 키(KEY)와 값(VALUE)을 입력한다
- 코드
```swift
import UIKit

struct PostData: Codable {
  let a: Int
  let b: Int
  let op: String
}

class PostRequestViewController: UIViewController {
  @IBOutlet weak var leftField: UITextField!
  @IBOutlet weak var rightField: UITextField!
  
  func encodedData() -> Data? {
    guard let leftStr = leftField.text, let a = Int(leftStr) else {
      showErrorAlert(with: "Please input only the number.")
      return nil
    }
    
    guard let rightStr = rightField.text, let b = Int(rightStr) else {
      showErrorAlert(with: "Please input only the number.")
      return nil
    }
    
    let data = PostData(a: a, b: b, op: "+")
    
    let encoder = JSONEncoder()
    return try? encoder.encode(data)
  }
  
  @IBAction func sendPostRequest(_ sender: Any) {
    guard let url = URL(string: "https://kxcoding-study.azurewebsites.net/api/calc") else {
      fatalError("Invalid URL")
    }
    
    // Code Input Point #1
    var request = URLRequest(url: url)
    // JSON으로 형식을 지정하는 코드
    request.addValue("application/json", forHTTPHeaderField: "Content-Type")
    // HTTP 메소드를 POST로 설정
    request.httpMethod = "POST"
    request.httpBody = encodedData() // 여기에 파라미터 값 들어간다
    
    let task = URLSession.shared.data(with: request) { (data, response, error) in
        if let error = error {
            self.showErrorAlert(with: error.localizedDescription)
            print(error)
            return
        }
        
        guard let httpResponse = response as? HTTPURLResponse else {
            self.showErrorAlert(with: "Invalid Response")
            return
        }
        
        guard (200...299).contains(httpResponse.statusCode) else {
            self.showErrorAlert(with: "\(httpResponse.statusCode)")
            return
        }
        
        guard let data = data, let str = String(data: data, encoding: .utf8) else {
          fatalError("Invalid Data")
        }
        
        self.showInfoAlert(with: str) // 두 수의 연산 결과가 나옴
    }
    
    task.resume()

    
  }
}
```
