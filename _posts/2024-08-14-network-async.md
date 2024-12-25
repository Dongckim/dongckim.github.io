---
title: "iOS 스터디 week6. 스위프트 네트워크와 비동기 프로그래밍"
layout: single
Typora-root-url: ../
categories: iOS
tag: [TIL, network]
use_math: true
---

## 네트워킹

네트워크 연결을 통해 받은 JSON형태 데이터를 다시 클래스나 구조체의 형태로 변환하는 것은 매우 어려운 일이다. 특히 이 작업을 하나하나 손으로 해야한다면 말이다.

[구조체로 바꿔주는 사이트]({{site.url}}/images/https:/2024-08-14-network-async/app.quicktype.io/)

↑ 위 사이트를 이용한다면 더 쉽게 형태를 바꾸어 서버에서 보내준 데이터를 사용할 수 있을 것이다.

자 이제 우리가 받아온 데이터를 우리가 쓰기 좋게 변환하는 과정(분석)을 해보자. 일단의 예전의 형태를 먼저 보면,
```swift

// 예전의 형태
func parseJSON2(_ movieData: Data) -> [DailyBoxOfficeList]? {
    
    do {
        
        var movieLists = [DailyBoxOfficeList]()
        
        // 스위프트4 버전까지
        // 딕셔너리 형태로 분석
        // JSONSerialization
        if let json = try JSONSerialization.jsonObject(with: movieData) as? [String: Any] {
            if let boxOfficeResult = json["boxOfficeResult"] as? [String: Any] {
                if let dailyBoxOfficeList = boxOfficeResult["dailyBoxOfficeList"] as? [[String: Any]] {
                    
                    for item in dailyBoxOfficeList {
                        let rank = item["rank"] as! String
                        let movieNm = item["movieNm"] as! String
                        let audiCnt = item["audiCnt"] as! String
                        let audiAcc = item["audiAcc"] as! String
                        let openDt = item["openDt"] as! String
                        
                        // 하나씩 인스턴스 만들어서 배열에 append
                        let movie = DailyBoxOfficeList(rank: rank, movieNm: movieNm, audiCnt: audiCnt, audiAcc: audiAcc, openDt: openDt)

                        
                        movieLists.append(movie)
                    }

                    return movieLists

                }
            }
        }

        return nil
        
    } catch {
        
        return nil
    }
    
}
```
너무 복잡하다... 현재의 스위프트에서는 매우 간략하게 바뀔 수 있다.

```swift
// 현재의 형태
func parseJSON1(_ movieData: Data) -> [DailyBoxOfficeList]? {
    
    do {
        // 스위프트5
        // 자동으로 원하는 클래스/구조체 형태로 분석
        // JSONDecoder
        let decoder = JSONDecoder()
        
        let decodedData = try decoder.decode(MovieData.self, from: movieData)

        return decodedData.boxOfficeResult.dailyBoxOfficeList
        
    } catch {
        
        return nil
    }
    
}

```
궁극적으로 배열로 반환된 데이터를 볼 수 있다. 이때 좀 살펴봐야하는 것이 Decodable이라는 프로토콜이 있는데, 이는 위에어 JSON Decoder를 선택할 때 필요한 프로토콜이라고 생각하면 되겠다. 마찬가지로 Encodable 이라는 프로토콜도 존재하는데, Decodable과는 반대로 구조체나 클래스를 데이터의 형태로 변형시켜주는 프로토콜이라고 한다.

- decode(변형하고 싶은 객체, from: 데이터) 메서드
- Encodable이랑 Decodable 두 개를 합쳐서 Codable이라는 프로토콜로 치환될 수 있다.

### dump
![]({{site.url}}/images/https:/2024-08-14-network-async/dump.png)

보통 안에 내용을 미리보기 느낌으로 보려면 print()함수를 많이 사용한다. dump도 동일한 함수인데, 다만 안에 있는 데이터를 좀더 보기 좋게, 더 자세하게 출력해주는 함수인 것이다.


