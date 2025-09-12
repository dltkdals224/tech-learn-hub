# 크로스플랫폼과 하이브리드앱

## 크로스플랫폼과 하이브리드앱

> 크로스플랫폼과 하이브리드앱의 차이

- 크로스플랫폼

프로젝트 하나로 Android, iOS 모두 동작시키는 앱.  
ex) ReactNative, Flutter

- 하이브리드앱

웹(React)과 앱(보통 RN)을 함께 사용하여 동작시키는 앱.  

> 사용 프레임워크

ReactNative는 Expo 프레임워크를 사용.  
React는 Next 프레임워크를 사용.

<br/>

## ReactNative와 Expo

> 프로젝트 세팅

하나의 프로젝트 폴더 내 web과 mobile을 폴더로 분리하여 같이 둔다.

web: npx create-next-app@latest (강의에서는 14)  
mobile: npx create-expo-app --template tabs@sdk52

--template의 종류와 관련해서는 [Expo Documentation](https://docs.expo.dev/more/create-expo/#--template)을 참조.

> mobile 프로젝트 폴더 설명

```
mobile
├── app  // JS 등의 소스코드
│   ├── ...
│   └── ...
├── app.json  // expo 설정 적는 곳
└── babel.config.js  // babel 설정 적는 곳
```

<br/>

## 에뮬레이터와 시뮬레이터

> 프로젝트 실행 방법

- 실물 핸드폰(Android)
- 실물 핸드폰(iOS)
- 안드로이드 에뮬레이터
- iOS 시뮬레이터 <- 맥북만 가능
- ~~브라우저(크롬, 사파리)~~ <- 앱 수준의 테스트를 수행할 수 없음

수업: Android Emulator 기준  
개인: iOS device + Android device

> 디바이스 세팅

[Set up your environment](https://docs.expo.dev/get-started/set-up-your-environment/)메뉴얼 기준으로 진행

안드로이드 에뮬레이터에서 SDK manager / Virtual Device manager를 중심으로 접근.

> 프로젝트 실행

- 생성된 기본 프로젝트에서 `npx expo start`명령어 실행
- 실행 후, 추가 키 입력에 따라 적절한 디바이스에 화면 출력 가능

> 주의 사항

- 모바일 디바이스를 QR code로 연결하는 상황에서, 동일한 wifi 연결이 필요
- iOS는 Expo Go 앱을 설치한 이후에 카메라를 통한 QR 추적 가능

> 실습

- mobile 프로젝트에서 불필요한 부분들을 적절히 제거하는 과정 필요 (영상을 다시 참고)
- 모바일 프로젝트는 `_layout.tsx`와 `index.tsx`만으로 구성, 내부에서 웹뷰인 \<iframe/>으로 동작하게 하는 형태
- `_layout.tsx`에서 앱기능의 수행만 구현

<br/>

## 리액트네이티브 태그들

> ReactNative에서 사용되는 태그 종류

```typescript
import { SafeAreaView } from "react-native-safe-area-context"
import { StatusBar } from "expo-status-bar"


<SafeAreaView
    style={{ flex: 1 }}  // 상단 노치 영역을 포함한 전체 화면
    edges={["top"]}  // 노치 영역 컨트롤('top' => '노치 영역을 인정'하는 것)
>
    // 노치 영역 스타일 지정
    <StatusBar style="light"/>
</SafeAreaView>
```

모바일 폴더에서 components/ 하위에 section별로 분리하며 수업을 진행.  
이후의 내용은 해당 테스트 필드에서 직접 작성.

<br/>

## 웹뷰

하이브리드 앱의 구성을 위해서 `<WebView>`컴포넌트를 통해 Next 프로젝트를 감싼다.

`<WebView>`는 `npm install` 또는 `yarn add`가 아닌 `npx expo install`을 통해 다운받아야 한다.  
해당 프로젝트의 expo version dependencies에 맞춰서 다운받아야 하기 때문.  
따라서 `npx expo install react-native-webview`를 입력한다.

```typescript
import { SafeAreaView } from "react-native-safe-area-context";
import { WebView } from "react-native-webview";

const Section01_03 = () => {
  return (
    <SafeAreaView
      style={{ flex: 1, backgroundColor: "white" }}
      edges={["top", "bottom"]}
    >
      <WebView source={{ uri: "https://www.naver.com" }} />;
    </SafeAreaView>
  );
};

export default Section01_03;
```

- naver는 safety-area에 대해서 natvie로 처리.  
top: reload by scroll, bottom: navigation

<br/>

## 웹뷰와 Next 연동

> 주의사항

nextjs도 로컬에서 가동
모방리 기기에서는 localhost 추적 불가하여 wifi 기준의 ip주소 맞추기 작업 필요ㅣㅇ

<br/>

## 웹뷰 연결 실패
<br/>