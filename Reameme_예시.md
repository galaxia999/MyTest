![Image Description](https://www.rsautomation.co.kr/ko/img/logo.png)

1. 순서 있는 목록 1  
	1.1 목록 1.1  
	- 1.1.1 목록1.1.1  
	1.2 목록 1.2  
2. 순서 있는 목록 2

-------------------------------------------------------------------------------
### V1.00.99.52 20240419 자동위치모드 개선, 홈밍 가감속 개선(DCT KNS대응)
1) 자동위치모드 기능 동작 수정 요청  
	위치완료 상태에서 자동위치모드로 변경시 파렛이 이동하지 않는 현상으로 후행 파렛 충돌현살 발생  
	- 원인: 위치완료 상태(13)에서 자동위치모드 시작(MOSI.8+GO)명령을 전달하면 바로 자동위치모드의 위치완료(50-51-52-53)로 진행함.  
	T13_50   위치완료(13)->자동위치모드 준비(50), STATUS 13은 서보온+GO 위치완료 상태이므로 T13_53으로 자동위치모드 위치정지 상태로 정상동작함.  
	- 동작 개선:  
	자동위치모드가 켜져있다면 항상 속도이동하여 현재 파렛을 배출함.
	MOSI.8=1인 경우 MOSI.1(Vel/Pos)에 상관없이 무조건 현재 파렛을 속도이동하여 배출하고 open시 LMS_STATUS 50으로 진행함.
	- 동작 수정(3가지 CASE):  
	VP13_01 위치완료(13)->속도모드초기(01)->자동위치모드 준비(50)
	VP10_01 위치준비(10)->속도모드초기(01)->자동위치모드 준비(50)
	VP02_50 속도이동(02)->자동위치준비(50)
2) Homing Mode 수정  
	홈밍트리거 이후 Offset값에 따라 속도 가감속이 발생하지 않는 현상 수정
	- LMS_STATUS 추가  
	LMS_HOME_OFFSET_INIT  
	LMS_HOME_OFFSET_RUN
		- 홈트리거를 만난 후 Offset위치
		- 5mm (감속거리)이하이면 바로 위치 완료(28)로 진행함.
	> 소스 예시 1

		LMS_IOMapping_P_COM(ax);  
		LMS_IOMapping_V_COM(ax);
	
	> Pseudo Code 예시 2
	
		void PROCESS_Homing_SwitchingRule(uI16 ax);      //20241211 홈밍 스위칭룰
		void PROCESS_SyncMotion_SwitchingRule1(uI16 ax); //20241211 동시이동 스위칭룰1
	
	- CCW 방향(HomingEntry_direction=-1)  
	*조건식1 (5mm이내, A구역) : 위치명령>(트리거 - 5mm)
	*조건식2 (5mm이내, B구역) : 위치명령-(트리거 - 5mm)
 	
3) MISO.3에 자동위치 모드(50~54)일때 상태 표시함.





-------------------------------------------------------------------------------
# 파렛 SPEC
-------------------------------------------------------------------------------
## PalletOffset 600개 정의
1) 순서 있는 목록 1  
	1.1 목록 1.1  
	- 1.1.1 목록1.1.1  
	1.2 목록 1.2  
2. 순서 있는 목록 2

1. 순서 있는 목록 1  
  1)목록 1.1  
  2)목록 1.2
2. 순서 있는 목록 2

1. 명령어 추가 (ERD, EWR)
	1) ERD300* : 300번 배열 저장장소 읽기	
	2) EWR300*  : 전체 저장(save), 플래쉬에 저장하기
  3) EWR300$  : 전체 로딩(load), 플래쉬에 저장된 값 읽기
  4) EWR300!  : default값으로 램저장장소 설정
  4) EWR300@  : all 0's, 테이블 0으로 초기화 하기
  5) EWR3004-1234  : 300번 배열, index 4번 위치에 -1234 값 쓰기
     EWR 한번에 여러개 쓰기하는 기능 추가
     EWR300% V1 & V2 & V3 & V4 & V5 & V6
  6) ERD3004  : 300번 배열, index 4번 위치 값 읽기.  
  
