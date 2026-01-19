## 영상처리
- 디지털 카메라를 통하여 영상을 전자적으로 얻은 후 컴퓨터로 영상을 처리하여 원하는 출력 영상을 기술 분야

## openCV 사용해보기
### openCV VisualStudio 라이브러리 설정
#### 프로젝트 속성 설정
1. C/C++ → 추가 포함 디렉터리
`C:\opencv\build\include`

2. 링커 → 추가 라이브러리 디렉터리
`C:\opencv\build\x64\vc16\lib`

3. 링커 → 입력 → 추가 종속성
`opencv_world480d.lib`
※ 릴리즈 모드라면 `d` 없는 버전 사용

> 버전 번호(480)는 설치한 OpenCV 버전에 맞게 변경

#### DLL 경로 설정 (실행 오류 방지)
방법 1
아래 경로를 **환경 변수 Path**에 추가
`C:\opencv\build\x64\vc16\bin`

방법 2
`.exe` 파일이 있는 폴더에
`opencv_world480d.dll` 직접 복사


#### 이미지 불러오기

#### 마우스 이벤트

#### 키보드 이벤트

#### 영상 불러오기

#### Mat 클래스
`CV_8UC3`
`8` : 비트수
`U` : 부호 여부
`C3` : 채널의 개수

`zeros()`
`ones()`
`eye()`

`randu(Mat, Scalar::all(0), Scalar::all(255))`

`A.clone()` : Deep Copy


`Rect`