## 네트워크 통신의 예시
```swift
struct MovieDataManager {
    
    let movieURL = "http://kobis.or.kr/kobisopenapi/webservice/rest/boxoffice/searchDailyBoxOfficeList.json?"
    
    let myKey = "7a526456eb8e084eb294715e006df16f"
    
    func fetchMovie(date: String, completion: @escaping ([Movie]?) -> Void) {
        let urlString = "\(movieURL)&key=\(myKey)&targetDt=\(date)"
        performRequest(with: urlString) { movies in
            completion(movies)
        }
    }
    
    func performRequest(with urlString: String, completion: @escaping ([Movie]?) -> Void) {
        print(#function)
        
        // 1. URL 구조체 만들기
        guard let url = URL(string: urlString) else { return }
        
        // 2. URLSession 만들기 (네트워킹을 하는 객체 - 브라우저 같은 역할)
        let session = URLSession(configuration: .default)
        
        // 3. 세션에 작업 부여
        let task = session.dataTask(with: url) { (data, response, error) in
            if error != nil {
                print(error!)
                completion(nil)
                return
            }
            
            guard let safeData = data else {
                completion(nil)
                return
            }
            
            
            // 데이터 분석하기
            if let movies = self.parseJSON(safeData) {
                //print("parse")
                completion(movies)
            } else {
                completion(nil)
            }
        }
        
        // 4.Start the task
        task.resume()   // 일시정지된 상태로 작업이 시작하기 때문
    }
    
    
    func parseJSON(_ movieData: Data) -> [Movie]? {
        // 함수실행 확인 코드
        print(#function)
        
        let decoder = JSONDecoder()
        
        do {
            let decodedData = try decoder.decode(MovieData.self, from: movieData)
            
            let dailyLists = decodedData.boxOfficeResult.dailyBoxOfficeList
            
            // 반복문으로 movie배열 생성 ⭐️
//            var myMovielists = [Movie]()
//
//            for movie in dailyLists {
//
//                let name = movie.movieNm
//                let rank = movie.rank
//                let openDate = movie.openDt
//                let todayAudi = movie.audiCnt
//                let accAudi = movie.audiAcc
//
//                let myMovie = Movie(movieNm: name, rank: rank, openDate: openDate, audiCnt: todayAudi, accAudi: accAudi)
//
//                myMovielists.append(myMovie)
//            }
            
            // 고차함수를 이용해 movie배열 생성하는 경우 ⭐️
            let myMovielists = dailyLists.map {
                Movie(movieNm: $0.movieNm, rank: $0.rank, openDate: $0.openDt, audiCnt: $0.audiCnt, accAudi: $0.audiAcc)
            }
            
            return myMovielists
            
        } catch {
            //print(error.localizedDescription)
            
            // (파싱 실패 에러)
            print("파싱 실패")
            
            return nil
        }
        
    }
    
}

```
코드 자체는 너무나 길지만 예시로서만 보는 걸로 하자.

- 주석을 달았던 것은, 앞에서 배웠던 고차함수를 바탕으로 치환될 수 있다. (고차함수를 배우는 이유)
- 클로저에서 객체의 속성이나 메서드에 접근하려면 self 키워드가 붙어줘야만 한다.

<Br/>

## 비동기 처리가 필요한 이유

서버에 데이터를 요청하는 일은 부하가 많이 걸리는 일이다. 이를 만약 비동기처리를 하지 않는다면, 테이블 뷰를 스크롤 할 때마다 버벅일 것이다.

### 소프트웨어적인 Thread

사실 빨리 종료되는 일은 1번 쓰래드에서만 처리해도 문제가 없음. 하지만, 네트워크 작업은 훨씬 오래 걸리는 일이다. 1번에서만 일이 진행이 되면 과부하가 걸리기 마련. 

### 앱의 시작과정과 동작원리
앱의 시작(과정)과 화면을 다시 그리는 원리 - 메인 쓰레드의 역할을 봐야하는데, 전체를 자세히 알아본다기 보단 간단히 살펴만 봐보자.
- 앱 객체를 생성해서 앱 실행 준비를 한다. c언어 기반의 main()함수가 존재하는데, UIKIT이 직접 관리한다. 
- 이를 바탕으로 화면이 준비가 되어지고, 런루프를 생성한다. `반복문` 과 동일한 것인데, 무한 반복문이라고 생각하면 편할 것 같다. 
- iOS Operating System에서는 포트(port)를 바탕으로 특정 프로세스를 식별하는 논리 단위를 생성하는데, 이런 특정 이벤트들이 이벤트 소스로 `큐`의 형태로 쌓이게 된다.
- 큐에 하나하나 쌓인 이벤트들은 다시 메인 런루프(객체)에 던져지면서 실행됨.
- 유저의 행동 하나하나를 즉각 반영하는 원리라고 보면 된다. 

