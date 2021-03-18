### SwiftUI OnBoarding Screen

>✔ **Xcode : Version 12.4 (12D4e)**
>
>✔ **OS :  macOS Big Sur 11.1**



#### Background

- 모바일 앱의 경우 개인 사용자들에게 UI / UX 튜토리얼 혹은 서비스 안내 등을 하나의 슬라이드 뷰로서 제공하기 좋음
- **SwiftUI** 에서는 `EnvironmentObject` 와 `ObservableObject` 를 제공하는데, 이를 이용해 `ViewRouter` 를 구현해 다양한 뷰 세그먼트를 구현할 수 있음

- 그 중 앱을 처음 실행했을 때, 튜토리얼 뷰 등을 띄워주기 위해 뷰 라우팅의 구현법을 알아볼 것



#### ViewRouter

~~~swift
import Foundation
import SwiftUI

class ViewRouter: ObservableObject {
		@Published var currentView: String
  	private var context = UserDefaults.standard
  
    init() {
        if !context.bool(forKey: "didLaunchBefore") {
            context.set(true, forKey: "didLaunchBefore")
            currentView = "Tutorial"
        } else {
            currentView = "MainView"
        }
    }
}
~~~

- `Published` 를 이용해 현재 보여줄 뷰 를 담고 있는 `ViewRouter` 의 구현
- `UserDefaults` 를 이용해 앱을 처음 실행하는지에 대한 체크를 진행하고 분기에 따라 튜토리얼 뷰에 대한 라우팅을 적용시킴
- 만약 처음 실행한다면 다음에 앱이 실행될 때는 튜토리얼이 뜨지 않도록 `UserDefaults` 에 저장함



#### ViewContainer

~~~swift
import SwiftUI

struct ViewContainer : View {
    @EnvironmentObject var viewRouter: ViewRouter
  
    var body: some View {
        VStack {
            if viewRouter.currentView == "Tutorial" {
                OnBoardingView()
            } else if viewRouter.currentView == "MainView" {
                MainView()
            }
        }
    }
}
~~~

- `ViewRouter` 를 이용해 뷰를 사용자에게 교체해서 보여줄 뷰 컨테이너를 구현
- 뷰 컨테이너는 현재 `ViewRouter` 의 `Published` 개체를 체크하며 해당 값이 변경될 경우 인지해 현재 표현할 뷰를 변경
- SwiftUI에서 Swift 의 `ViewSegment` 와 같이 뷰의 교체 / 이동 등을 구현할 때 용이함



#### SceneDelegate

~~~swift
...
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
	...
	window.rootViewController = UIHostingController(rootView: ViewContainer().environmentObject(ViewRouter()))
...
~~~

- `SceneDelegate` 에서 앱의 현재 UI ( Scene ) 이 생성될 때 하나의 ViewRouter 만을 보도록 변경

