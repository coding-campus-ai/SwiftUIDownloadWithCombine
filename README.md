Combine프레임워크를 사용한 JSON Data 다운로드
이번에는 Comebine프레임워크를 사용하여 코딩을 해보도록 하겠습니다. 이번 게시글을 통해 Combine프레임워크가 얼마나 유용하고 강력하고 간결하고 부드러운지 알 수 있을 거라고 생각됩니다. 
우선 시작하기전에 JSON Data를 @escaping을 사용하여 다운받아오는 방법에 대한것을 숙지하지는 것을 추천드립니다. 이번 게시글은 @escaping을 사용한 것과 새로운 프레임워크인 combine을 사용한 것을 비교하기 위한 포스팅입니다.
저는 개인적으로 combine을 사용해 JSON data를 다운로드해 가져오는 방법을 선호하는 편입니다. 왜냐하면 더 간결하고 구현하기 쉽기 때문이죠 :)
 

저번 게시물에서 사용했던 Fack JSON Data URL과 Model을 그대로 사용하도록 할게요.

guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else { return }

//MARK: MODEL
struct PostModel: Identifiable, Codable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
}

Fack JSON의 주소는 이곳 입니다.

import SwiftUI
 
// MARK: MODEL
// get post 1의 Json 데이터와 동일한 모델 생성
// postModel을 디코딩하고 인코딩을 원하기 때문에 codable 추가
 
struct PostModel: Identifiable, Codable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
}
 
class DownloadWithCombineViewModel: ObservableObject {
    init() {
        
    }
    
    func getPosts() {
        
    }
}
struct DownloadWithCombine: View {
    var body: some View {
        Text("Hello, World!")
    }
}

ViewModel에 PostModel 가지는 @Published를 생성해주고 빈 배열에 추가한 다음 Body에서 @StateObject로 ViewModel을 초기화해주겠습니다. 그리고 Body의 View까지 UI를 꾸며줄게요.

import SwiftUI
 
// MARK: MODEL
// get post 1의 Json 데이터와 동일한 모델 생성
// postModel을 디코딩하고 인코딩을 원하기 때문에 codable 추가
 
struct PostModel: Identifiable, Codable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
}
 
class DownloadWithCombineViewModel: ObservableObject {
    
    @Published var posts: [PostModel] = []
    
    init() {
        
    }
    
    func getPosts() {
        
    }
}
struct DownloadWithCombine: View {
    
    @StateObject var vm = DownloadWithCombineViewModel()
    
    var body: some View {
        NavigationView {
            List {
                ForEach(vm.posts) { post in
                    VStack(alignment: .leading, spacing: 10) {
                        Text(post.title)
                            .font(Font.title.bold())
                        Text(post.body)
                            .foregroundColor(Color(UIColor.systemGray2))
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                }
            }
            .navigationBarTitle("Fake JSON Data")
            .listStyle(PlainListStyle())
        }
    }
}

이제 함수 getPosts에서 필요한 것은 지난 게시글에서 했던 것과 같은 Fack URL변수를 넣어주는 것입니다.