![]({{site.url}}/images/https:/2024-08-14-network-async/runloop.png)
앱이 실행 중인 동안의 화면이다. 
- 앱이 시작될 때 앱을 담당하는 메인 런루프(반복문)가 생김.
- 이벤트 처리를 담당 → 어떤 함수를 실행시킬 것인지 선택/실행
- 함수 등의 실행의 결과를 화면에 보여줘야함. → 화면 다시 그림.

### 소프트웨어적인 Thread, 비동기가 필요한 이유
![]({{site.url}}/images/https:/2024-08-14-network-async/mainThread.png)
직접 그리진 않지만, 내부적인 메커니즘에 의해 렌더링 프로세스(코어애니메이션 → 렌더서버 → GPU → 표시)가 진행됨.
- 아주 잠깐 잠깐의 비는 시간에 화면을 다시 그리는 셈. → 버벅거릴 수 밖에 없다.

<br/>

### 그럼 분산처리를 해볼까?
메인 쓰레드는 1초 60번 화면을 다시 그려야하는 역할도 하기 때문에 너무 오래걸리는 작업들을 시키면 안된다.
- 분산처리를 어떻게 하는지에 대한 코딩 방법론을 **비동기 처리/동시성 프로그래밍**이라고 한다. 

iOS에서의 동시성을 처리하는 방법은 타 언어에 비해 매우 쉬운 편이라고 한다. 작업(TASK)을 "대기행렬"에 보내기만 하면, 운영체제가 알아서 여러 개의 쓰레드를 나눠서 분산처리(동시적 처리)를 한다.
- 큐가 항상 선입선출의 방식으로 자동 동작하게 한다.

<br/>

### 동시성 처리(iOS에서 동시성 처리하는 방법)
iOS 프로그래밍에서는 대기열에 크게 2가지 종류가 있다.
1. DispatchQueue(GCD)
2. OperationQueue(여기서는 알 수준이 아니다.)

- 직접적으로 쓰레드를 관리하는 개념이 아닌, **대기열(Queue)의 개념을 이용해서, 작업을 분산처리하고, OS에서 알아서 쓰레드 숫자(갯수)를 관리**
- (쓰레드 객체를 직접 생성시키거나 하지 않는) 쓰레드 보다 더 높은 레벨/차원에서 작업을 처리 
- 메인쓰레드(1번)가 아닌 다른 쓰레드에서 **오래걸리는 작업(예: 네트워크 처리)들**과 같은 작업들이 **쉽게 비동기적으로 동작하도록 함.**

![]({{site.url}}/images/https:/2024-08-14-network-async/thread.png)

### 병렬과 동시성의 개념
**Parallel VS Concurrency**
1. 병렬(Parallel) : 물리적인 쓰레드에서 실제 동시에 일을 하는 개념
2. 동시성(Concurrency) : 메인 쓰레드가 아닌 다른 소프트웨어적인 쓰레드에서 동시에 일을 하는 개념

결국 동시성이 개발자가 신경써야 하는 영역이다. 물리적인 쓰레드를 알아서 switching하면서 엄청나게 빠르게 일을 처리 (예를 들어, 2개의 쓰레드에서 일을해도 내부적인 물리적인 쓰레드는 1개만 동작 하고 있을 수도 있음)

<Br/>

## 비동기 VS 동기
비동기 처리(Async)는 일을 다른 쓰레드에 시작을 시키고 나는 기다리지 않는다. 이런 개념이라고 생각하기 편함.

그렇다면 반대로 동기는, 작업을 시작시키고, 뿐만 아니라 해당 작업이 끝날 때까지 기다린다. 라는 개념.

