---
layout:     post
title:      "[개발이야기] Over the View: VoiceOver 오버뷰: 시리즈 1"
subtitle:   " \"블루펭귄 개발 과정\""
date:       2020-01-08 12:00:00
author:     "램프"
catalog: true
tags:
    - VoiceOver
    - Accessibility
    - Swift
    - iOS
    - 보이스오버
    - 접근성
    - 앱 개발
---

### 블루펭귄 개발 과정

주식회사 와들의 첫 iOS 앱인 **[블루펭귄](https://www.waddlelab.com/)**을 개발하는 과정을 담은 글이다.

### VoiceOver?

VoiceOver는 애플에서 시각장애인을 위해 제공하는 접근성 도구로, 화면을 음성으로 읽어주는 기능이다. 사용자의 제스처를 기반으로 기본적으로는 위에서 아래로, 왼쪽에서 오른쪽으로 화면을 탐색하며, 각 항목의 테두리가 볼드 처리되어 눈으로도 확인할 수 있다. 

iOS 사용자라면 누구나 설정 → 일반 → 접근성 → VoiceOver (iOS 13 이전) / 설정 → 손쉬운 사용 → VoiceOver (iOS 13 이후) 로 이동하여 기능을 활성화할 수 있으며, Siri에게 "VoiceOver 켜줘." 라고 말해도 가능하다. 

VoiceOver가 제공하는 기능은 제스처로 정해져 있으며 많이 알 수록 화면을 탐색하는 데 편리하다.

- 오른쪽/왼쪽으로 쓸어 넘기기 : 다음/이전 항목으로 이동
- 위/아래로 쓸어 넘기기 : 로터 설정을 사용하여 이전/다음 항목으로 이동
- 두 손가락 위/아래로 쓸어 넘기기 : 가장 위/선택한 항목에서부터 페이지 읽기
- 두 손가락으로 한 번 탭 : 말하기 일시 정지 또는 계속
- 두 손가락으로 두 번 탭 : 현재 동작을 시작하거나 정지
- 두 손가락으로 문지르기 : 뒤로가기
- 세 손가락 위/아래로 쓸어 넘기기 : 한 페이지 아래/위로 스크롤
- 세 손가락 오른쪽/왼쪽으로 쓸어 넘기기 : 한 페이지 왼쪽/오른쪽으로 스크롤
- 세 손가락으로 한 번 탭 : 항목 요약 읽기 (이미지 텍스트 추출)
- 세 손가락으로 두 번 탭 : 말하기 켬 또는 끔
- 세 손가락으로 세 번 탭 : 화면 커튼 켬 또는 끔
- 네 손가락으로 화면 상단/하단 한 번 탭 : 첫 번째/마지막 항목으로 이동
- 네 손가락으로 오른쪽/왼쪽으로 쓸어넘기기 : 다음/이전 앱으로 전환
- (반)시계 방향으로 회전 : 로터 설정 선택

### VoiceOver 최적화

- 화면 내 항목의 읽기 순서 정렬

    화면 내의 항목을 정해진 순서(상하좌우)로만 읽으면 이해하기 어려울 수 있다.

- 정확한 초점 이동

    화면 이동/버튼 클릭 등의 사용자의 동작에 의해 초점이 이상하게 이동할 수 있다. 이를 초점이 튄다고 표현한다.

- 불필요한 내용은 읽지 않도록 설정

    맥락을 파악하기 어렵다.

- 항목에 대한 추가적인 설명
- 초점 이동의 최소화

    원하는 정보를 얻기 위해 화면을 탐색하는 데 수많은 초점 이동이 필요할 수 있다.

### UIAccessibilityElement Property

가장 기본적이면서도 설정을 따로 해주지 않으면 정확한 정보를 제공해줄 수 없을 뿐 아니라 '버튼인지 텍스트필드인지', '누르면 어떻게 되는지' 등 사용자가 예측하게 만들어 불편함을 줄 수 있는 부분이다.

- ```var isAccessibilityElement: Bool```

    UIKit의 객체는 기본적으로 true 값을 가지며, 보통 불필요한 내용을 읽어주지 않게 하기 위해 false로 이 값을 변경한다.

        @IBOutlet weak var detail: UILabel!
        detail.isAccessibilityElement = false

- ```var AccessibilityLabel: String?```

    해당 항목에 초점이 맞춰졌을 때 어떤 내용을 읽어줄 지 결정한다. 짧고 간단하게 정하는 것이 좋으며 UIKit의 객체는 제목(title)이 기본값으로 설정된다.

        @IBOutlet weak var saveBtn: UIButton!
        saveBtn.accessibilityLabel = "Save"

- ```var AccessibilityTraits: UIAccessibilityTraits```

    버튼, 머리말, 텍스트필드 등 사용자가 인터랙션하는 데 도움을 줄 수 있는 특성을 나타낸다.

    button, header, image, link, searchField, selected, staticText 외에도 매우 많다.

    아래 예시는, tableView의 cell을 클릭했을 때 동작이 있는 경우 각 cell의 accessibilityTraits를 .button으로 설정해 주어야 한다는 뜻이다.

        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) {
        	let cell = tableView.dequeueReusableCell(withIdentifier: "identifier") as! ExampleCell
        	cell.accessibilityTraits = .button
        }

- ```var AccessibilityHint: String?```

    VoiceOver가 AccessibilityLabel 이후에 읽어주는 텍스트이다. Label을 읽은 후 약간의 간격을 두고 읽어주기 때문에 듣지 못할 수 있으며, 활성화하지 않은 사용자는 아예 들을 수 없다. 명료하고 짧을 수록 편리하다. '상세 설명을 보려면 이중 탭 하십시오.', 등 도움말의 의미를 가진 텍스트를 지정해주면 좋다.

        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) {
        	let cell = tableView.dequeueReusableCell(withIdentifier: "identifier") as! ExampleCell
        	cell.accessibilityTraits = .button
        	cell.accessibilityHint = "상세 설명을 보려면 이중 탭 하십시오."
        }