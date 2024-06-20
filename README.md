# 2024-NC2-M38-SensitiveContentAnalysis
## 🎥 Youtube Link
(추후 만들어진 유튜브 링크 추가)

## 💡 About Augmented Reality
> **⚠️ 이미지나 비디오에서 불건전하거나 부적절한 콘텐츠를 감지하는 기능**

**이 기술은 어떻게 활용할 수 있을까요?**
> 
- 외부로부터 업로드되는 이미지나 영상이 민감한 정보를 담고 있는지를 체크하는 상황에 이 프레임워크를 활용할 수 있어요
- 메세지 앱(iMessage), 교실(Classroom) 앱 등에서 활용되고 있어요.
    - ex ) 메세지 앱에 업로드된 콘텐츠가 민감한 정보를 담고 있는지 체크함
    - ex ) 교실 앱에서는 개인 기기로부터 학급의 과제 제출 혹은 기타 학급 활동을 위한 공용 공간으로
    업로드된 콘텐츠가 민감한 정보를 담고 있는지를 판단하고, 개입할 수 있음

## 🎯 What we focus on?
- **이벤트 드리븐 상태 관리**
    - `@State` 변수를 사용하여 분석 상태(`AnalysisState`)를 관리하고, 분석 상태에 따라 UI를 업데이트합니다.
- **비동기 이미지 분석**
    - `isSensitive(image:)` 로 이미지 데이터를 받아 비동기적으로 민감 콘텐츠를 분석합니다.
- **조건부 UI 렌더링**
    - `analysisState`의 값에 따라 다른 UI 요소를 표시합니다.
    - 예를 들어, 분석 중일 때는 ProgressView를, 민감한 콘텐츠가 감지되었을 때는 블러 처리된 이미지를 보여줍니다.
- **애니메이션**
    - 민감 콘텐츠가 감지되었을 때, 블러 처리된 이미지를 클릭하면 애니메이션과 함께 블러를 해제합니다.

## 💼 Use Case
> 익명의 쪽지앱을 사용하다 누군가 보낸 민감한 사진을 필터링해주자!

## 🖼️ Prototype

<table>
  <tbody>
    <tr>
      <td colspan="1" align="center"><b>닉네임 등록</b></td>
      <td colspan="1" align="center"><b>쪽지 작성</b></td>
      <td colspan="1" align="center"><b>받은 쪽지함</b></td>
      <td colspan="1" align="center"><b>민감 정보 필터링</b></td>
    </tr>
    <tr>
      <td align="center"><img src="https://github.com/DeveloperAcademy-POSTECH/2024-NC2-M38-SensitiveContentAnalysis/assets/64794813/4752b940-733d-405c-a9c0-001cf48159e0" width="260px;" alt=""/></td>
      <td align="center"><img src="https://github.com/DeveloperAcademy-POSTECH/2024-NC2-M38-SensitiveContentAnalysis/assets/64794813/cfc11098-dfa8-48d3-9fe9-7a707bd44470" width="260px;" alt=""/></td>
      <td align="center"><img src="https://github.com/DeveloperAcademy-POSTECH/2024-NC2-M38-SensitiveContentAnalysis/assets/64794813/86161290-bd3b-4649-9dc3-83a825846f64" width="260px;" alt=""/></td>
      <td align="center"><img src="https://github.com/DeveloperAcademy-POSTECH/2024-NC2-M38-SensitiveContentAnalysis/assets/64794813/856f8183-a6e8-44c2-8ece-e559c5217732" width="260px;" alt=""/></td>
    </tr>
  </tbody>
</table>

## 🛠️ About Code
### 민감 정보인지 비동기적으로 확인

```swift
enum AnalysisState { // 분석 상태를 구분하기 위한 열거형
        case notStarted
        case analyzing
        case isSensitive
        case notSensitive
        case error(message: String)
    }
    
func isSensitive(image: Data?) async {  // 민감 정보인지 비동기적으로 확인
        analysisState = .analyzing
        let analyzer = SCSensitivityAnalyzer()
        let policy = analyzer.analysisPolicy
        
        if policy == .disabled {
            print("Policy is disabled")
        }
        
        do {
            guard let image = UIImage(data: image!)
            else {
                print("Image not found")
                analysisState = .error(message: "Image not found")
                return
            }
            let response = try await analyzer.analyzeImage(image.cgImage!)
            analysisState = response.isSensitive ? .isSensitive : .notSensitive
        } catch {
            print("Unable to get a response", error)
            analysisState = .error(message: error.localizedDescription)
        }
    }
```

- 분석상태는 `AnalysisState` enum으로 구분함
- `isSensitive` 함수는 비동기적으로 실행됨
    - 이 함수는 비동기적으로 이미지 데이터를 분석하여 민감 정보를 포함하고 있는지 확인하고, 그 결과에 따라 상태를 업데이트 함
    - 비동기적 처리는 네트워크 호출이나 시간이 오래 걸릴 수 있는 작업에서 UI가 중단되지 않고 원활하게 동작할 수 있도록 도와줌

### 응답을 바탕으로 필터링 적용
```swift
 case .isSensitive:
                        if let imageData = post.image, let uiImage = UIImage(data: imageData) {
                            ZStack {
                                Image(uiImage: uiImage)
                                
                                // 민감 정보 블러 처리
                                if !showSensitiveContent {
                                    Color.black.opacity(0.5)
                                        .frame(height: 200)
                                        .cornerRadius(10)
                                    
                                    // 블러 이미지
                                    VStack {
                                        Image(systemName: "eye.slash")
                                            .font(.system(size: 50))
                                            .foregroundColor(.white)
                                        Text("Sensitive Content")
                                            .font(.title)
                                            .foregroundColor(.white)
                                            .padding(.top, 10)
                                        Text("This photo may contain graphic or violent content.")
                                            .font(.body)
                                            .foregroundColor(.white)
                                            .padding(.top, 5)
                                        Button(action: {
                                            withAnimation {
                                                showSensitiveContent = true
                                            }
                                        }) {
                                            Text("See why")
                                                .foregroundColor(.white)
                                                .padding()
                                                .background(Color.gray.opacity(0.7))
                                                .cornerRadius(10)
                                                .padding(.top, 20)
                                        }
                                    }
                                    .padding()
                                    .background(Color.black.opacity(0.5))
                                    .cornerRadius(20)
                                }
                            }
                        }
```

- `SCSensitivityAnalyzer`에서 policy에 대한 응답을 받은 후
    - `isSensitive` 상태일 때 즉, 민감정보일 경우에 해당 이미지를 블러처리 해줌