### 제어권과도 관련이 될 수 있음.
CPU의 제어권과도 관련된 개념이 될 수 있다. 이 떄 나타나는 개념이 Blocking 과 Non-Blocking의 개념이다. 동기 처리의 경우 다른 일을 하지 못하고, 작업이 끝날 때까지 기다리기 때문에, CPU 제어권이 바로 반환되지 않기 때문에, Blocking이라고 볼 수 있다. 반면 비동기 처리의 경우, 다른 일을 할 수 있도록 CPU 제어권이 바로 반환되기 때문에, Non-Blocking이라고 생각할 수 있음. 

<br/>

## 직렬과 동시의 개념

이 두 개념은 `큐`에 대한 내용이다. 대기 큐가 어떤 성질을 갖고 있을까?
1. 직렬(serial)
(보통 메인에서) 분산처리 시킨 작업을 **"다른 한개의 쓰레드에서"** 처리하는 큐

2. 동시(Concurrent)
(보통 메인에서) 분산처리 시킨 작업을 **"다른 여러개의 쓰레드에서"** 처리하는 큐

분산처리 하려는 것이라면.. “Concurrent큐”가 무조건  좋아 보이는 것 같아보이는 데, 왜 직렬(Serial)큐가 필요할까?

```swift
DispatchQueue.global().async {
    task6()
}

DispatchQueue.global().async {
    task7()
}

DispatchQueue.global().async {
    task8()
}

DispatchQueue.global().async {
    task9()
}

DispatchQueue.global().async {
    task10()
}
```

**클로저는 작업을 하나로 묶음이다** 따라서, 내부적으로 동기적으로 동작할 수 밖에 없다.

위의 코드는 순서를 보장할 수 없다. 따로따로 진행이 되기 때문에.

### 직렬큐 vs 동시큐

```swift
let serialQueue = DispatchQueue(label:"serial") //label엔 아무거나 쓰면 됨.

serialQueue.async {
    task1()
}
serialQueue.async {
    task2()
}
serialQueue.async {
    task3()
}

```
비동기적으로 작업을 하기 위해 async메서드를 사용하지만, 그럼에도 불구하고 직렬 큐이기 때문에 순서대로 작업 완료를 볼 수 있다. (작업완료가 미리 되어있을 수도 있지만)

```swift
let concurrentQueue = DispatchQueue.global()

concurrentQueue.async{
    task1()
}

concurrentQueue.async{
    task2()
}

concurrentQueue.async{
    task3()
}

```

비동기적으로 동시큐로 작동하기 때문에 순서와 상관없이 완료되는 대로 보여진다.

<br/>

## GCD의 개념 및 종류

아까 봤던 큐안에는 DispatchQueue라는 게 있다고 했는데, 이 것도 3가지로 나누어진다.
1. 메인큐 -> 1번 쓰레드를 메인 큐로 생각하면 편함
2. 글로벌큐
3. 프라이빗(커스텀)큐  

### 글로벌큐와 Qos의 이해
큐의 종류가 여러개이며, 기본설정은 동시큐라고 생각하면 편하다. 특히 이 큐는 서비스 품질에 따라 쓰레드의 갯수 차이가 나는데, 밑의 예시를 봐보자.
![]({{site.url}}/images/https:/2024-08-14-network-async/qos.png)
qos의 종류에 따라 배정되는 쓰레드의 개수의 차이가 생긴다. → 작업의 품질이나 속도를 조절.

> **큐의 서비스 품질의 개념**
><br/>iOS가 알아서 우선적으로 중요한 일임을 인지하고 쓰레드에 우선순위를 매겨 더 많은 쓰레드를 배치하고 CPU의 배터리를 더 집중해서 사용하도록 해서 일을 빨리 끝내도록 하는 개념

![](table.png)

```swift
let userInteractiveQueue = DispatchQueue.global(qos: .userInteractive)
let userInitiatedQueue = DispatchQueue.global(qos: .userInitiated)
let defaultQueue = DispatchQueue.global()  // 디폴트 글로벌큐
let utilityQueue = DispatchQueue.global(qos: .utility)
let backgroundQueue = DispatchQueue.global(qos: .background)
let unspecifiedQueue = DispatchQueue.global(qos: .unspecified)

```

### 프라이빗 큐
커스텀으로 만드는 큐, 기본설정 직렬(Serial), QoS(설정가능)
허나 동시큐로도 커스텀할 수 있다는 점.


```toc

```