    func getPosts() {
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else { return }
    }

그리고 이번 게시글에 핵심인 Combine을 사용할 건데, Apple Document를 확인해보죠.

솔직히 말해서 위에 말이 정확히 어떤 의미인지 조금은 헷갈립니다.. 그렇다면 이렇게 예시를 들어보죠!

Combine을 쉽게 이해하기 위한 예시
1. 어떠한 패키지에 대한 매월 정기구독을 가입함

2. 고객이 주문하면 회사는 패키지를 공장에서 생산함

3. 회사는 패키지를 배송하고 고객은 상품을 전달받음

4. 사용자는 패키지 상자의 상태가 손상되었는지 확인함

5. 상자를 열고 항목이 올바른지 확인함

6. 구입한 항목을 사용함!

7. 위 구독은 언제든지 취소가 가능함

 

대충 틀은 이렇다고 보시면 됩니다. 이제 이것을 차례대로 사용해보죠!!

 

1. Publisher생성 (dataTaskPublisher)
 " 어떠한 패키지에 대한 매월 정기구독을 가입함 "
지난 게시글에서는 URLSession의 dataTask를 사용했지만 이번에는 dataTaskPublisher을 사용합니다. 이제 Publisher은 시간이 지남에 따라 값을 게시할 것임을 알고 있습니다. 그리고 만들어준 url을 전달해주겠습니다. 

URLSession.shared.dataTaskPublisher(for: url)

2. subscribe publisher을 background Thread로 옮겨줌
 " 고객이 주문하면 회사는 패키지를 공장에서 생산함 "
이게 Publisher을 background Thread로 옮겨줘서 그곳에서 수행되도록 해주겠습니다. 실제로 dataTaskPublisher의 기본값은 background Thread입니다. 그래서 우리는 실제로 이 2번째 작업(subscribe)을 할 필 요는 없지만 때로는 명시적으로 background Thread에 있지 않을 수 있으므로 이 작업을 해주는 것이 중요합니다.

   

.subscribe(on: option:)을 사용하고 option은 사용하지 않을 것이기 때문에 이 부분은 지워주고 DispatchQueue를 사용하여 추가합니다.

.subscribe(on: DispatchQueue.global(qos: .background))

3. Main Thread에서 수신함
 " 회사는 패키지를 배송하고 고객은 상품을 전달받음 "
자, 고객이 상품을 전달받았다는 것은 UI가 업데이트되었다는 의미죠? UI가 업데이트되려면 어느 Thread에서 작업해야 한다고요?! 

 

그렇죠! Main Thread입니다! receive를 사용할게요.

.receive(on: DispatchQueue.main)

4. tryMap (data가 있는지 없는지 상태가 좋은지 확인)
 " 사용자는 패키지 상자의 상태가 손상되었는지 확인함 "
상자를 받았으니 상자의 상태를 확인해야 합니다. 이것이 바로 tryMap입니다.    

.tryMap을 입력하고 엔터를 치면 완성된 코드에 두 개의 조각이 있습니다. 

우리는 이곳에 각각 data와 response를 넣어줄 것입니다. 

TryMap은 실패하고 오류를 발생시킬 수 있는 맵입니다.
            .tryMap { data, response in
               //some code
            }

기본적으로 이 tryMap에 어떤 것을 가지고 있는지 정해줄 수 있는데 Data를 넣어주도록 하겠습니다.

.tryMap { data, response -> Data in {
 
}

그리고 저번 게시글에서도 했듯이 응답 상태에 관한 guard를 넣어주면 되겠네요.

            .tryMap { data, response -> Data in
                guard
                    let response = response as? HTTPURLResponse,
                    response.statusCode >= 200 && response.statusCode < 300 else {
                    
                    }

이렇게 응답을 받아왔고, 만약 응답이 실패한다면 오류를 던져줘야겠죠? throw URLError를 입력 후 괄호를 열고 엔터를 쳐주고 URLError에 오른쪽 마우스를 클릭해 Jump to Definition을 눌러줄게요.

            .tryMap { data, response -> Data in
                guard
                    let response = response as? HTTPURLResponse,
                    response.statusCode >= 200 && response.statusCode < 300 else {
                        throw URLError(URLError.Code)
                    }
                   return
            }

그리고 Foundation의 extension URLError {} 을 보면 모든 에러 유형이 나오는데 우리는 서버 응답이 안 좋다는 에러를 보내줘야 하기 때문에 그에 맞는 에러를 찾아주면 될 거 같아요. badServerResponse가 맞겠네요.

그런 다음 data를 반환해줍니다.

            .tryMap { data, response -> Data in
                guard
                    let response = response as? HTTPURLResponse,
                    response.statusCode >= 200 && response.statusCode < 300 else {
                        throw URLError(.badServerResponse)
                    }
                return data
            }

이렇게 tryMap으로 데이터가 있는지 확인 -> 응답 상태가 좋은지 확인 -> 만약 좋지 않다면 무슨 오류가 났는지 확인 -> data 반환 이런 식으로 코드가 작성되었네요!

5. decode (데이터를 PostModel로 디코딩)
 " 상자를 열고 항목이 올바른지 확인함 "
여기에서 상자란 PostModel을 말합니다. 고로 이제 return data의 data에 진짜 데이터가 있는지 확인해야 하기 때문에 PostModel을 디코딩해야 합니다. 

 

PostModel을 보면 이미 codable을 준수하고 있습니다. 그렇기에 우리는 이미 디코딩하고 인코딩할 수 있습니다. 

JSON Data를 잠깐 보면 배열 [ ] 을 사용하고 있기 때문에 decode type도 동일하게 [PostModel]을 넣어줘야 합니다. 

.decode(type: [PostModel].self, decoder: JSONDecoder())

6. sink(항목을 앱에 추가한다)
 " 구입한 항목을 사용함! "
마지막으로 이 항목을 동기화시킬 수 있습니다. 기본적으로 항목이 앱에 저장되는 것이죠. 즉, PostModel을 가져와서 앱에서 사용하기 때문이죠. 이것을 사용하려면 .sink 괄호 후 엔터를 누르면 자동으로 receiveCompletion이 완성됩니다. 이 이름은 Completion으로 지정해주고 receiveValue는 [PostModel] 이므로 우리는 completion의 코드를 먼저 작성해주는 게 좋겠네요.

이제 receiveValue에는 반환된 포스트 값을 넣어줄 건데 아래와 같이 코드를 수정해주겠습니다.

            .sink { completion in
                print("Completion: \(completion)")
            } receiveValue: { returnedPost in
                self.posts = returnedPost
            }

여기서 self는 지금 강한참조를 하고 있습니다. 저번 게시글에서도 언급했듯이 우리는 앱에서 강력한 것을 원하지 않는 상황이 종종 있습니다. 참조는 자신의 기억을 유지하기 때문입니다. 바로 약한참조로 바꿔주도록 할게요. 

            .sink { completion in
                print("Completion: \(completion)")
            } receiveValue: { [weak self] returnedPost in
                self?.posts = returnedPost
            }

7. store (필요한 경우 구독을 취소함)
 " 위 구독은 언제든지 취소가 가능함 "
.store(In : &Set<AnyCancellable>)을 사용하여 언제든 구독을 취소할 수 있는데 우리는 아직 이 Set를 가지고 있지 않습니다. 그렇기 때문에 ViewModel에 Combine을 사용하여 몇 가지를 추가해줘야 합니다.

 

우선 combine을 import 해줍니다. 그리고 ViewModel에 다음과 같이 추가해줍니다.

class DownloadWithCombineViewModel: ObservableObject {
    
    @Published var posts: [PostModel] = []
    var cancellables = Set<AnyCancellable>()
    
}

이제 이 cancellables를 .store에 사용할 수 있게 됐네요.

.store(in: &cancellables)

8. 시뮬레이터 실행
자! 여기까지 하고 시뮬레이터를 실행해볼까요?

 

현재까지의 코드는 이렇습니다.

import SwiftUI
import Combine
 
//MARK: MODEL
struct PostModel: Identifiable, Codable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
}
 
class DownloadWithCombineViewModel: ObservableObject {
    
    @Published var posts: [PostModel] = []
    var cancellables = Set<AnyCancellable>()
    
    init() {
        getPosts()
    }
    
    func getPosts() {
        guard let url = URL(string: "https://jsonplaceholder.typicode.com/posts") else { return }
        
        URLSession.shared.dataTaskPublisher(for: url)
            .subscribe(on: DispatchQueue.global(qos: .background))
            .receive(on: DispatchQueue.main)
            .tryMap { data, response -> Data in
                guard
                    let response = response as? HTTPURLResponse,
                    response.statusCode >= 200 && response.statusCode < 300 else {
                        throw URLError(.badServerResponse)
                    }
                return data
            }
            .decode(type: [PostModel].self, decoder: JSONDecoder())
            .sink { completion in
                print("Completion: \(completion)")
            } receiveValue: { [weak self] returnedPost in
                self?.posts = returnedPost
            }
            .store(in: &cancellables)
 
    }
}
struct DownloadWithCombine: View {
    
    @StateObject var vm = DownloadWithCombineViewModel()
    
    var body: some View {
        NavigationView {
            List {
                ForEach(vm.posts) { post in
                    VStack(alignment: .leading, spacing: 10) {
                        Text(post.title)
                            .font(Font.title.bold())
                        Text(post.body)
                            .foregroundColor(Color(UIColor.systemGray2))
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                }
            }
            .navigationBarTitle("Fake JSON Data")
            .listStyle(PlainListStyle())
        }
    }
}

성공적으로 JSON Data를 가져오는 데 성공했습니다. 

 

9. 만약 호출에 성공하지 못했다는 가정하에 몇 가지 로직을 넣음
끝마치기 전에 한 가지 해줘야 할 것이 있습니다. 우리는 지금 Fack JSON Data를 가져왔고 이것이 정확하기 때문에 문제없이 출력될 수 있었습니다. 하지만 그렇지 않은 경우 에러가 날 수도 있겠죠? 만약 출력에 성공하지 못했다면 아래에 몇 가지 로직을 넣을 필요가 있을 것 같네요.

 

수정해줘야 할 부분은 바로 tryMap 부분을 함수로 만들어 주려고 합니다. 잠시 아무 곳에 tryMap을 넣어줘 볼게요.

바로 이 부분을 함수에 복사하여 넣어줄 것입니다.

    func handleOutput(output: URLSession.DataTaskPublisher.Output) throws -> Data  {
        
    }

이제 tryMap의 guard부분을 복사해서 handleOutput 함수에 붙여 넣어 주겠습니다.

    func handleOutput(output: URLSession.DataTaskPublisher.Output) throws -> Data  {
        guard
            let response = response as? HTTPURLResponse,
            response.statusCode >= 200 && response.statusCode < 300 else {
                throw URLError(.badServerResponse)
            }
        return data    
    }

response와 return data쪽에 오류가 났는데, 이 앞쪽에 각각 output.을 넣어줄게요.

    func handleOutput(output: URLSession.DataTaskPublisher.Output) throws -> Data  {
        guard
            let response = output.response as? HTTPURLResponse,
            response.statusCode >= 200 && response.statusCode < 300 else {
                throw URLError(.badServerResponse)
            }
        return output.data
    }

마지막으로 getPost함수의 .tryMap의 코드를 지우고 아래처럼 수정합니다.

.tryMap(handleOutput)

엄청 간단하고 깔끔해졌죠?

 

이렇게 Combine을 사용하여 JSON Data를 가져와봤는데 저번 게시글에서 했던 @escaping을 사용하여 JSON Data를 가져온 코드를 비교해서 살펴보는 것을 추천드립니다.

 

실제로 두 코드는 비슷하지만 확인해보면 Combine프레임 워크를 사용하는 것이 더 간편하고 깔끔하다는 것을 알 수 있습니다. 

