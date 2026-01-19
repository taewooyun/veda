### 이동 생성자
2011 ~ : modern C++

copy constructor : 복사 생성자
move constructor : 이동 생성자

`Mat m1 = m2`
`Mat m1(m2)`
`Mat m1{m2}`

### Functor
```cpp
A = imread("d:/lenna.jpg", IMREAD_COLOR);
Rectr(10, 10, 100, 100);
Mat D = A(r); // functor
```

###
`ex 3.14` ~




### 화소 처리
- point proccessing

`at()` : 픽셀 하나를 처리할 수 있다
`saturate_cast<>` : 오버플로우 방지

at보다 배열 참조가 더 성능이 우수

oimage

`convertTo()` : 


밝기 및 콘트라스트 조정
색상 교정
색상 교환

#### 공간 필터링
- filtering


#### LUT 사용
- look up table

#### 감마 보정

#### 