ID 입력받아서 오프셋을 출력하는 기능
   :300번지에 uuid 12345678 쓰기
   #EWR300%12345678&-22222222&-33333333&-44444444&-55555555&-666666666
   :uuid 12345678로 등록된 배열과 오프셋값 가져오기
   #ERD!12345678
   응답: [uuid][배열][v1][v2][v3][v4][v5][v6]
   $ERD12345678&300&12345678&-22222222&-33333333&-44444444&-55555555&-666666666
   # 쓰기 예시
   파렛1 설정 (Table 1, ID: 1111, CW offset:100, CCW_offset:-200)
   파렛2 설정 (Table 2, ID: 2222, CW offset:300, CCW_offset:-500)
   #EWR001%1111&0&100&-200&0&0
   #EWR002%2222&0&300&-500&0&0
   
   


-------------------------------------------------------------------------------
# LMMT 입출력 신호 SPEC
-------------------------------------------------------------------------------
1.물리신호 출력기능 추가: TG_ON,SRDY, P_COM, V_COM  
   LMS_IOMapping_TG_ON(ax); //MISO.1
   LMS_IOMapping_SRDY(ax); //MISO.2
   LMS_IOMapping_P_COM(ax);
   LMS_IOMapping_V_COM(ax);
   LMS_IOMapping_NEAR_AND_PHS(ax); //MISO.8 ~ MISO.13 and MISO.14
   LMS_IOMapping_SALM();


&nbsp;공백 1개 들여쓰기  
&ensp;공백 2개 들여쓰기  
&emsp;공백 4개 들여쓰기
	dfdf
	

1.물리신호 출력기능 추가  
> 1. TG_ON,SRDY, P_COM, V_COM   
	blockQ사용
>>	LMS_IOMapping_TG_ON(ax); //MISO.1  
>>	LMS_IOMapping_SRDY(ax); //MISO.2

> 2.물리신호 예시
>> TG_ON,SRDY, P_COM, V_COM   
	blockQ사용
	LMS_IOMapping_TG_ON(ax); //MISO.1  
	LMS_IOMapping_SRDY(ax); //MISO.2
	
&emsp;
	LMS_IOMapping_P_COM(ax);  


1. 순서 있는 목록 1  
  1.1 목록 1.1  
  1.2 목록 1.2  
2. 순서 있는 목록 2

1. 순서 있는 목록 1  
  1.1 목록 1.1  
  1.2 목록 1.2  
2. 순서 있는 목록 2

	
	LMS_IOMapping_V_COM(ax);  
	LMS_IOMapping_NEAR_AND_PHS(ax); //MISO.8 ~ MISO.13 and MISO.14
	LMS_IOMapping_SALM();

	LMS_IOMapping_P_COM(ax);
	LMS_IOMapping_SALM();



> 2. 속도 신호 예시  
	LMS_IOMapping_P_COM(ax);
	LMS_IOMapping_V_COM(ax);
	LMS_IOMapping_NEAR_AND_PHS(ax); //MISO.8 ~ MISO.13 and MISO.14
	LMS_IOMapping_SALM();
	
	
	
	https://www.rsautomation.co.kr/ko/img/logo.png
![Image Description](https://www.rsautomation.co.kr/ko/img/logo.png)

### 기본 1
Click [here] (https://www.rsautomation.co.kr/ko/img/logo.png)  

### 기본 2
갤럭시의 블로그 [here](https://www.rsautomation.co.kr/ko/img/logo.png)  

### 기본 3
갤럭시의 블로그 [here]  
[here](https://www.rsautomation.co.kr/ko/img/logo.png)  

### 기본 4
title 옵션사용시 커서를 링크 위로 위치하면, title이 노출된다.  
![Image Description][bh]
[bh] : https://www.rsautomation.co.kr/ko/img/logo.png "Click Here~~"

### 기본 5
![Image Description](https://www.rsautomation.co.kr/ko/img/logo.png)

