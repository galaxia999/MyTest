
Known issues)
1)모니터링 변수
2)모터데이터
3)PalletOffset 600개 정의 : 명령어 추가 (ERD, EWR)
     1) ERD300*  : 300번 배열 저장장소 읽기
        *ID 12345678로 찾는 기능
        #ERD!12345678
     2) EWR300*  : 전체 저장(save)
     3) EWR300$  : 전체 로딩(load)
     4) EWR300!  : default값으로 램저장장소 설정
     4) EWR300@  : all 0's
     5) EWR3004-1234  : 300번 배열, index 4번 위치에 -1234 값 쓰기
     6) ERD3004  : 300번 배열, index 4번 위치 값 읽기.
     7) EWR300% V1 & V2 & V3 & V4 & V5 & V6 : 레코드 한번에 쓰기
        EWR300%11&22&33&44&55&66 : 300번 배열에 6개의 값을 연속적으로 쓰기
        EWR000%11&222&333&444&555&666 : 0번 배열=0x4001 연속적으로 쓰기

4)자동위치(AutoPos) 전환 기능

5) LMS_TorqueLimit_readyFlag(토크제한 플래그) 동작 정의
	0x2A98	152 LMS_Torque Limit Ready
	OFF 조건:
		전원 부팅시 OFF 상태
   	코일위에서 파워싸이클시 OFF
   	홈밍 동작시에는 항상 OFF (위치값이 수정되기 때문에 상위에서 알고 있는 위치와 다르기 때문에 OFF됨)
	ON 조건:
   	코일밖에 있으면 항상 ON
   *특수조건
   	알람 발생후 알람해제되는 과정에서 다음의 동작을 따른다.
   	ft-1.52 D0 Incremental Pos Latch Enable 설정시 그대로 유지
   	ft-1.52 D0 Incremental Pos Latch disable 설정시 0으로 초기화됨.
   	#위치값 처리
   		-알람 리셋할때 readyflag=0 이면 pos_fbk=0
         -readyflag=1 이면 pos_fbk=이전값 유지(ft-1.52 D0 기능 사용시)
   	#알람reset 발생시 알람 관련 플래그는 동작.
   		flag.reset 실행시 flag_encoder_reset는 1로 설정
   		인터럽트CC에서 엔코더리셋 명령(ID 7)을 전송하고 현재 위치값을 별로 래치변수(pos_fbk_latch)로 저장. flag_encoder_reset는 2가 됨.
			flag_encoder_reset=2일 때  파렛(Valid_Direct)이 있으면 파렛감지플래그(IsDirectON_alramreset) 1로 설정함.
											파렛이 없다면   파렛감지플래그(IsDirectON_alramreset) 0으로 설정함.
			LMS Valid 관련변수 모두 초기화, LMS_Torque Limit ReadyFlag 0으로 초기화함.
			flag_encoder_reset는 0으로 초기화함.

		#파렛감지플래그(IsDirectON_alramreset) ON 되었을 경우 (즉 알람리셋이 파렛이 코일에 있었다면)
			1)ft-1.52 D0 Enable시
				래치된 위치값 변수(pos_fbk_latch)를 현재 위치값로 업데이트함.
				엔코더의 이전값을 현재 싱글턴과 일치, 증분값을 0으로 처리함. 
				LMS_TorqueLimit_readyFlag는 그대로 유지함.
					즉 알람 발생전에 1이면 1로 유지(현재 위치 유효)
					즉 알람 발생전에 0이면 0로 유지(현재 위치 무효)
			2)ft-1.52 D0 Disable시
				LMS_TorqueLimit_readyFlag 0 초기화(현재 위치 무효)
				현재 위치값는 싱글턴으로 리셋됨.

   		-latch_pos_flag==0이고 캐리어open일때 1 (ready)


6) LMS용 부하율 시간 설정 기능
  -전류값으로 부하율 계산하는 방법
	RSware Scope의 전류피드백 단위는 암페어임
	Torque Actual Value, 0x6077 값은 0.001 [Amps] 단위
	Motor 정격전류 2.4A 일때 0.074mA 값은  0.074*100/2.4 = 3.08 %

  -평균 부하율에 대한 설정 시간 동안의 이동 평균값을 계산함.
  -내부 버퍼크기 : 99999+1 로 설정
  -2ms 인터럽트에서 계산함.

   평균부하율 계산
  - ft4.23 Current Feedbvack Squared Integral Interval, 단위 [ms], LMMT special
  - MDM 44 0x2A2C Current RMS 		(서보 기존 기능임) 측정시간 설정시 동작함, 0x2417, 10(default), 0~60000 [ms]
  - MDM 45 0x2A2D Current RMS Max	0x2A2C의 최대값

  ----------- DCT special ------------  
   이동 평균부하율 계산
  -LMS 이동평균 부하율 시간 설정 [sec]
  - ft-1.55 0x2137 LMMT_Current Load Moving Avg Time Setting, 0(default), 0~99999[sec]
  - MDM 44 0x2A96	LMS Motor Utilization(RMS) 			: Valid 센서 ON동안의 Current Load Factor(RMS) 제공 (LMS only)
  																		  (0x2A2C와 동일함, 토크 순시값)
  - MDM 45 0x2A97	LMS Motor Moving Utilization(RMS)	: Valid 센서 ON동안의 Current Load Factor(RMS)의 이동 평균값 제공 (LMS only)
  																		  (0x2137 설정시 이동평균값 표시)
--------------------------------------------
SPEC
--------------------------------------------
1) 전류과부하 발생 조건
  두개의 코일위에 정지시 모터에서 소음발생하며 E.022 연속과전류 알람발생
  조건: 모든 구간에서 110% 2초 유지 or
        파렛이 없는 상태에서 70% 30초 유지시 발생함
2) 과속도 알람 E.018 발생조건 : 100ms 동안 설정속도값 이상일때 발생함.
3)엔코더 프로토콜 간단 요약
   // 엔코더 필드값
   VALID                      DF3.7, DF7.7
   Position offset            DF4, DF5
   RFID                       DF6
   Extended Proximity Sensor  DF7
4)엔코더 알람비트
   if (Reg5810.RxDATA5 & 0x1F00) { // D4:TIMOT, D3:SFOME, D2:FOME, D1:CRCE, D0:CONTE
         TIMOT(time Out Error):
         SFOME(Short Form Error):
         FOME(From Error):
         CRCE(CRC error):
         CONTE(control field error):
5) 모니터링 변수 추가
   0x2A96	150 LMMT_Motor Utilization(RMS)			,0x2A2C Current Load Factor Feedback(RMS)와 같은값임.
   0x2A97	151 LMMT_Motor Moving Utilization(RMS)	,0x2A96의 ft-1.55시간동안의 이동평균값  
   0x2A98	152 LMMT_Torque Limit Ready
   0x2A99	153 LMMT_Enc Start Value, Valid ON되는 시점의 엔코더값 모니터링 추가
   0x2A9A   154 LMMT_Extended Proximity Sensor
   
   0x2A9B 155	PalletOffsetMonitor.pallet_id
   0x2A9C 156	PalletOffsetMonitor.pallet_opt
   0x2A9D 157	PalletOffsetMonitor.EA_ofs
   0x2A9E 158	PalletOffsetMonitor.EB_ofs
   0x2A9F 159	PalletOffsetMonitor.EA_ofs_dft
   0x2AA0 160	PalletOffsetMonitor.EB_ofs_dft

   0x2AA1 161 SyncEA_Status
   0x2AA2 162 SyncEB_Status
   0x2AA3 163 SyncEA_StartValue
   0x2AA4 164 SyncEB_StartValue
   0x2AA5 165 EA_TRIGGER
          166 EB_TRIGGER
          	  EA_NEW_PALLET
          	  EB_NEW_PALLET
   0x2AA7 167 SyncEA_PalletInCnt
   0x2AA8 168 SyncEB_PalletInCnt
   0x2AA9 169 LMS_ID3_Enc
   0x2AAA 170 LMS_ID5_Enc
   0x2AAB 171 LMS_Master_Enc
   0x2AAC 172 LMS_Slave_Enc
   0x2AAD 173 LMMT_MOVE MOSI.2			상위 기동명령에 대한 모션동작을 정확히 분석하기 위해 추가함.(MOSI.2)
	0x2AAE 174 SyncEA_Velocity Feedback 동시모드에서 EA 속도피드백
	0x2AAF 175 SyncEB_Velocity Feedback 동시모드에서 EB 속도피드백
	0x2AB0 176 Sync_Velfdk Timetick		동시모드 속도피드백 시간틱
 
6) 제2위치 선택 기능
	MOSI.13	Position Select	제1/2 위치 설정 (제1위치:0, 제2위치 :1)
	0x213B, ft-1.59, LMMT_Positive 2nd Distance/Position
	0x213C, ft-1.60, LMMT_Negative 2nd Distance/Position

7) 확장 근접센서(DHE) I/O 4점 지원
	-확장센서와 모든 홀신호는 오브젝트의 고정자리에 맵핑되도록 사양 결정 (DCT요청사항) //20230215
   0x2A9A LMMT_Extended Proximity Sensor (하위바이트: Enc A, 상위바이트: Enc B 대응)
	   Encoder DF3/DF7 
	   EncA I/O [DF3 bit 5,6] : 0x2A9A bit 0,1에 대응
	   EncB I/O [DF7 bit 5,6] : 0x2A9A bit 8,9에 대응
   0x2A9A (DF3/DF7 전체를 표시한다)		
   bit#0	DFx.bit0	HR : Digital Hall Sensor Right		A엔코더 HR				A엔코더 HR		
   bit#1	DFx.bit1	HU : Digital Hall Sensor U				A엔코더 HU				A엔코더 HU		
   bit#2	DFx.bit2	HV : Digital Hall Sensor V				A엔코더 HV				A엔코더 HV		
   bit#3	DFx.bit3	HW : Digital Hall Sensor W				A엔코더 HW				A엔코더 HW		
   bit#4	DFx.bit4	HL : Digital Hall Sensor Left			엔코더 HL				A엔코더 HL		
   bit#5	DFx.bit5	Master side Proximity Sensor 1		A엔코더 근접센서 신호 #1	A엔코더 근접센서 신호 #1		
   bit#6	DFx.bit6	Master side Proximity Sensor 2		A엔코더 근접센서 신호 #2	A엔코더 근접센서 신호 #2		
   bit#7	DFx.bit7	Encoder A Valid							A엔코더 Valid			A엔코더 Valid	
   	
   bit#8	DFx.bit0	HR : Digital Hall Sensor Right		B엔코더 HR				B엔코더 HR		
   bit#9	DFx.bit1	HU : Digital Hall Sensor U				B엔코더 HU				B엔코더 HU		
   bit#10	DFx.bit2	HV : Digital Hall Sensor V			B엔코더 HV				B엔코더 HV		
   bit#11	DFx.bit3	HW : Digital Hall Sensor W			B엔코더 HW				B엔코더 HW		
   bit#12	DFx.bit4	HL : Digital Hall Sensor Left		B엔코더 HL				B엔코더 HL		
   bit#13	DFx.bit5	Slave Proximity Sensor 1			B엔코더 근접센서 신호 #1	B엔코더 근접센서 신호 #1		
   bit#14	DFx.bit6	Slave Proximity Sensor 2			B엔코더 근접센서 신호 #2	B엔코더 근접센서 신호 #2		
   bit#15	DFx.bit7	Encoder B Valid						B엔코더 Valid			B엔코더 Valid		

9) A/B 개별 불감대 영역 설정기능(1.50은 테스트용으로 추가 변경예정)
  0x2131	LMS_EncA Base Deadzone Offset	LMS_엔코더A 불감대 위치 오프셋 Ft-1.49		1.20.10.22버전 이상
  0x2132	LMS_EncB Base Deadzone Offset	LMS_엔코더B 불감대 위치 오프셋 Ft-1.50		1.20.10.22버전 이상

10) 2pallet 알람 발생 조건
	*2 Pallet 알람 발생 조건(E.112 ESTOP)
	*ft-1.41 D0 enable 되고 서보온인 경우 발생			
	용량		센서값(Proxi)	
	400W 이하		01_1011	(0x1B)
	400W 이상		11_0111	(0x37)
	400W 이상		11_1011	(0x3B)
	400W 이상		33_0011	(0x33)
	
11) 파라메터 저장명령(0x1010:01/03/04)시 파렛오프셋 자동저장 기능 동작사양(DCT요청사항, 20231005) 
	0x1010:01 "save" 명령시(Store all Parameters)           flgCallBack = 0x80101001, FlashBackup 과 SavePalletOffset 모두 호출함. 
   0x1010:03 "save" 명령시(Store cia402 Parameters)        flgCallBack = 0x80101003, SavePalletOffset 호출
   0x1010:04 "save" 명령시 (Store CSD7 specific Parameters) flgCallBack = 0x80101004, FlashBackup 호출

12) 무버 ID 동작 사양(10.61기준), 남기혁책임 요청사항
	-목적: RFID는 무버마다 가지는 엔코더의 오프셋을 관리하기 위해 도입됨
	-사용조건
      Ft-1.22=1: 인코더 오프셋값 적용 유무 (default: 적용 안함, 1: 인코더 통신으로부터 적용)
      Ft-1.52 D1=Enable (0: 서보 저장 파렛오프셋값 적용 안함, 1: 서보 저장 파렛오프셋값 적용함)
		무버 ID 값은 엔코더 통신 패킷상 DF6(1byte)으로 수신합니다.
	-검색 트리거조건
		파렛이 외부에서 진입시 엔코더가 동작상태가 될때 ID값을 자동으로 찾습니다.
		이는 전원이 off/on되고 파렛이 존재하고 엔코더가 동작상태가 될때 자동 검색하는 것과 동일한 메카니즘으로 동작합니다.
   -SET(1) 조건 
	무버 ID 검색 성공시 ID와 오프셋 할당되고 이 값은 파렛이 배출되기 전까지 유지됩니다.
      MISO.14(MID_STS)에 1(set)로 비트 반영됨
      0x4000:02, Option Low Word : 테이블 번호, 0x4003이면 마지막 숫자 3을 뜻함
      0x4000:02, Option High Word : RFID 검색 결과 성공여부, 성공=1, 실패=0
	CLEAR(0:) 조건
		검색 실패하는 경우(찾는 무버 ID가 없음) MISO.14=0(clear), 0x4000 값들은 모두 0으로 초기화됩니다.
		엔코더의 Valid 신호가 없을때 모두 0으로 초기화됩니다.
		RFID값이 0으로 수신되는 경우 파렛오프셋이 미적용되도록 모두 0으로 초기화됩니다.(10.61버전 추가) 

	*알람발생시 위치유지 기능이 있는 특이한 상황에서 파렛오프셋값 동작 정의
		서보 일반 알람발생시 : 현재 위치유지함. 파렛오프셋값??  
		서보 엔코더관련 알람발생시 : 현재 위치 유지하지 않음 파렛오프셋값?? 
	
	*특별히 ID값 처리하는 디바이스가 없는 경우 ID를 수동으로 설정하여 오프셋을 적용할 수 있는 기능을 제공합니다. 
		ft-1.50 Pallet ID에 ID값을 0이 아닌 값으로 설정시 ft-1.22보다 우선하여 동작합니다.
		이 기능은 LMS제어 운영프로그램상에서 구동될 수 있으며 통신지연이 발생할 수 있습니다. 

12) 속도 감속시점 조정 기능
	ft-1.44 (0:disable, 0 아닌 위치값을 입력시 등속에서 감속되는 시점을 임의 조정 가능)
	예시)동작사양
		A	목표위치	300000
		B	설정값	50000						
		C=(A-B)	남은거리	250000							
										
		*현재 속도 0.8m/s로 200ms 감속시간으로 250000 위치에 정지하는 감속기울기 성생									
			속도			800	mm						
			감속시간		0.2	초						
			이동거리		80	mm,1mm = 1000 pls				
			필요한 위치값	80000	pls						
									
	프로파일 생성기는 목표위치에서 위치설정값 50000 pls 앞의 가상의 위치(250000)를 최종 target으로 설정한다.															
	감속에 필요한 위치값은 80000 pls이므로 남은 거리 170000 위치부터 감속을 시작한다.								
	1-1)LMS_making_profile에서 남은거리-위치조정값을 빼준 조금 짧은 거리만큼 이동하기 위한 기대속도를 계산한다.
	1-2)남은 거리가 위치조정값 이상인 경우(남은거리 > 위치조정값) 프로파일  계속 생성

13) 동시이동 관련 변수와 신호 정의
	1)동시이동 파렛 카운팅 설정 변수, ft-2.47 [D0..D3] Pallet Out Counter for SyncMotion
	현재 코일위에 새로 진입하는 파렛에 대해서 속도이동으로 배출해야 하는 파렛의 카운터(갯수)를 입력한다.
	-진입하는 파렛을 위치정지시키는 경우 0 으로 입력한다.
	-첫번째 진입 파렛을 속도이동으로 배출하고 두번째 파렛을 위치정지시키는 경우 1 으로 입력한다.
	-첫번째,두번째 진입 파렛을 속도이동으로 배출하고 세번째 파렛을 위치정지시키는 경우 2 으로 입력한다.
	D7          					D8            내부변수
	----------------------------------------------------------------------------------------------
	ft-2.47 D0, 0x222F:01, 		ft-3.40	SyncEA_palletOutCnt_CW  CW방향으로 새로운 파렛 진입시 배출해야 할 파렛의 갯수
	ft-2.47 D1, 0x222F:02,		ft-3.41	SyncEB_palletOutCnt_CCW CCW방향으로 새로운 파렛 진입시 배출해야 할 파렛의 갯수
	
	오브젝트(D7)
	0x222F	Pallet Out Counter for SyncMotion	동시이동 파렛 배출 개수
		동시이동 파렛배출 카운터   CW		1	Pallet Out Counter for CW
		동시이동 파렛배출 카운터   CCW	2	Pallet Out Counter for CCW
		동시이동 파렛배출 카운터2 CW		3	Pallet Out Counter2 for CW
		동시이동 파렛배출 카운터2 CCW	4	Pallet Out Counter2 for CCW

	2)새로 진입하는 파렛 카운팅 변수
	현재 모듈위에서 속도/위치 모드를 결정하는데 사용된다. 
	0x2AA7 167 SyncEA_PalletInCnt 동시모드에서 EA 홀센서의 Rising(0->1) 트리거 발생시 1씩 증가하는 증분형 카운터
	0x2AA8 168 SyncEB_PalletInCnt 동시모드에서 EB 홀센서의 Rising(0->1) 트리거 발생시 1씩 증가하는 증분형 카운터
	
	3)엔코더 홀센서의 트리거 신호
	*트리거 신호는 Valid 신호가 rising에서 ON되고 EA/EB VALID 값이 변경되면 OFF된다.
		Sync_Trigger_EA 신호 : 새 파렛이 EA 진입시 ON 되고 VALID 3 되면 OFF됨
		Sync_Trigger_EB 신호 : 새 파렛이 EB 진입시 ON 되고 VALID 3 되면 OFF됨

	*새파렛 진입되었음을 확인하는 _NEW_PALLET신호는 Valid 0 이되면 OFF된다. 그렇지 않으면 계속 유지한다.
		SYNC_EA_NEW_PALLET : 엔코더A Valid ON될때 ON, Open시 OFF된다.
		SYNC_EB_NEW_PALLET : 엔코더B Valid ON될때 ON, Open시 OFF된다.
		
	0x2AA5:01 165 Sync_TRIGGER_EA : EA Valid OFF->ON 시점에 SET, Valid 상태가 바뀔때 초기화(OFF)된다.
   0x2AA5:02 166 Sync_TRIGGER_EB : EB Valid OFF->ON 시점에 SET, Valid 상태가 바뀔때 초기화(OFF)된다.
	0x2AA5:03 163 SYNC_EA_NEW_PALLET : EA Valid OFF->ON 시점에 SET, Valid 상태가 0아닌 값으로 바뀌어도 초기화되지 않는다.
   0x2AA5:04 164 SYNC_EA_NEW_PALLET : EB Valid OFF->ON 시점에 SET, Valid 상태가 0아닌 값으로 바뀌어도 초기화되지 않는다.
		ON 조건: 
		  1)EA or EB Valid 신호가 ON 될 때
		  2)위치모드에서 위치정지 후 위치티칭값 변경시 ON, 항상 위치이동이 가능하도록 하기 위해서.
       	    등속구간에 감속정지하기 위한 판단 조건으로 사용함.
     	    (중요) MOSI.9 계속 ON인 동안 위치/스텝이동 가능하도록 처리하리하기 위한 용도.
		OFF조건: 
		  1)동시모드 진입시 항상 OFF시킴
		  2)동시모드 탈출시 항상 OFF시킴(미구현상태)

	4)속도피드백 유지시간 기능
		D7                D8            내부변수
 	-----------------------------------------------------------------------------------------------------------
	ft-2.48, 0x2230,  ft-2.48	SyncEA Velocity Feedback Keeping Time for EB Trigger	EB 트리거시 EA 속도피드백 유지시간
	ft-2.49, 0x2231,  ft-2.49	SyncEB Velocity Feedback Keeping Time for EA Trigger	EA 트리거시 EB 속도피드백 유지시간
	ft-2.50, 0x2232,  ft-2.50  RFID Offset Apply Delay Time
	모니터 변수	
	0x2AAE 174 SyncEA_Velocity Feedback 동시모드에서 EA 속도피드백
	0x2AAF 175 SyncEB_Velocity Feedback 동시모드에서 EB 속도피드백
	0x2AB0 176 Sync_Velfdk Timetick		동시모드 속도피드백 시간틱

	5) Enc_changing 변수: 엔코더 ID 변경 순간을 알려주는 플래그
		2: ID5로 변경 예정
		1: ID3로 변경 예정
		0: 변경완료된 상태
		             
	6) SM_PalletOpenedFlag : 현재 파렛이 배출되어 오픈상태가 될때 ON됨. (TBD)

	
--------------------------------------------
History
--------------------------------------------
1) DCT 제공 
	V 1.20.10.04 (20221012)

--------------------------------------------
	FW 사양 & 개선 검토할 사항들 취합
--------------------------------------------
1)엔코더 양간오차 검증
	-엔코더 전환시점에 발생하는 오프셋만큼을 위치편차로 계산한다.
2)모터 정지시 떨림으로 시작하는 속도명령이 0에서 출발하지 않는 현상 개선 
	*ft-2.44 (Encoder Delta Threshold for Motor Stop Detection)
		정지상태에서 출발시 속도프로파일을 계산할 때 시작 명령값을 속도피드백으로 판단합니다. 
		설정값 이하로 흔들리는 경우 정지상태로 판단하고 속도명령의 시작값을 0 으로 시작하는 기능입니다.
3)MOSI/MISO 신호 동작 상태
	MISO.7 PCOM	Trigger(LMS Trigger Flag)신호가 ON이고 위치에러가 ft-3.18보다 같거나 작을때 ON 됨
          (Trigger신호는 속도/위치일때 항상 ON이고 홈밍시작할때 OFF되고 홀센서 트리거될때 ON 된다)
	MISO.1 TGON ON조건: 속도명령이 ft-2.19보다 큰 경우
			OFF조건: 다음의 조건들 중 하나라도 만족하는 경우임.
	       1)PCOM ON일때
			 2)속도준비단계(0)
			 3)속도정지단계(3)이고 속도명령이 0인 경우
			 4)위치준비단계(10)
			 5)홈밍준비단계(20)
4)부가 기능 개발 내용들
	1)엔코더1회전에 대한 IO펄스 출력기능_20231108
		-Z상 출력 기능 추가, LMS_EtherCAT_Z, Z상 출력 IO 맵핑
		-Digital Output #1 출력후 스코프로 파형 측정, (admin 777입력시 동작)
	  		ft-0.28 D2 LMS_Homing Status가 맵핑되어 있음.

5) XML 오브젝트 작업시 주의사항
	#PDO 등록하는 방법
	1)등록하고자 하는 오브젝트의 Entry Description에 (OBJACCESS_RXPDOMAPPING|OBJACCESS_TXPDOMAPPING) 속성을 등록한다.
	2)XML에는  오브젝트 Flags속성에 <PdoMapping>T</PdoMapping> 를 추가한다. 
	3)오브젝트 리스트에 추가시 오브젝트 구조체의 포인터변수 pVarPtr가 NULL이 되지 않도록 변수의 어드레스를 등록한다.
		NULL로 등록된 경우는 주로 06000번대이고 이 경우 CoE 초기화시 어드레스를 등록해 주어야 한다
	4)오브젝트 구조체의 pVarPtr에 변수 어드레드스 등록된 경우 READ/WRITE함수는 호출되지 않으므로  RD_0x2AXX 위치에 NULL로 써도 된다.
	5)서보인덱스를 가지는 오브젝트는 sDrive_2A3D와 같이 TOBJBITIDX형의 자료구조를 사용해야 한다.
	  할당된 변수 sDrive_2A3D에 실제로 사용되는 값들을 업데이트한다.
	  
-----------------------------------------------------------------------------------------------------
*신관 longrun TEST 이력
   //20231115
	1)AX4 아침에 처음 전원 넣은 상태에서 EA에 아무 신호도 발생하지 않음. 서보알람 없음, EB는 정상
	   ->서보 전원 off/on 이후 정상동작함 (최철승 수석과 함께 증상 확인함)
	2)AX2에 파렛1이 있는 상태에서 E.107발생, AX3는 파렛없고 E.022 발생한 상태로 멈춰진 상태, AX1은 서보온+파렛2가 있는 상태임.
		#define ErrCode_SerialCommErr   0x60  // E.107 SERCE
		Rsware 연결하여 OSC 파형 Run Trigger모드로 동작중인 상태였음.
		E.107은 rs_ware_protocol함수에서 체크섬 오류일때 발생하는 건데 RSware정상 연결되어 있고 Awaiting Triger로 되어 있음.
		롱런 1시간 반정도 진행하다가 발생한 걸로 나옴.
	3) AX1 정지상태에서 inposion 신호 채털링
		3.18값 10->100변경
	//20231120 59버전 롱런 진행중

-----------------------------------------------------------------------------------------------------
동시이동 구현하면서 위치틀어짐 발생된 이력 내용
1)위치 틀어짐
	47 변경점 검토: 동시구동 모드에서 한코일 위에 2pallet진입시 pos_fbk값이 잘못 적용되던 버그 수정
	   - 정방향 구동시 한 코일 위에 두개의 pallet가 진입시 TxDATA는 5 -> 3으로 스위칭됨. 이때  pos_fbk에 LMMT_ID3_Enc값을 넣어주도록 수정함.(역방향 구동시(TxDATA 5 -> 3)일 때도 동일한 방법으로 수정함)
	   -  TxDATA는 5 -> 3 또는 3 -> 5로 Enc 스위칭 시 LMMT_pos_fdk_del값이 잘못 계산되던 버그 수정.
	
	1-1) TxData 변경시 follow position 업데이트 위치 중복 여부?
		pos_fbk = LMMT_ID5_Enc;//역방향 2파렛 진입시 위치값 계산
	1-2) old 변수 동작 
		sI32 LMMT_pos_fdk_del_old;  //20231023
   	sI32 LMMT_Enc_data_old; //20210728
   1-3) pos_fbk에 LMMT_ID5_Enc값을 넣을때 inverted 사용시 inverted 해 주는가?
   	pos_fbk = (FullTurn +1) - LMMT_ID5_Enc;//역방향 2파렛 진입시 위치값 계산 ???
	1-4)
		
-----------------------------------------------------------------------------------------------------
## 동시이동 FW 개발 난제 모음 (contd.)
	엔코더 스위칭룰 개발: 51버전에서 완료함.
		1) 1pallet만 이동시 스위칭룰에 따른 전기각 보상로직 동작
      2) 2pallet만 일때는 스위칭룰에 따른 진입시 전기각 보상로직 동작
		난제: 2pallet 이동시 CASE 1)의 로직을 적용할 수 없음 (CASE 2와 기능 충돌)
   	  	  CASE 2) 기반으로 동작, A->AB 단독구간 진행시 보정한 오프셋값이 비정상 적용됨.
     
	DIR_VALUE 변경시점 -> 58버전에서 개선함.
	동시이동시 엔코더 스위칭룰의 신호 정의
		Enco_2Poffset_Diff, Enco_2Poffset_Diff_old
		Enco_2Poffset, Enco_2Poffset_old

	동시이동 모드 진입 조건 검증할것들.
		-복합모션(위치.속도.자동위치.동시이동)
		-소보온+GO ON 일때  운전모드 변경시 이상없는지 검증
		VP모드       SM모드
		----------------------------------------------------------------------------------------------------------
		TestCASE        설명                                       동작결과                   세부동작 설명
		----------------------------------------------------------------------------------------------------------
      T0_50    T0_60  속도준비(0)->동시구동대기(60)  SM 진입시 기동안됨      STATE 0은 서보오프 + 속도 모드 상태이므로 T0_60 으로 정상동작함.
      T3_50    T3_60  위치완료(3)->동시구동대기(60)  SM 진입시 기동안됨      STATE 3은 서보온 + 속도 정지 + GO OFF 상태이므로 T3_60 으로 정상동작함.
      T10_50   T10_60 위치준비(10)->동시구동대기(60) SM 진입시 기동안됨      STATE 10은 서보오프 + 위치 모드 상태이므로 T10_60 으로 정상동작함.
      T13_50   T13_63 위치완료(10)->동시구동대기(63) SM 진입시 기동안됨      STATE 13은 서보온 + 위치완료 상태이므로 T13_63으로 동시모드 위치정지상태로 정상동작함.
      T14_50   T14_60 위치정지(14)->동시구동대기(60) SM 진입시 기동안됨      STATE 14는 서보온 + 위치정지 + GO OFF 상태이므로 T3_60 으로 정상동작함.

	응용 모션
	[1.2]->WW 구동시 1축의 속도값을 작은 속도->정상속도 이상으로 바꾸면 느리게 시작해서 빨리 따라가는 모션을 만들수 있음.(Hepco 모션과 유사하게) 

	## NRSA 	기본모듈
		자석 정보, (기구도면상값, 실측), 지그 제작 가격, 
		에어갭
		TW08감도
		모터 RLC값 믿을만한가

-----------------------------------------------------------------------------------------------------	
to do list.
1) 1ms 인터럽트 수정할것
//20190116 pp기능의 스케쥴링을 위해서 타이머 이벤트 삭제
**타이머 우선순쉬 개선 필요
	#56버전에서 타이머우선순위 변경 후 롱런시 E.203발생하는 현상 디비깅할 신호
         Sync0WdCounter
         Sync0WdValue
         dwWD0_MIssCounter
         bDcRunning
         bPllRunning
         
      Cnt2mRoutine++;
  if( Cnt2mRoutine == CYCLE_05T || Cnt2mRoutine == CYCLE_15T ) // step [4], [12]
  {
     ISR = MACRO_INT_TIMER0;
     //Cnt_HWtimer1msFRC++;
  }
  else
  {
     //20190415 adjusting interrupt timing.
     if(Cnt2mRoutine == CYCLE_01T || Cnt2mRoutine == CYCLE_09T) // step [1], [9]
     {
        nRepeatCnt_PP = 0;  // PP모드일때 동기화 플래그
     }

     if( Cnt2mRoutine >= CNT_2MSEC_SC ) // assuming 16.
     {
        Cnt2mRoutine = 0;
        ISR = MACRO_INT_2MS;
     }
  }

#else
   if ((++Cnt2mRoutine) >= CNT_2MSEC_SC) {
      Cnt2mRoutine = 0;
      ISR = MACRO_INT_2MS;
   }
#endif++;

*assert_param사용법 추가
*Alias ID 확장 필요

## 메모리 절약할 수 있는 부분
1) EcatCmd_V, EcatCmd_T;
      case 81:         Ltmp = EncEA.Cnt; break;
      case 82:         Ltmp = EncEA.Cnt_old; break;
      case 83:         Ltmp = test_A; break;
      case 84:         Ltmp = test_B; break;

*우선순위에 따른 Trigger EA/EB 신호 On/Off 동작 유의
	(1)PROCESS_SyncMotion_SwitchingRule1
	(2)LMS_TRIGGER_routine
	(3)PROCESS_SyncMotion_TriggerCondition
	ETC----------------------------------
	(4)Proximity Sensor
	(5)Set_PalletOffset_fromTable
		파렛오프셋 처리 기능

*DCT 현장 이력사항
	20250214 이력 발생: 84버전 
		1)홈밍 기능 수행시 홈밍 후 정지위치가 홈오프셋으로 동작하지 않고 위치값으로 동작하는 현상
		2)위치정지 +GO OFF 상태에서 구동방향 바꿀 때 파렛이 이동하는 현상
 

///////////////////////////////////////////////////////////////////////////////
0) V02.00.00.00 20250404 ECN 빌드용
1) 인터럽트 마진 신호 출력을 위한 TP 설정 추가
	Inner : SET_TP0(0); //20250402
	Outer : SET_TP1(1); //20250402
	2ms   : TP97_SET(); //20250402
	main  : TP89_SET(); //20250402
	*기존에 flash 측정용으로 사용했던 코드 주석 처리함, 20150605	

///////////////////////////////////////////////////////////////////////////////
0) V01.20.99.02 20250331 Oscilloscope관련 수정사항
1) 기존: RSWare OscilloScope Trigger 시 1회 Buffer가 채워진 후 Trigger 가능
       수정: Pretrigger Pecentage만큼의 Data가 버퍼에 채워지면 Tregger가능
2) ReadMe파일 수정(89버전, 90버전 위치바뀜 수정)
 
///////////////////////////////////////////////////////////////////////////////
0) V01.20.99.01 20250326 ECN을 위한 수정사항 추가
  - V01.20.10.90 버전에서 1.20.99.01 버전을 만듬
1) LMMT 기능에서 사용하지 않는 ft-1.19 기본값 원복, 5->20
   - 정지판단을 위한 위치 흔들림 크기 폭 설정 기능은 ft-2.44로 할당됨.
2) LMS 인코더의 홀센서 상태값 모니터링과 오실로스코프에 추가
	LMS Hall Value, MDM 23
3) LMS 제어용 모니터링 변수 맵핑
	MDM 20 속도루프에 실제로 적용되는 속도에러 		 : Analog velocity command voltage
	MDM 21 파렛이동 진입시 실제로 적용되는 속도피드백  : Analog current command voltage

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.90 20250312(20250214수정내용 머지) 위치명령 업데이트 오류로 비정상적인 구동 현상 수정(DCT전달)
1) S커브 실시간 적용 개선하면서 로터리모터 구동안되는 오류 수정
2) 속도가감속과 S커브를 자동위치와 동시이동에 확대 적용
	적용단계: 속도/위치/자동위치/동시이동 초기화 단계(LMS_VEL_INIT/LMS_POS_INIT/LMS_VP_INIT/LMS_SM_INIT)
	호출함수: Update_velocityAccDec(0), ApplicationPara(208, Para[2][8].val);
3) 홈밍 기능 수행시 홈밍 후 정지위치가 홈오프셋으로 동작하지 않고 제1/2위치값으로 동작하는 현상
  -81버전에서 수정한 Update_TargetPositionWithDirection 함수를 LMMT control에서 항상 호출하도록 변경한 부분의 side effect
  -홈밍모드에서는 master position이 홈밍 오프셋값으로 적용되도록 수정함.
4) 위치정지 + GO OFF 상태에서 구동방향(MOSI.3) 바꿀 때 파렛이 이동하는 현상
  -81버전에서 수정한 Update_TargetPositionWithDirection 함수를 LMMT control에서 항상 호출하도록 변경한 부분의 side effect
    원인:
    정지 상태에서 방향을 바꾸면 Update_TargetPositionWithDirection 함수에 의해서 target position이 변경된다.
    이때 LMS_STATUS는 변경되지 않은 상태에서 위치에러(pos_err)만 발생하여 속도명령이 생성되어 파렛이 이동하게 됨.
    수정:
    정지상태에서 방향을 바꾸면 LMS_STATUS(10) 준비상태로 천이함.
    정지상태에서 위치값을 바꾸면 LMS_STATUS(10) 준비상태로 천이하는 것과 동일하게 처리함.
  Update_Direction_And_StatusChanging(ax) 추가하고 Update_TargetPosition_with_PDOupdateTiming 함수와 함께 사용함.
  -적용되는 상태: 위치완료(13/53/63) 상태에서 구동방향 변경시 위치준비(10)
///////////////////////////////////////////////////////////////////////////////

0) V01.20.10.89 20250312 Blackbox기능 추가 외
1) Blackbox기능 추가(SEMES Model FW 기준으로 기능추가함)
  - uI16 IsBBoxSave(void) 함수추가
  - void DelayTimer_Init(Type_DelayTimer *pTimer )함수 추가
  - void DelayTimer_Update(Type_DelayTimer *pTimer )함수 추가
  - void DelayTimer_Start( Type_DelayTimer *pTimer, uI32 SetStatus )함수 추가
  - uI08 DelayTimer_GetStatus( Type_DelayTimer *pTimer )함수 추가
  - 관련 변수 및 구조체 추가
2) Black Box PreTrigger 80%로 고정
3) Black Box Chanel Default값 변경
  - CH A: Current Command
  - CH B: Velocity Command
  - CH C: LMS Digital Input Data
  - LMS Status
4) LMS System이 아닐 때(Rotary Motor구동시) RollandTriggerMode()함수가 125us마다 불려오도록 수정(기존 250us마다 불렸음)
5) Ft-5.14(LMS Motor Overload Detection Level)추가 
  - Parameter Range 0 ~ 100
  - Ft-5.14값이 0이면 기존 검출레벨로 알람 발생.(Overload검출레벨: E.022 -> 정격전류의 110%이상 2초, E.104 -> 정격전류의 70%이상 30초)
  - Ft-5.14값이 0이아닐 경우 기존 검출레벨 * Ft-5.14 * 0.01을 검출레벨로 적용함.
6) Off-Line Autotuning관련 기능 수정
  - 관성비 추정 후 공진 주파수 추정 기능 주석 처리
  - RAM확보를 위해  ResFrqDetection()함수 주석 처리
  
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.88 20250122 동시이동 모드에서 RFID 오프셋 적용 오류 수정 
1) 동시이동 모드에서 RFID 오프셋 적용시 오프셋 검색완료 되지 못하여 오프셋 적용안되는 현상 수정
   -일반모드에서 RFID 오프셋 지연 기능은 정상 동작함.
	-타이머 시간틱 32비트 변수로 확장 (최대 10000ms 입력시 *16하여 160000 입력됨)
   -동시이동 파렛 진입시 진행방향에 따라 트리거 신호를 구분하여 처리함
	  정방향 CW 구동시 EA측 외부 진입 시간 지연틱 1부터 시작.
	  역방향 CCW 구동시 EB측 외부 진입 시간 지연틱 2부터 시작.
   -RFID 오프셋 검색완료되는 시점에 파렛오프셋(Pallet Offset)을 한번 갱신하여 위치명령이 업데이트되도록 한다.
   -RFID 오프셋 지연 시간 설정값은 파렛A의 Trigger EA ON되는 시간과 다음 파렛B의 
 	 Trigger EA ON되는 시간보다 작게 설정해야 한다. (설정 가능한 최대 시간 범위 이내 사용해야 함.)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.87 20250114 위치 정지시 영속도 제어로 정지하는 기능 확대 적용외 2건
1) 서보온상태에서 파렛이 진입할 때  위치 정지 명령이면 프리런되는 현상 개선
   -LMS_POS_READY(10) 상태에서 코드 누락으로 동작하지 않았음.(D7 미동작, D8은 정상 동작)
   -위치(10)/자동위치(50)/동시이동(60) 대기 상태에서 적용됨.
   -서보온상태에서 파렛이 진입할 때  위치 정지 명령이면 프리런되는 현상 개선
   -외부에서 Servo On -> Pos Mode -> 파렛 진입 시 Free Run 되는 현상 수정
	-대기(READY) 상태에서 기동시 속도명령(1/2 Velocity) 변경시 가감속값 업데이트함.
2) 운전모드 우선순위 조정
	동시이동(SM) > 자동위치(VP) > 속도위치(POS)=스텝이동(STEP)=홈밍(HM)
3) HW/FW 비상정지 개선
   -파렛 정지 속도 판단 수식을 수정하여 감속시간이 정확하게 나오도록 개선함.
       속도0 정지 명령 판단 기준: 0.001mm/s 이하
   -적용되는 운전모드 확대
       개선전: 속도/위치/홈밍 모드에서만 비상정지 동작
       개선후: 속도/위치/홈밍/스텝/자동위치/동시이동 등 비상정지 모드외 상시 동작되도록 수정

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.86 20241231 동시이동 위치완료 상태에서 동작을 자동위치모드 사양과 동일한 사양으로 적용함.
1) 위치/자동위치/동시이동 모드의 위치정지 상태에서의 동작 사양
	(1)위치모드 진입시  MOSI.2 OFF시 현재 위치 유지
	(2)자동위치모드 진입시  MOSI.8, MOSI.2 동시 OFF시 현재 위치 유지(53)
	(3)동시이동모드 진입시  MOSI.9, MOSI.2 동시 OFF시 현재 위치 유지(63)
2) 변수 이름 수정
	LMS_palletOutCnt_CW	-> SyncEA_palletOutCnt_CW
	LMS_palletOutCnt_CCW	-> SyncEB_palletOutCnt_CCW
	LMS_TG_mode_flag		-> Sync_POSMODE_flag
3) RFID 통신지연으로 인한 위치 오프셋 업데이트 늦어지는 것을 개선하기 위해 업데이트함수를 속도루프 주기로 갱신하도록 함.
   -DIR(FWD/BWD), RFID 사용시 오프셋 적용 시점에 해당함
	-LMMT control에서 Update_TargetPositionWithDirection(0) 함수 호출
   
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.86 20241226 85버전 Code composer에서 열리지 않는 문제
1) 코드 수정사항 없음 85버전과 동일함. (85버전의 Source폴더를 복사하여 84버전에서 빌드하여 86으로 버전업)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.85 20241226 RFID 오프셋 적용 시점을 지연시키는 기능을 파라미터(ft-2.50)로 추가(DCT서형식 수석 요청)
1) RFID 오프셋 적용 지연 시간 설정 파라미터 추가
   RFID Offset Apply Delay Time							 	0x2232		Ft-2.50 	0x2232	P0-2.50		Y	6018
     기존과 동일하게 사용하기 위해서는 400 [ms] 로 설정함.
2) 기타 수정사항들
     내부 변수/함수 이름 수정
		이 름               D7		            		D8
	LMS_P_SET	-> LMS_VELPOS 					LMS_VELPOS_BUF
	LMMT_MOVE	-> LMMT_MOVE  					LMS_GO_BUF
						EcatLmsFlag.START_IXG 	LMS_Start_switch
	DI_C_DIR		-> LMS_DIR_VALUE				LMS_Dir_value 						
	DI_V_SEL		-> LMSFlag.V_SELECT			LMS_V12_BUF
						 								LMS_Vel_Select_switch
   Set_EncDiffCompensator -> Set_Update_PosFbk_with_EncDiff
	Update_posfdk_latch    -> Set_posfdk_latch
	근접센서 변수 초기화	
	   Proxi_Sensor = Proxi_Sensor_old = Proxi_Sensor_new =0;
      Proxi_sensor_err_count=0;
     함수 모듈화     
      PROCESS_Homing_SwitchingRule(0);
      PROCESS_SyncMotion_EncoderDeltaCalc_EA_EB -> PROCESS_SyncMotion_Enc_Delcnt_EA_EB

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.84 20241206 RFID 오프셋 적용 시점을 일정시간 지연시키는 기능 추가(DCT서형식 수석 요청)
1) RFID 오프셋 적용 시점을 일정시간 지연시키는 기능 추가
	일반모드/동시모드 400ms 시간 지연 후 오프셋 적용함.
	파렛오픈시 적용된 파렛오프셋 0으로 초기화함 (D8: BG처리, D7:Extint1처리)	
2) 서보 알람 발생시 상태워드값 업데이트 안되는 현상 수정
   following error값을 상태워드 비트13에 등록하는 기능의 코드 오류로 발생하였음.(80~83버전)
3)기타 수정사항
     토크제한 플래그(0x2A98 LMS_TorqueLimit_readyFlag)를 Status Word 비트14 에 할당함.(주석만 수정)
     사용하지 않는 변수 삭제하여 메모리 확보
     함수 모듈화     
		void PROCESS_SyncMotion_SwitchingRule1(uI16 ax);            //20241206 스위칭룰1에 대한 내용 모듈화.
		void PROCESS_SyncMotion_TriggerCondition(uI16 ax);          //20241206 동시이동 트리거 조건 처리.
		void PROCESS_SyncMotion_EncoderDeltaCalc_EA_EB(uI16 ax);    //20241206 동시이동 인코더 개별 속도피드백 계산
		void PROCESS_SyncMotion_PositionUpdate_EA_EB(uI16 ax);      //20241206 동시이동 개별 인코더 위치값 업데이트

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.83 20241129 동시이동 사용시 엔코더 스위칭후 400ms 시간지연 후 RFID를 이용한 파렛오프셋 기능 
1) 동시이동 사용시 엔코더 스위칭후 400ms 시간지연 후 RFID를 이용한 파렛오프셋 기능
	-동시이동시 DCT 인코더의 처리 시간 부족으로 RFID값을 전송하지 못함.
	-인코더에서 RFID 처리하는 시간 최대 400ms 정도 필요함.  

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.82 20241122 동시이동 사용시 RFID를 이용한 파렛오프셋 기능 미동작 현상 수정 
1) 동시이동 사용시 RFID를 이용한 파렛오프셋 적용 안되는 현상 수정 
   -변경전: 파렛 외부 진입시 즉 EA and EB Valid 모두 OFF 상태에서만 동작함.
	           동시이동 구동시에는 인코더 신호가 모두 off되는 구간이 발생하지 않아서 검출 루틴이 동작되지 않았음.
   -변경후: EA or EB 트리거 신호 조건 발생시 검출하도록 수정함.
                동시이동 파렛 동시신호(EA/EB Valid OFF->ON) 진입을 알리는 시간틱(encPalletIncomingTick)
2) 메모리 iram 영역 부족으로 코드 추가 불가 수정
	터치프로브 파일을 dram 영역으로 이동함, TouchProbe.obj(.text)
	로터리 모터 구동시 이상없는지 확인 필요함(ECN 진행시 검토)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.81 20240903 위치정지 상태 + GO OFF 상태에서 위치값 변경시 속도명령 발생하여 파렛이 움직이는 현상 수정 
1) 위치정지 상태에서 위치값 변경시 속도명령 발생하여 파렛이 움직이는 현상 수정 
	이력 발생 상황:
		백호프 제어기 사용시 GO와 위치명령을 동시에 변경하여 진행하였음. 이와 같은 상황에서는 이슈가 발생하지 않았음												
	원인:
	PDO로 위치값(제1/2 위치값) 변경시 목표위치(target position)값은 변경되었으나 제어루프에서 목표위치(target position) 되었음을 인식 못함.												
	이로 인해 제어루프의 STATUS가 변경되지 않았고 이 상태에서 목표위치(target position)가 변경(위치에러 발생)으로 속도명령이 자동출력되었습니다.												
	FW에서 PDO로 동작시 현재값과 이전값을 비교하여 New 위치를 Old 위치값으로 업데이트하는 부분이 문제가 됨.												
	PDO에서 old 위치값이 업데이트가 되고 LMS control이 실행될 때는 조건문이 발생하지 않아서												
	결과적으로 현재 상태(13)를 유지하게 되었고 목표위치는 변경되었기 때문에 속도명령이 발생하였다.												
	수정:
	위치값은 이더캣 통신주기로 PDO/SDO를 통해서 업데이트만 되고 위치값 변경 유무는 LMS Control 루프에서만 검사하도록 수정함.											
	영향받는 STATUS:         
         LMS_POS_POS: // 13
         LMS_VP_POS:  // 53
         LMS_SM_POS:  // 63
	코드 변경점: 위치명령의 변경 여부를 확인하는 코드를 통합함.         
	변경전:
   	Update_TargetPosition_And_StatusChanging(0);
      lms_pos_dir = LMS_DIR_VALUE;
      Update_PositionOld_PositionNegative(0);
     변경후:
      Update_TargetPosition_with_PDOupdateTiming(0);
         
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.80 20240816 동시이동 최근 내용 정리(DCT전달)
1) MDE 174/175/176 추가(XML추가)
	0x2AAD 173 LMS_MOSI_GO 				   MOSI.2 기동명령비트
	0x2AAE 174 SyncEA_Velocity Feedback 동시모드에서 EA 속도피드백
	0x2AAF 175 SyncEB_Velocity Feedback 동시모드에서 EB 속도피드백
	0x2AB0 176 Sync_Velfdk Timetick		동시모드 속도피드백 시간틱
2) 동시이동 속도피드백 유지시간 기능 추가
	D7                D8            내부변수
	ft-2.48, 0x2230,  ft-2.48	SyncEA Velocity Feedback Keeping Time for EB Trigger	EB 트리거시 EA 속도피드백 유지시간
	ft-2.49, 0x2231,  ft-2.49	SyncEB Velocity Feedback Keeping Time for EA Trigger	EA 트리거시 EB 속도피드백 유지시간
3) 내부 변수 이름 변경(FW/XML)
	Sync_PalletInMon_EA -> SyncEA_PalletInCnt
	Sync_PalletInMon_EB -> SyncEB_PalletInCnt
	enc_normal_alarmreset -> IsDirectON_alramreset
4) 토크제한 플래그(0x2A98 LMS_TorqueLimit_readyFlag)를 Status Word 비트10 에 등록하여 알려주는 기능 추가
	현재 위치값(follow position)이 유효/무효한지를 알려주는 상태 비트임.
	파렛 외부진입하여 현재 위치값이 유효한지를 알려주는 플래그를 Status Word에 등록하여 알려주는 기능 추가
5) HW(input1) 비상정지시 적용되는 감속값(ft-2.07)이 정상적으로 동작하도록 수정함.
	ft-2.06, ft-2.07 로터리기준으로 되어 있는 것을 리니어 타입 구분하도록 수정함.
	기존에는 단위는 m/s^2인데 기본값이 로터리모터 단위로 41.667 rev/s^2로 크게 되어 있었다.
	default 감속도값=41.667[m/s^2] 단위일 때 정지시간은 대략 ~20ms(at 0.4 m/s)이내 빠른 감속으로 정지하였다.
	구동속도 : 0.2 m/s
	수정 전 FW: 41.667 적용시 감속시간 16ms 정도 측정됨.
	수정 후 FW: 0.2 [m/s^2] 적용시 감속시간 1[sec]로 정확하게 계산됨. 
	          0.4 [m/s^2] 적용시 감속시간=속도/감속도= 0.2[m/s]/0.4[m/s^2]=0.5[sec]
6) 통신끊김시 비상정지 동작하지 않는 오류 수정
   V01.20.10.12버전에서 수정한 내용이 비상정지 기능 동작시 고려되지 못했음. 
     통신 SafeOP 변경시 서보오프 안되는 현상을 수정할 때 비상정지 기능이 동작중이면 서보으프하지 않도록 조건문 추가하여 수정함.
	HW or 통신 비상정지시 MISO.15 비트가 1로 설정됩니다.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.79 20240809 내부 임시버전

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.78 20240731 D7&D8 FW 기능 통합 버전(DCT전달)
1) 기능상 77버전과 동일함.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.77 20240625 UDB백업시 자간거리 정보 오류 개선, 76버전에서 수정한 내용 중 필요없는 부분 삭제.
1) 모터정보의 자간거리 유효자리수가 보장될 수 있도록 처리하는 방법 정리
	원인: 모터DB 입력시 정수형->다운로드시 부동소수형 변환(서보 내부변수)->UDB 저장시 정수형 변환(원래 데이터와 오차발생)
	수정:1)3rd Party일때 다운로드된 모터정보를 사용하도록 command_SET 함수 수정.
		     LMMT FW만 special하게 처리되어 있어서 수정함.(D5/D7 표준 모델은 정상)
		 2)부동소숫점 타입 자간거리변수로 정수형 ppr을 구할때 최소 mm단위로 스케일링하던 것을 사용하지 않도록 원복함
			  Motor.Enc.ppr = Motor.Enc.Line_cnt * Motor.Cycle_len;

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.76 20240618 모터DB에 저장된 자간거리의 소숫점 발생으로 인한 ppr 계산값 오류 수정
1)	ppr 계산시 최소 mm 단위로 스케일링하여 계산하여 FullTurn 값이 잘못 계산되는 현상을 개선함.
	3rd Party 모터로 사용시 ppr 소숫점값에 의해 FullTurn 계산값 오차 발생하고 속도피드백 틔는 현상으로 나타남.
	64버전에서 수정한 모터의 자간거리값에 스케일 적용시 양자화 오차 발생되는 현상을 개선하면서 발생한 side effect임.
		Motor.Enc.ppr = (sI32)((Motor.Enc.Line_cnt * (uI32)(Motor.Cycle_len * 1000)) / 1000)
		HalfTurn 변수 실수값으로 계산하도록 수정
			HalfTurn = (uI32)((FP32)(FullTurn+1)/2. + .5)
		Del_cnt 계산시 HalfTurn을 초과하는 경우 (FullTurn+1)값을 더하도록 계산식 수정.
		 	ENCODER.Del_cnt = ENCODER.Del_cnt - SIGN(ENCODER.Del_cnt) * (FullTurn+1)
	
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.75 20240613 74버전의 오류 수정외 최적화
1) 엔코더 분해능에 따라 적용되는 게인 스케일 변수의 오류 수정, EncRes_gainScale
2) EtherCAT 제어/상태 워드 신호 임시 맵핑, 	MDM 79/80

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.74 20240611  DCT향 베트남 하이퐁 튜닝이슈, 속도 감속시점 조정 기능
1) 속도 감속시점 조정 기능
	ft-1.44 (0:disable, 0 아닌 위치값을 입력시 등속에서 감속되는 시점을 임의 조정 가능)
	위치정지가 필요한 위치/자동위치/동시이동 모드에 모두 적용함.
2) 동시이동시 스위칭시 토크어시스트 기능 추가
	ft-2.12 (0:disable, 0 아닌 값을 입력시 엔코더주기를 tick으로 계산한다. 1600입력시 62.5*1600=100ms)
3) 동시이동 트리거 EA or EB 신호 발생시 출력 기능, TG_ON신호(output 2)에 맵핑, admin=777
4) 엔코더 Del_cnt값 이상 모니터링을 위한 신호 맵핑 수정
	엔코더 이전값을 할당하는 부분을 rsware 모니터링 후에 적용하도록 함. ENCODER.Cnt_old = ENCODER.Cnt;
	전류uvw상 모니터링 :동시이동시 전류위상을 보기 위해 원복함.
	속도피드백 모니터링 : 엔코더 스위칭시 속도에러에 대한 전류명령 틔는 현상을 보기 위해 필터값 제거한 값을 보여줌, w_rad

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.73 20240528 자동위치모드 동작사양 수정(VP모드에 대한 6ms 필터 처리외) (DCT 남기혁 KNS대응) 
1) 자동위치 정지 후 PLC에서 비트8을 OFF시켰을 때 속도모드로 젼환되어 이동하는 현상을 막기 위한 개선 요청
       상태(13)에서 MOSI.1을 OFF하여도 현재 위치를 유지하는 것과 동일한 사양으로 동작하도록 해야 혼란이 없기 때문.
	기존: 상태(53)에서 MOSI.8 OFF시 MOSI.1의 값에 따라 속도(3)/(13)위치로 전환되었음.
	변경: 상태(53)에서 MOSI.8 OFF시 MOSI.1과 2의 조건에 따라 동작함.
2) 자동위치 모드  동작사양 수정 (LMS_POS_POS(13) 상태와 동일하게 동작하도록 수정함.)
      자동위치 정지(53) 후 MOSI.8=OFF && GO 비트의 on/off(에지) 발생시 MOSI.1 조건에 따라 속도(0)/위치(53) 상태로 이동함. (홈밍(20) 설정시 우선함)
      자동위치 정지(53) 후 MOSI.8=OFF && GO=ON 유지 조건일때 MOSI.1 조건에 따라 속도(0) 혹은 현재 위치(53) 상태 유지함.
      자동위치 정지(53) 후 MOSI.8=OFF && GO=OFF 조건일때 현재 위치(53) 유지
3) 동시이동 스위칭시 전기각 정보를 점진적으로 적용하는 방법 추가, tt2.13에 시간값 설정(default:disable)
4) 자동위치모드 비트에 대해 6ms 필터 처리
	GO명령과 동기를 맞추어 동시 OFF일때 현재 위치를 유지하기 위함
5) 동시이동 스위칭시 전기각 정보를 점진적으로 적용하는 방법(Type1을 사용함)
5) 모니터링
	Valid신호 변수 추가 (LMMT_Master_Valid, LMMT_Slave_Valid)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.72 20240423 디폴트모터 사용시 엔코더마스킹값 오류 수정, TPC향 시작버전
1) 분해능 10um사용시 신규 홈밍동작 오류 발생할것으로 예상되어 수정함.	EncMask 0xfff->0x1fff

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.71 20240419 홈밍 속도0 정지 오류 수정, 대기시간 100ms 추가

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.70 20240417 Homing Mode수정
1) 홈밍트리거 이후 Offset값에 따라 속도 가감속이 발생하지 않는 현상 수정
  - LMS_STATUS 추가
     LMS_HOME_VEL_TRIG_STOP,  //25  홈트리거를 만났을때 이미 위치를 지났으면 정지하는 단계
     LMS_HOME_OFFSET_INIT,    //26
     LMS_HOME_OFFSET_RUN,     //27  정지 후 Offset값 만큼 이동하는 단계
  - 홈트리거를 만난 후 정지위치 즉 감속거리가 5mm이하이면 위치 정지후 Offset위치로 가도록 수정.
  
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.69 20240416 자동위치모드 기능 동작 개선 요청(DCT KNS대응)
1) 위치완료 상태에서 자동위치모드로 변경시 파렛이 이동하지 않는 현상으로 후행 파렛 충돌현살 발생
       원인: 위치완료 상태(13)에서 자동위치모드 시작(MOSI.8+GO)명령을 전달하면 바로 자동위치모드의 위치완료(50-51-52-53)로 진행함.
      T13_50   위치완료(13)->자동위치모드 준비(50), STATUS 13은 서보온+GO 위치완료 상태이므로 T13_53으로 자동위치모드 위치정지 상태로 정상동작함.
       동작 개선:
           자동위치모드가 켜져있다면 항상 속도이동하여 현재 파렛을 배출함.
     MOSI.8=1인 경우 MOSI.1(Vel/Pos)에 상관없이 무조건 현재 파렛을 속도이동하여 배출하고 open시 LMS_STATUS 50으로 진행함.
       동작 수정(3가지 CASE):
      VP13_01 위치완료(13)->속도모드초기(01)->자동위치모드 준비(50)
      VP10_01 위치준비(10)->속도모드초기(01)->자동위치모드 준비(50)
      VP02_50 속도이동(02)->자동위치준비(50)
      
///////////////////////////////////////////////////////////////////////////////
0) V01.20.90.68 20240408 TPC향 FW : 38~46버전, 47~67버전에서 발생된 위치틀어짐 오류를 수정
1) 홈밍동작 원복
2) Z상 출력 신호를 기계각 모니터링 변수에 임시 맵핑, LMS_EtherCAT_Z
3) 38~46버전, 47~65버전에서 발생된 위치틀어짐 오류를 수정. DCT검증필요 (DCT엔코더 사용시 주의사항)
   1pallet 상태와 2pallet 상태일때 위치증분값을 동일하게 적용하지 않도록 수정함.
   2pallet 상태일때만 위치증분 old값(LMMT_pos_fdk_del_old)을 적용하고
     다른 경우는 1pallet 일때와 동일(위치값 불연속 처리)하게 적용해야 위치반복 정밀도를 유지하게 된다.
   2pallet되는 시점은 상태머신2->3으르 변경되는 시점임.(조건: SyncEA_Status == SM_TRANS_STEP3)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.90.66 20240405 위치반복 오차 발생 수정한 버전 66버전 디버깅.(임시)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.66 20240314 위치반복 오차 발생 수정한 버전
1) 동시구동 모드에서 2pallet진입시 pos_fbk 잘못 적용되던 버그를 수정한 것을 다시 원복함. 위치반복 오차 발생 이슈 발생함.
       아래 코드 삭제함.
   if(abs(LMMT_pos_fdk_del) > 200)
   {
       LMMT_pos_fdk_del = LMMT_pos_fdk_del_old;
   }
2) 속도에러 스케일 잘못된 버그 수정, 속도에러=속도명령-속도피드백으로 계산
3) 동시이동 내부 위치모드로 변경시 상태 플래그 추가, LMS_TG_mode_flag
	MDE 163 SyncEA_StartValue 에 맴핑

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.65 20240311 동시 이동시 속도0에서 출발하지 않는 현상 개선을 위해 관련 기능을 원복함
1) 외부엔코더 사용유무에 따라 속도정지 판단 기능 적용되도록 수정함. 
  DCT엔코더의 외부엔코더 사용시에 위치증분값 0으로 입력되는 경우가 있음(엔코더 FW 성능 한계가 있다고 함, DCT 남기혁)
  -서보 FW에서 이를 방지하기 위해 로직 삭제한 것을 다시 원복함.
    엔코더 선택(홀타입/외부스케일)	4	Selection for Hall/Scale Encoder 
  ft-2.42 D3 0일때 홀엔코더 타입, 속도정지 판단 기능 Enable
             1일때 홀엔코더 타입, 속도정지 판단 기능 Disable  

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.64 20240228 RFID값 엔코더로부터 사용하지 않을 경우  0으로 초기화 요청(DCT 남기혁책임 요청사항)  
1) 엔코더로부터 파렛오프셋을 사용하지 않을때  RFID_DF6 모니터값을 0으로 표기되도록 개선함.
2) 모터의 자간거리값에 스케일 적용시 양자화 오차 발생되는 현상을 개선함.
	->UDB로 모터 정보를 저장하고 로드를 반복하는 경우 발생 
		->자간거리(Motor.Cycle_len)는 내부에서 1/10000배 스케일 변경하여대한 반올림 적용하여 정수로 보이도록 함.
	    입력값 30->29.9로 보임, 29.9를 저장하여 다시 Write하면 29.8로 보임.. 이렇게 0.5 보다 작을때 까지 계속 변함.
3) 홈밍시 홈밍오프셋 적용시 가감속 발생하지 않는 현상 개선(Command polarity:Normal에서만 동작확인함)
	가감속 발생하지 않을때는 오프셋값이 트리거위치와 비교하여 2500펄스(1um시 25000펄스) 이내이고
	트리거를 만나는 속도의 방향과 반대위치일때 가감속없이 속도명령이 발생함으로써 부하에 충격이 발생함. 
	->트리거 후 일시 정지하고 진행하여 충격을 완화시키도록 홈밍로직 변경함.
	->홈밍오프셋 거리가 25mm 이내인 경우 creep 속도(홈밍속도의 25%로 설정됨)를 적용한다.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.63 20240205 속도피드백 모니터링 0으로 보이도록 개선함.(DCT 최봉철책임 요청사항)  
1) DCT 파악 내용 및 요청 사항
	1.문제 사항 : 로그 상에 FeedBack 되는 속도가 정상적이지 않다고 판단하여, 해당 로그 이력을 수정하기 위한 목적 / 실제 구동은 정상입니다.
	2.펌웨어 버전 : CSD7N_V_01_20_10_35_20230717 (첨부된 자료)
	3.로그 데이터 중 커맨드 속도 기록 여부 : X 데이터 없음
	4.구동 중 커맨드가 바뀌어 해당 속도 변경 여부 : X 1m/s 로 등속 / 구동 상 커맨드 변경 없음.
	5.수정 요청 사항 :  팔렛트가 코일 위 진입 이탈 시 Feedback Velocity가 0이 아닌 다른 값으로 변경되는 사항 개선 요청 ( 실구동과 상관 없음.)
	6.현장은 ANI(수원)이고, End user는 삼성
	7.개선 요청사항
	-실제 구동이 정상이고 파렛가 없음에도 기존 속도 정보를 남기고 있는 것은 일반적인 상태에서는 정상은 아니라고 판단 되어, 
	  구동상에 문제가 없으면, 앞으로 제작되는 펌웨어에는 해당 내용(파렛 이탈 시 속도값은 0) 0으로 표기하였으면 합니다.
	-코일 위에서 이탈할 경우 속도피드백을 0 로 표시되도록 개선함.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.62 20231213 DCT동시이동 이상을 디버깅하기 위한 내용 추가 
1) 동시이동 중 서보오프시, 캐리어 오픈시  모두 동일한 동작이 되도록 상태머신 천이 수정
	-동시이동 상태머신을 벗어나면 엔코더스위칭이 변경되어 오작동 가능성이 있어서 수정함.
	-LMS_POS_READY->LMS_SM_READY
	LMS_SM_VEL
	LMS_SM_POS
	LMS_SM_STOP
2) Sensor polarity 설정기능 추가
	Home Sensor, POT, NOT Polarity 설정 기능 Ft-5.38(0x2526) 추가
3) 동시이동 이상 발생용 디버깅 내용 추가 
	SyncEA_Status, SyncEB_Status 변수 enum 타입으로 변경 
	임시 신호 맵핑
	81	   Ltmp = CntCC; // 16회/1ms
	82    Ltmp = CntSC; // 4회/1ms
   83    Ltmp = CntMonitor; //12회 /1ms	
	         
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.61 20231206 RFID값이 0일때 예외처리 추가(DCT전달)
1) RFID값이 0으로 수신되는 경우 파렛오프셋이 미적용되도록 모두 0으로 초기화함.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.60 20231124 20231123 20231121
1) 동시멀티이동 2pallet 2shift, 0x222F 추가, 모니터링 변수 정리
2) 동시이동 완료하고 같은 방향으로 동시이동 진행시 충돌현상 발생
  -진행방향이 같을때에도 진입 파렛 카운터 변수를 초기화함.
3) 위치완료상태에서 서보온이고 CCW방향의 GO비트가 살아있을때 동시모드 진입하면 CCW방향으로 구동되는 현상
     위치준비(10) 상태로 천이되고 기동비트가 살아있어서 설정된 방향 CCW방향으로 속도이동되기 때문임.
	-위치완료(13) 상태에서 동시모드(MOSI.9) 진입시 동시모드 위치정지(63)으로 변경되고 현재 위치 유지하도록 동작 개선
	T13_50 : 위치모드에서 VP모드 진입시 위치완료(13)에서 자동위치로 즉시 이동 
	T13_63 : 위치모드에서 SM모드 진입시 동시위치완료(63)으로 진입되고 현재 위치를 유지한다. 

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.59 20231120 모니터링 변수 정리 중 오류 수정, 59버전에서 빌드날짜만 바뀜
 
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.59 20231117
1) 모니터링 변수 정리 & XML 수정
	이름 수정
	2A36 54 LMMT_Encoder Data Differrance ->LMMT_Encoder Data Difference
	2A38 56 LMMT_Dead Zone Flag           ->LMMT_Initial Position
	2A3B 59 LMMT_RFID                     ->LMMT_RFID_DF6
	2A3E 62 LMMT_Initial Pos              ->LMMT_ABS_DF4DF5
3) MDM 45 모터 최대부하율 원복

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.58 20231116
1) 동시이동모드에서 구동명령 방향과 반대로 이동하는 현상을 개선함
     관련변수: EcatLmsFlag.START_IXG,   LMMT_MOVE, LMS_DIR_VALUE,LMSFlag.V_SELECT
  -명령비트(MOSI)가 디지털 입력 필터(6ms)후에 처리되는 것을  outer 루틴으로 이동함.
  -방향(MOSI.3)에 따라 위치티칭값 적용이 안되는 현상을 개선하기 위해  동기가 되도록 outer 루틴에서 처리하도록 이동함.
  -기동명령 (MOSI.2)시 방향(MOSI.3)이 적용 안되는 현상을 개선하기 위해 동기가 되도록 outer 루틴에서 처리하도록 이동함.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.57 20231115 동시이동 관련 오브젝트 추가 (0x2AA1~0x2AA8)
1) 동시이동 파렛 카운팅 설정 변수 사용, ft-2.47 [D0..D1]
   LMS_palletOutCnt_CW
   LMS_palletOutCnt_CCW
2) 동시모드일때만 엔코더 스위칭시 전기각 오프셋 보정 옵션을 사용하도록 수정, ft-2.15
3) 2파렛 이동시 CW운전하고 CCW로 운전할때 파렛이 분리되는 현상, 53버전에서 수정부분에 대해 추가로 보강함.
	- SM_INIT(61)의 마지막 단계에서 위치명령이 업데이트 하도록 변경함. 
4) 변수 이름 변경
   SYNC_POS_NEW_PALLET-> SYNC_EA_NEW_PALLET
	SYNC_NEG_NEW_PALLET-> SYNC_EB_NEW_PALLET
	Sync_MovePalletCount1-> Sync_PalletInMon_EA
	Sync_MovePalletCount2-> Sync_PalletInMon_EB
         
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.56 20231114 55버전에서 변경한 타이머14 인터럽트(INT14_Timer0Handler)에 우선순위 설정을 이전 상태로 원복함
1) 구동중 랜덤하게 E203 발생함
   55버전에서 변경한 타이머14 인터럽트(INT14_Timer0Handler)에 우선순위 설정을 이전 상태로 원복함
       타이머가 우선 순위 밀리면서 DCrunning 플래그가 false 상태 상태로  변경되는 것으로 예상함.
       이 부분은 이더캣 스택과 관련되어 있어서 스택을 최신버전으로 함께 변경해야 해결될것으로 보임. 
       양산버전과 호환성을 위해서 이전 상태로 원복하며 추후에 스택 업그레이드 후 충분한 시험과 검증이 필요한다.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.55 20231109 
1) 엔코더 1회전시 2개의 펄스가 출력되도록 함 Fullturn을 4구간으로 나눔
   오실로스코프로 확인할 수 있도록 Digital Output #1 에 맵핑, (admin 777입력시 동작) (패스워드 없을때 ft-0.28 D2 LMS_Homing Status가 맵핑되어 있음.)
2) 타이머14 인터럽트(INT14_Timer0Handler)에 우선순위가 설정되지 않아 Inner_Rutine 주기가 늘어나던 버그 수정.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.54 20231108 0x4000번 오브젝트 PDO 맵핑 추가, RFID 동작 사양 반영(DCT 남기혁책임)
1) 0x4000번 오브젝트 PDO 맵핑 추가
2) RFID 동작 사양에 따른 수정사항 반영 (위 동작사양 참고)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.53 20231107
1) Command Polarity Inverted 상태에서 CCW 구동시 위치운전으로 잘못되는 동작하는 현상
	- CCW(AX3->AX1)로 이동시 마지막축(AX3)에서 발생
	- 통신 DIR(FWD/BWD) 지연으로 인한 위치명령이 SM_READY(60) 상태에서 업데이트 되지 않음
	- SM_INIT(61)에서 DIR이 바뀌는 것을 확인하여 이 상태에서 위치명령 업데이트 하도록 수정함.
	- 인터럽트 마진 부족으로 예상됨.
	- 현재는 위치운전으로 진행되나 파렛 배출 방법이 결정되면 이 부분은 FW 블럭이 수정될 예정임.
2) 0x4000번지 Pallet Enc Offset Monitor SDO 읽기오류나는 현상 수정, PDO상 일기는 추후 수정.
2) 52버전은 코드가독성이 떨어지고 디버깅이 안되므로 사용금지하고 51버전으로 롤백함.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.51 20231031
1) 동시이동모드 2pallet 진입시 전기각 보정 로직 수정(RSA, DCT엔코더 동일)
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.50 20231030
1) 48버전에서 수정한 pos_fbk 수정사항이 엔코더(DCT, RSA) 사양에 따라 다르게 적용되어야 하는 부분을 반영함.
1) 속도기동시 뒤로 밀리는 현상 개선 (동시이동 +RSA엔코더 사용시만 적용됨)
2) 동시이동모드 진입하는 파렛의 속도명령에 적용하는 피드백 옵션 기능 추가

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.48 20231029
1) Command Polarity가 Inverted일때 
   - 동시구동 모드에서 2pallet진입시 전기각 옵셋 보정로직 적용.
   - 동시구동 모드에서 2pallet진입시 pos_fbk잘못 적용되던 버그 수정
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.47 20231023
1) 동시구동 모드에서 한코일 위에 2pallet진입시 pos_fbk값이 잘못 적용되던 버그 수정
   - 정방향 구동시 한 코일 위에 두개의 pallet가 진입시 TxDATA는 5 -> 3으로 스위칭됨. 이때  pos_fbk에 LMMT_ID3_Enc값을 넣어주도록 수정함.(역방향 구동시(TxDATA 5 -> 3)일 때도 동일한 방법으로 수정함)
   -  TxDATA는 5 -> 3 또는 3 -> 5로 Enc 스위칭 시 LMMT_pos_fdk_del값이 잘못 계산되던 버그 수정.
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.46 20231011 20231013
1) 방향전환시 전기각 오프셋 변수 초기화함.
2) 전기각 오프셋 old값 적용시 1주기 이전값을 적용함
   TX:3->5 변경시 CW방향 단독구간 진입시 계산식
     Enco_2Poffset_old = (sI32)(LMMT_ID5_Enc_old - theta_cnt_tmp); // 1)CW A->B pallet중앙
   TX:5->3 변경시 CCW방향 단독구간 진입시 계산식
	  Enco_2Poffset_old = (sI32)(LMMT_ID3_Enc_old - theta_cnt_tmp); // 2)CCW B->A pallet중앙

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.44->V01.20.10.45 20231005
1) 파렛오프셋 관련 수정 함수 이름만 변경
	LoadEncoderOffset->LoadPalletOffset
	SaveEncoderOffset->SavePalletOffset
	ReadEncoderOffsetFromFlash->ReadPalletOffsetFromFlash
	WriteEncoderOffsetToFlash->WritePalletOffsetToFlash
2) Rsware상 전송하는 ERD 패킷 응답 오류 수정(내부변수는 정상임)
3) 1pallet 이동시 엔코더 스위칭 시점에 전기각을 보상하는 로직 추가
   - 2pallet 일때는 스위칭룰이 달라서 보상값이 잘못되는 오류가 있음.  

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.43->V01.20.10.44 20230919
1) 동시이동 위치정지 후 위치티칭값을 변경할때 속도이동되는 버그 수정
	SYNC_POS_NEW_PALLET 플래그 동작을 수정함.
	ON 조건: 동시위치정지 후 위치티칭값 변경시 ON
	OFF조건: 동시모드 진입시 OFF
   
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.42->V01.20.10.43 20230915
1) 동시이동의 속도모드 배출시 위치정지하는 현상 수정(임시)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.41->V01.20.10.42 20230914
1) 엔코더 양간오차 전기각 보정기능 추가: CW보상 정상, CCW보상 정상 동작함.
2) 동시이동의 속도모드 배출시 위치정지하는 현상 있음.
     동시이동 제일 뒷단의 서버축을 0x205(동시이동속도)로 명령으로 이동중 정방향 티칭위치값을 지날때 위치정지하는 오류
 
  
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.40->V01.20.10.41 20230913
1) CW보상 정상, CCW보상 수정중

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.39->V01.20.10.40 20230911
1) 스위칭룰1 함수로 분리처리, 기능상 39버전과 동일함.
   PROCESS_SyncMotion_SwitchingRule1()

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.38->V01.20.10.39 20230906
1) 동시이동 Inverted설정시 1 shift CW/CCW 기능 검증
     동시이동시 후행엔코더의 시작값 설정이 Normal일때는 정상, Inverted일때 잘못되는 코드오류가 있어서 수정함.
     삭제한 코드: LMMT_Enc_data_old = Enc_data; // 위치오차=0
2) 동시이동 관련 변수 추가
   -MDM (161~166)
   -동시이동 파렛 카운팅(pallet counter for moving) 설정 변수, ft-2.47
   -파렛 카운팅 상태변수 : Sync_MovePalletCount1, Sync_MovePalletCount2

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.37->V01.20.10.38 20230904
1) 동시이동 Normal설정시 1 shift CW/CCW 기능 검증

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.36->V01.20.10.37 20230810-1
1) 절대치 매크로 로직 누락된 부분 추가 변경, abs ->labs로 로직 변경
2) 외부엔코더 사용시 위치증분값 0으로 계산되는 현상이 발생하지 않도록 관련된 로직 삭제함.
   ft-2.44 와 관련된 코드 주석 처리함.
     주석처리한 코드: if(Pos_oscill_cnt >=5) { vin_out = 0; PreProcess_flt = 0; }

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.35->V01.20.10.36 20230810 외부엔코더 사용시 위치증분값 0으로 계산되는 현상
1) 절대치 매크로 로직 변경, abs ->labs로 로직 변경

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.34->V01.20.10.35 20230717 서보오프 상태에서 파렛 이동중 서보온시 프리런으로 계속 진행되는 현상 수정
1) 컴파일러 옵션 변경
   ${CCS_UTILS_DIR}/bin/gmake -k
   ->${CCS_UTILS_DIR}/bin/gmake -j 8
2) 서보오프 상태에서 파렛 진입하여 이동중 서보온시 프리런으로 계속 진행되는 현상
     서보온시 빌트인에는 서보온 dot 표시됨.
     파렛을 손으로 밀다가 멈추는 순간에 서보온이 동작하는데 만약 멈추지 않으면 계속 프리런으로 빠져나가면서 낙하 위험성 있음.
   :제어오류로 인해 폭주하는 경우가 있는데 강제 서보온을 하여 파렛이 추락하지 않도록 하기 위함.(DCT요청) 
   -서보온 시퀀스상의 속도피드백 검사루틴에서 서보온되는 상한값을 파렛 구동속도를 고려하여 변경함. 50mm/s->1000mm/s    
      
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.33->V01.20.10.34 20230714 S커브 적용 개선 
1) S커브 실시간으로 적용안되는 현상 수정
2) 코드 정리후 가용메모리양(최종)
  IRAM                  11802000   0003a000  0003809b  00001f65  RWIX
  
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.33 20230713 속도제어에 사용하는 속도피드백 필터 적용시 오류 수정+코드 최적화
1) DVSC.c 파일을 SDRAM으로 이동한 경우
	IRAM                  11802000   0003a000  00039b2b  000004d5  RWIX
	IRAM                  11802000   0003a000  0003910b  00000ef5  RWIX (옮긴후 메모리 확보양)
2) main.c 를 파일을 SDRAM으로 이동함.
	IRAM                  11802000   0003a000  0003910b  00000ef5  RWIX
	IRAM                  11802000   0003a000  00036aeb  00003515  RWIX (옮긴후 메모리 확보양)
3)속도명령 순간적으로 피킹되는 현상때문에 S커브 적용되는 함수 임시 삭제 
  ApplicationPara(208, Para[2][8].val);
4) 속도피드백 필터관련
 -MAF 필터 계산 위치 변경, 엔코더 스위칭을 고려하여 보정된 w_rad 계산 이후에 이동평균 속도를 계산함.
 -최소 4Hz->2Hz 이상 가능하도록 변경   

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.32 20230707  코드 최적화 검색어: CODE_REDUCTION
1) TestFunction, TEST_FPGA_Status 함수 삭제
    -강제로 알람,경고를 발생시키는 테스트코드 삭제
2) 프로젝트 수정사항
   -빌드에서 modbus, indexing 제외, cmd파일 수정
   	LMS에서 indexing 변수 사용하던 것을 전역변수로 신규 생성
   -Utilities파일을 iram 할당
3) 최적화 결과 : admin 패스워드 700설정+스코프 0.125us 연속트리거+모션동작시 Rsware 정상동작함. 
	before>>>
	        name            origin    length      used     unused   attr    fill
	----------------------  --------  ---------  --------  --------  ----  --------
	  IRAM                  11802000   0003a000  00037bb3  0000244d  RWIX
	  SDRAM                 c0000000   00700000  000ef723  006108dd  RWIX
	
	after>>>
	         name            origin    length      used     unused   attr    fill
	----------------------  --------  ---------  --------  --------  ----  --------
	  IRAM                  11802000   0003a000  00039b2b  000004d5  RWIX <--다 집어 넣고 남은 메모리
	  SDRAM                 c0000000   00700000  000d3cb3  0062c34d  RWIX
	   	
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.31 20230705 속도피드백 필터사용 옵션 검증중 통신마진 부족으로 Rsware 끊김 발생
   LMS control 처리부에서 필터 초기값 설정은 LPF를 MAF필터로 대체함.
   LMS control 처리부에서 속도/위치에서 필터 초기값이 설정된 이후 구동시점에 필터계산을 수행함.
1) 속도피드백 주파수 1~3Hz의 작은 값에 대해서 최소값 4Hz로 적용
2) 파렛이동을 위한 속도 피드백 선택 방법, 0x222A subindex1 (ft-2.42 D0)
 	0 : raw data (속도명령)
 	1 : Moving Average Fiilter
 	2 : 1st LPF-> MAF적용
 	3 : 2nd LPF-> MAF적용
 	4 : Velocity Cmd Following Method (속도명령 추정)
3) 속도제어기에 사용되는 속도피드백 선택 방법, 0x222A subindex2 (ft-2.42 D1)
    0 : (속도피드백) raw data 
    1 : 1st LPF filtered
    2 : 2st LPF filtered
    3 : MAF filtered
	ft-2.40 값이 0(disable)이면 raw data 적용함
  <<시험결과>>
    - Kvp, Kvi(30,30), Kp=40 at 200W모터에서 실험데이터
    - 속도 1차LPF: 30Hz, 2차LPF:100Hz 정도 적합함. 
	- 파렛진입시 속도 추정 방법은  4(Velocity Cmd Following Method)
	    속도제어기에 사용되는 속도피드백 선택 방법은 1(1st LPF filtered) 을 적용할때 속도리플/전류소음을 기대 효과가 있었다.
4) 파렛 정지속도 판단 기준
    엔코더 증분값이 ft-2.44보다 작거나 같은 경우가 5회 이상 발생시 파렛 정지 상태의 떨림으로 판단하여 속도명령 0으로 출발한다.
   이 값을 크게하면 이동중인 경우도 포함되므로 구동 속도에 따라 적절한 값으로 설정해야 한다.
   예시) D8의 경우 1펄스는 속도 0.1m/s에 해당하고 구동속도가 0.5m/s이고 엔코더더가 +=20% 오차로 측정되었다면
         이 값을 3(=0.3m/s)이하로 설정한다.
5) 기타 수정사항
	1) 속도피드백 평균횟수 변경 : 4->8 ((DCT요청사항)
    	-속도모드(기반영), 위치/자동위치(적용), 스텝(미적용), 동시모드(적용)
	2) 신규 파라메터 서보온중에도 실시간 변경하여 성능 확인을 할 수 있도록 속성 변경
	3) Velocity Error 모니터에 속도피드백 필터 적용된 에러(속도명령-속도피드백) 값을 표시함.
	4) 엔코드 스위칭시 Direct(off), Tranf(on)되는 특수한 경우에 엔코더값이 0 으로 초기화되는 현상을 개선함
   		-동시이동과 같은 스위칭 조건시 발생할 수 있음. 
	    :CW방향 설정된 상태에서 CCW방향으로 손으로 파렛을 밀면 valid B->A&B->A (스위칭발생) 시점에 발생함.
   		-또는 valid A->A&B(스위칭발생) 발생시 Valid A(off), Valid B(on) 상태 발생함.
   		ID3에 대한 Valid 신호는 OFF, ID5에 대한 Valid 신호는 ON 상태가 될때가 있음. 
     	이때 enc_data값이 한주기 0으로 보임->속도리플 발생 가능성 있어서 개선함.
     
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.30 20230626
1) 오브젝트 데이터 타입 수정(0x2228, 0x2229)
2) RD_BITIDX(), WR_BITIDX()함수에 오브젝트 0x222A, 0x2316에 대한 case문 추가
3) XML수정(V2.4.0)
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.29 20230626
1) 속도 2차 LPF 초기화 코드 추가
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.28 20230623
1) 2.42 D1 값이 3인 경우 속도피드백 이동평균값을 적용하도록 추가함.
2) 오브젝트 이름, 타입 수정

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.27 20230622 파라메터&오브젝트 추가, 속도피드백 선택방법 추가, 토크어시스트 적용, 속도제어기에 사용되는 속도피드백 선택방법 추가, RSware수정, XML수정(V2.4.0)
       파라메터&오브젝트 추가
   - Ft-1.45 D2 0x212D:03: LMS Positive VC IntegralSum Enable
   - Ft-1.45 D3 0x212D:04: LMS Negative VC IntegralSum Enable
   - Ft-1.61 0x213D: Positive Torque Assit Value
   - Ft-1.62 0x213E: Negative Torque Assist Value
   - Ft-2.25 ~ Ft-2.39 Reserved로 추가
   - Ft-2.40 0x2228: Velocity Feedback Filter Cutoff Frequency
   - Ft-2.41 0x2229: Jerk Acc/Dec Time
   - Ft-2.42 D0 0x222A:01: Velocity Feedback Select for Moving
   - Ft-2.42 D1 0x222A:02: Velocity Feedback Select for Velocity Loop
   - Ft-2.42 D2 0x222A:03: Jerk Limited Dec Profile Enable
   - Ft-2.43 0x222B: Velocity Cmd Following Threshold
   - Ft-2.44 0x222C: Encoder Delta Threshold for Motor Stop Detection
   - Ft-2.45 ~ Ft-2.46 스마트튜닝 조건 설정을 위해추가
   - Ft-3.22 D0 0x2316:01: Vibration Auto Search 
   - Ft-3.22 D1 0x2316:02: Vibration Filter Type
2) 파렛이동을 위한 속도 피드백 선택방법 추가
   - Ft-2.42 D0값에 따라
     0: Raw data
     1: Moving Average Filter
     2: 1st LPF
     3: 2nd LPF
     4: Velocity Cmd Fllowing Method(속도명령 추정)
3) 속도 제어기에 사용되는 속도 피드백 선택방법 추가
   - Ft-2.42 D1값에 따라
     0: Raw data (속도 피드백)
     1: 1st LPF Filtered
     2: 2nd LPF Filtered
     (Ft-2.40값이 0(Disable)이면 Ft-2.42 D1은 비활성화 되고, 속도피드백은  Raw data를 적용함)
4) 토크 어시스트 적용
   - Ft-1.45 D2값에 따라
     0: Disable 이면 미적용
     1: Enable 이면 Ft-1.61값을 토크어시스트로 적용.
  
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.26 20230609 터치프로브 모니터링 기능 추가, XML수정(V2.3.0)
1) 모니터링 관련 
   -PDO상 4byte 정령을 위해 temp_xx 변수 사용함,
   -터치프로브 관련 변수 PDO 등록 추가, 60BA/60BB/60BC/60BD/60D5/60D6/60D7/60D8 
   -60D5/60D6/60D7/60D8 DINT 타입으로 확장.
   -모니터링 변수 업데이트를 Extint2에서 업데이트되면서 중복 코드 삭제함.
   -0x6074, 토크명령 모니터링 변수는 토크피드백과 같은 단위로 보이도록 함. 0.001 [Amps]
   -U/V상 모니터에 확장센서값(ExtPHSensor)을 표시도록 변경함.
2) 제2위치값의 오브젝트 번호 잘못된 부분 수정
3) XML 수정사항
   -터치프로브 에지 카운터는 모니터링 변수로 쓰기 위해서 DINT으로 변경함
   -오브젝트 번호 잘못된 부분 수정
    0x2142->0x213C
    0x2143->0x213D
    0x2144->0x213E

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.25 20230608 역주행 방지 기능+FOE 펌웨어 업데이트 표시 플래그+터치프로브 에지카운터 추가
1) 모니터링 오브젝트 & MDM 추가
  0x6074  xx Torque Demand Value   전류명령 모니터링용
  0x606B  xx Velocity Demand Value 속도명령 모니터링용
  PDO/Rsware 공통
  Object MDM Name                  내부변수     
  0x60BA  38 엔코더 증분값                       ENCODER.Del_cnt
  0x60BB  39 엔코더 증분값 과거값             ENCODER_DEL_CNT_OLD
  0x60BC  40 속도명령                              w_command
  0x60BD  41 속도에러                              w_err*1000
  0x60D5  81 (TBD)  
  0x60D6  82 (TBD)
  0x60D7  83 (TBD)
  0x60D8  84 (TBD)
2) 사용자 PDO맵에 6074/606B/60BA/60BB 추가
  6041  Status Word
  3101  LMMT MISO
  6064  Position Actual Value       위치피드백 [pulse]
  606C  Velocity Actual Value       속도피드백 [pps]
  6077  Torque Actual Value         토크피드백 [0.001Amps]
  6074  Torque Demand Value         토크명령 [0.001Amps]
  606B  Velocity Demand Value       속도명령(속도제어기 입력단, w_rad) [mm/s]
  60BA  Touch Probe Pos2 Pos Value  속도명령(프로파일 출력단, w_command) [mm/s]
  60BB  Touch Probe Pos2 Neg Value  속도에러(w_err*1000) [0.001mm/s]
3) 역주행 방지 로직1/2 추가
    -로직1: 스위칭동안 엔코더 1주기 이전의 증분값을 적용하는 방식
    -로직2: 스위칭동안 속도계 1주기 이전의 값을 적용하는 방식
4) FOE를 이용한 FW 다운로드시 빌트인 표시 개선
   -다운로드시 시작시 "GO", 완료시 "done", 다운로드 에러 발생시 "Error"
   -FOE를 이용한 다운로드시 자동 드라이브 리셋이 되지 않으므로 새로운 버전을 적용할려면 반드시 전원 on/off를 해야 한다.
   -FW 업데이트가 한번이라도 완료된 상태이고 전원을 끄지 않았다면 "done" 표시는 유지된다.
   -FW가 업데이트되었고 전원을 꺼지 않았은 경우 사용자가 알수 있도록 오브젝트(터치프로브 함수)에 표시함
   -FOE로 FW업데이트 되었음을 알려주는 상태 플래그, MB_firmwareUpdatedFlag = FALSE;   //2023
5) 위치 정지시 0속도 제어로 정지하도록 개선함
   -서보온상태에서 파렛이 진입할 때  위치 정지명령이면 프리런되는 현상 개선
6)초기 전원부팅시 에러발생하면 cia402 ErrorCode에 표시되도록 수정함.
   0x603F 에러코드 표시안되는 오류 수정
   -부팅시 발생 알람이 0x603F에 반영되지 않는 문제 수정.  
7) 기타 수정사항 
  -오브젝트 번호 잘못된 부분 수정
    0x2140->0x213A
    0x2141->0x213B
    0x2142->0x213C

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.24 20230215 DCT요청사항
1) DCT에서 확인한 엔코더 정보 이력사항 수정
2) 확장센서와 홀센서는 고정자리에 맵핑되도록 사양 결정

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.23 20230206
1) 시간 단축을 위한 내용 추가
     -엔코더 값을 ASCI에서 한번만 읽어와서 처리함, localReg5810
     -PDO 맵핑함수 내부 중복 코드 단일화 처리함.
2) 2차위치값 설정 버그 수정
   MOSI.9 -> MOSI.13 로 변경
3) 엔코더 A/B raw 싱글턴 데이터 PDO 맵핑
4) 확장 근접센서(DHE) I/O 4점 지원

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.22 20230125 21버전과 동일함.
1) 속도피드백 이동평균 적용을 위한 코드 추가
   ft-1.17=1일때 LMS_vel_maf_enable 플래그로 동작함
2) 위치모드 변경 경계점 파라메터로 추출
   ft-1.44=2500 (default) LMS_pos_switching_threshold 변수로 사용됨.

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.21 20230113 속도음수구간 개선 RSA 엔코더사용시에만 적용

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.20 20230110 DCT요청사항
1) 2차 위치티칭값 파라메터 추가, ft-1.59, ft-1.60
   위치선택: MOSI.13
2) 토크리미트, 토크오프셋 PDO 추가
   0x60E0, ft-4.07
   0x60E1, ft-4.08
   0x60B2, torque offset
3) 엔코더 A/B을 값을 PDO로 업데이트 추가
	0x2AA9 LMS_ID3_Enc   
	0x2AAA LMS_ID5_Enc   
	0x2AAB LMS_Master_Enc
	0x2AAC LMS_Slave_Enc 
4) DP_objTorqueDemandValue 추가, [0.1%]
5) 통신스텍 일부 업그레이드
   APPL_InputMapping, APPL_OutputMapping

///////////////////////////////////////////////////////////////////////////////
V 1.20.10.16~19 : skip (이전과 변경사항이 많아서 예비버전으로 남겨둠)

///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.15 20230106
1) 속도음수구간 발생 수정사항이 DCT 모터지그에서는 좀더 검증이 필요하여 코드 원복함.
     파라메터 ft-1.19) 설정을 바꾸어야 하는데 양산라인에 적용하지 못하는 관계로 오작동 가능성이 높음.

20230106
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.14 20230106
1) 현재 위치값이 위치 티칭값과 같은 스케일로 보이도록 수정, DCT요청사항 고정도
     현재 위치=현재 위치값(티칭값 기준) - 파렛오프셋값
2) 모니터링 변수 잘못 할당된 부분 수정
   0x2A3C Position_offset
   0x2A3E LMMT_Pallet_Offset (DF4,DF5) raw data

20230105
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.13 20230105
1) PHS 근접센서값을 DF2에서 읽어오도록 처리방법 변경(DCT요청사항)
   기존)ID 3이면 DF2, ID5이면 DF6에서 읽는데 RFID가 DF6에 할당되어서 PHS 신호가 보이지 않음.
2) 통신 SafeOP 변경시 서보오프 되지 않는 현상 수정
  EtherCAT Stack의 상태머신에서 서보온되지 않도록 수정한 부분 적용

20230104
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.12 20230104
1) 통신 SafeOP 변경시 서보오프 수행 (완전히 수정된건 않고 간헐적으로 서보온 현상 발생함)

20230102
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.11 20230102 속도음수발생 구간 개선
1) 속도음수구간 발생 수정
   -D8_M에서 나왔던 이슈를 D7에 적용함.
     상위 제어 속도 모드에서 부하 정지판단시 속도명령이 0에서 시작하도록 변경.
   -> In Positon만으로 부하정지판단 후 곧 이어 속도 명령 인가했을 때, 0이 아닌 속도 Feedback이  명령 생성에 영향을 미치는 것을 방지하기 위함.
   -> Ft 1.19로 정지판단을 위한 위치 흔들림 크기 폭 설정 가능 
      D8: Ft 1.19 Default값 20->5 변경
      D7: Ft 1.19 Default값 20->5 변경

20230102
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.10 MDM99: 20230102 확장 근접센서 동작 추가
1) 확장 근접센서 동작사양 최종
   엔코더A/B는 각각 2점의 IO 입력센서값을 받는다.
  Valid 신호와 상관없이 엔코더통신으로 받은 센서값을 상위로 전송한다.
   
20221228
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.09 MDM99: 20221216 20221228 DCT전달
1) 확장 근접센서 변수 타입변경
   - 0x2A9A DEFTYPE_INTEGER32->DEFTYPE_UNSIGNED32
   - 0x2A9A 하위 byte : 엔코더A, 상위 byte : 엔코더B에  비트 맵핑 

20221214
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.08 MDM99: 20221214
1) 자동속도위치모드 조건문 누락된 버그 수정
2) 싱크모션(sync motion) 모드 추가
   -Normal+CW 방향: EA의 rising 신호를 감지함
   -Normal+CCW 방향: EB의 rising 신호를 감지함
   -Inv+CW 방향: EB의 rising 신호를 감지함
   -Inv+CCW 방향: EA의 rising 신호를 감지함
   
   -Statae Machine
   LMS_SM_READY  =  60, //20221214  SyncMotion mode
   LMS_SM_INIT,
   LMS_SM_VEL,
   LMS_SM_POS,
   LMS_SM_STOP,


20221208
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.07 MDM99: 20221208 함수 모듈화 완료
1) pallet ID(사용자 파라메터 입력시, ft-1.50) 오프셋 검색되도록 변경
2) Z상 출력 기능 추가, LMS_EtherCAT_Z
3) 평균부하율 추가, MDE 150/151
4) 파렛 오픈시 적분게인 초기화, Iref_It
5) warning 제거
	extern void free(void *ptr);
	extern void *malloc(size_t size);
      	
20221206
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.06 MDM99: 20221206 함수 모듈화 시작버전
1) RSWare Oscilloscope Default Value 수정
   CH3/4 Current feedback/command
2) 모션 변경점, ft-1.56 설정에 따라 DCT special처리된 부분을 D8_M과 같이 동작사양으로 결정.
	1)속도준비상태에서 스텝기동시 현재 정지 위칭위치를 기준으로 함.
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
	2)속도기동 정지후 스텝이동시 현재 위치로 기준점 변경
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
	3)위치기동중 정지후 스텝이동시 현재 위치로 기준점 변경
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
		

20221130
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.05 MDM99: 20221130 (D8_M 99.22버전과 동기화시킴) 
1) EWR 음수처리 버그 수정
2) RFID 관련 수정사항
   -ID 검색기능 테이블 인텍스 잘못되는 버그 수정
   -ft-1.50 값에 따라 RFID 검색 기능 구분
   -RFID값 Valid ON 구간에만 적용하고 오픈상태는 0으로 초기화함.
4) 모니터링 변수/오브젝트 이름 변경
   0x2A38/0x2A96/0x2A97

20221012
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.04 MDM99: 20221012 외부 진입시 엔코더값 0에서 시작하는 버그를 수정함.
   ft-1.52 D0 disable일때만 재현됨.
     1.20.00.00까지 정상이나 1.20.10.01부터 잘못 수정된 부분을 바로 잡음.
     수정코드:
      if (Para[1][56].val != 0) {  // 20221011 위치값 0으로 시작하는 오류 수정, NOT DCT Encoder
         pos_fbk = 0; // init.
      }
1) 코드 동기화시킴: V1.02.95.10

20220902
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.03 MDM99: 20220902
1) 긴급정지 파라메터 정리
   ft-1.57, 0x2139 "LMMT_Emergency Stop Deceleration" Range:0 ~ 2147483647
   ft-1.58, 0x2140 "LMMT_Emergency Stop Torque Limit" Range:0 ~ 500 [%]
2) 속도 가감속 PDO 맵핑에 추가
   0x211A, 0x211B
   0x211D, 0x211E

20220830
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.02 MDM99: 20220830
   {/*Pr-1.57*/   0,    0, {0, 0,0,0,0,0,0,59}},//Range:0 ~ 2147483647            // LMS_Emergency Stop Deceleration, mm/s^2 //20220817 비상정지 관련 추가
   {/*Pr-1.58*/   0,    0, {0, 0,0,0,0,0,0,32}},//Range:0 ~ 500                   // LMS_Emergency Stop Torque Limit, %      //20220830 비상정지 관련 추가
OBJCONST UCHAR OBJMEM aName0x2139[] = "LMMT_Emergency Stop Deceleration"; //20220817 비상정지 관련 추가
OBJCONST UCHAR OBJMEM aName0x2140[] = "LMMT_Emergency Stop Torque Limit"; //20220830 비상정지 관련 추가


20220721
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.01 MDM99: 20220722
1) BackEMF UVW 신호 확인 및 Z pulse 확인 위한 Code추가

20220720
///////////////////////////////////////////////////////////////////////////////
0) V01.20.10.01 MDM 99 : 20220720 ET1100 시작버전에서 DCT향 신규기능 추가한 스페셜 버전
1) 긴급 감속정지 추가된 기능, (영우DSP 요청건)
  //TC0720_1 : 긴급정지 비활성화 상태에서는 알람 즉시 발생.
  //TC0720_2 : 긴급정지 활성화상태, 서보오프상태일때는 1sec 지연후 알람발램 발생
  //TC0720_3 : 긴급정지 활성화상태, 서보온 정지 혹은 구동상태에서 속도프로파일의 속도생셩 명령을 기준으로 감속정지2) 버전 3자리 표기가 되도록 수정한 빌드 실행파일을 사용함
  //TC0720_4 : 긴급정지동안 다른 명령은 처리지 않는다.
2) 엔코더 양간오차 보정 기능 추가
   LMMT_PLS_Diff값이 HalfTurn 값 보다 큰경우 LMMT_Diff_Signed값 수정
   현재위치 유지기능이 특정조건(A&B)구간에서 이상동작 현상 수정
   :DCT엔코더만 해당하고 RSA엔코더는 양간오차가 0 이므로 보정이 필요없다.
   :코드 통일을 위해 추가함.
3) 엔코더 홀신호 모니터링 변수 추가
   MDE 159, sI32 LMMT_enc_PSstatus; // enc PS status
   MDE 160, sI32 LMMT_enc_Halluvw; // enc hall uvw
4) 변수이름 변경
    enc_commflag -> enc_ce_alarmflag
    enc_resetState -> enc_normal_alarmreset
5) 토크모드(CST) 제어모드 관련 코드 추가후 주석처리(향후 사용하기 위해서)

20220404
///////////////////////////////////////////////////////////////////////////////
0) V01.20.00.00 MDM 99 : 20220404
1) 제조 생산버전

20220404
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.19 MDM 99 : 20220404
1) DSP PINMUX 설정시 IPcore/ET1100보드를 구분하여 분기처리함.
2) MDM 90/91/92추가
	90 : ecat state machine
	91 : cia402 state machine
	92 : Al Status Codes : 0x134:135h

20220331
///////////////////////////////////////////////////////////////////////////////
0)  V01.10.90.18 MDM 99 : 20220331
1) 전류제어기 입력용 DC Bus 전압 레벨(nVdc_flt)을 190V->250V 로 복구
   수정이유: 연산시 overflow 가능성을 차단하기 위해서 변경
2) 0x2A50 product revision 에 보이는 문자열 변경
             H/W        0x2A49(OS Version)   0x2A50(Product Revision)
   -----------------------------------------------------------------------------------------
   IPcore   Ser.A(FPGA) Rev.A OS20           Rev.A                      gControlBoardRev(1)
            Ser.A(ASIC) Rev.B OS20           Rev.B                      gControlBoardRev(2)
            Ser.B       Rev.B OS20           Series.B                   gControlBoardRev(2)
   -----------------------------------------------------------------------------------------
   ET1100   Ser.A       Rev.B OS20           Ser.A Rev.D                gCtrlBdRev(4)
            Ser.B       Rev.B OS20           Ser.B Rev.D                gCtrlBdRev(4)

20220328
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.17 MDM99: 20220328
1) 드라이브 리셋에 의한 오동작 방지를 위해 bRunApplication 변수 초기화함.
2) MDM 16/17/18 U,V,W상전류 모니터링 기능에 패스워드 입력시 엔코더 동작변수 모니터링 되도록 수정
   U : flag_encoder_reset
   V : LMMT_Enc_state
   W : LMS_TorqueLimit_readyFlag
3) PDO상 위치값 0으로 보이는 현상 수정
   V01.10.90.16에서 수정한 내용이 PDO 오브젝트에 반영안되는 버그
4) 서보 알람하고 리셋 동작시 현재 위치값 유지 기능 수정
   ft-1.52 D0 Incremental Pos Latch Enable 기능 수정
   테스트용(삼익/가온) 추가된 코드를 DCT FW와 통합하면서 DCT의 요청사항으로 수정함.
	0 (disable): 알람리셋 이후 현재 위치값은 싱글턴으로 설정
	1 (enable) : 알람리셋 이후 현재 위치값 유지
	             (단 엔코더알람 발생한 경우 disable 동작과 같이 싱글턴으로 설정)  

20220328
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.16 MDM99: 20220324
1) Readme.txt 엔코딩 변경: euc-kr -> utf-8
2) 소스변경 없음

20220324
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.16 MDM99: 20220324
1) SDO상 0x1000번대 오브젝트에 대한 데이터 밀림 현상 수정
   원인: 1.10.90.12의 수정사항이 0x1000번대 오브젝트에 영향을 줌
   수정: temp_xx로 대체된 오브젝트만 특수처리함.
2) 0x6063 PositionActualInternalValue 값이 0으로 모니터링되는 오류 수정
   temp_objPositionActualInternalValue 변수에 실제 MotorPosition2 변수를 할당함.

20220317, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.15 MDM99: 20220317
1) E.210 에러 발생 시 RSWare 연결 안 되는 문제 수정(이 문제는 다른 모델에서는 발생되지 않고 DCT 모델에서만 발생함)
   - DCT는 RSWare연결이 안되어 발생되지는 않으나 잠재적으로 내재된 RSWare 연결 시 DISPLAY가 깜빡이는 문제또한 함께 수정됨
   - 원인: ESC의 PDI 설정이상으로 ECAT 관련 코드 처리에 부하가 발생하여 RSWare 통신처리(및 DISPLAY 처리)가 안되어 문제 발생
          ESC register에 값을 쓰고 확인하는 코드를 실행하게 되는데, 무한 do/while문을 사용하고 있어서에서 정상 동작을 보장하지 못한다.
          DCT모델의 경우에는 HW_DisableSyncManChannel()함수에서 빠져나오지 못하여 BackGround에서 RSWare Protocol 처리를 
                 해주지 못해 통신이 안되었던 것으로 파악되었다. 다른 모델들은 정확한 지연 사유는 파악하지 못했지만 유사할 것으로 보인다.
   - 조치: E.210 에러 발생 시 불필요하게 실행되는 ECAT 관련 코드들을 실행하지 않도록 bRunApplication을 FALSE로 유지하도록 변경함
   
20220317
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.14 MDM99: 20220317
1) 엔코더분해능 변수 잘못 계산되는 현상 수정. 
   FullTurn 계산 이후 엔코더 마스킹 적용하도록 위치 이동.

20220316
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.13 MDM99: 20220316
1) 엔코더분해능 변수 0x1FFF(=8192d) 기준으로 재정의하여 분기처리함.
   오류: DCT모터와 엔코더(10um) 사용시 위치모드 구동할때 EncMask에 따른 위치모드 전환시점이 1um 사용 조건으로 
        잘못 인식되어 목표위치 25000 카운터 이전에 위치모드로 구동되면서 서서히 감속하는 현상발생.
   수정: EncMask==0x1FFF 조건식을 만족할 수 있도록 EncMask값을 재정의함.
   DCT모터사양시 엔코더 분해능은 ft-1.21 값에 따라 자동 선택된다.
   ft-1.21 : 0 (1um)			위치모드 전환시점: 25000 pls
             1 (0.97656 um)     위치모드 전환시점: 25000 pls
             others (10um)      위치모드 전환시점: 2500 pls

20220315
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.12 MDM99: 20220315
1) SDO, RSWare PDR 32비트 엑세스 코드누락된 변수 최종정리 (DCT는 PDR 명령없음)
   //    DP_objPositionActualInternalValue(0) = temp_objPositionActualInternalValue; // 0x6063
   //    DP_objPositionActualValue(0)         = temp_objPositionActualValue;         // 0x6064
   //    DP_objVelocityActualValue(0)         = temp_objVelocityActualValue;         // 0x606C
   //    DP_objTouchProbePos1PosValue(0)      = temp_objTouchProbePos1PosValue;      // 0x60BA
   //    DP_objTouchProbePos1NegValue(0)      = temp_objTouchProbePos1NegValue;      // 0x60BB
   //    DP_objTouchProbePos2PosValue(0)      = temp_objTouchProbePos2PosValue;      // 0x60BC
   //    DP_objTouchProbePos2NegValue(0)      = temp_objTouchProbePos2NegValue;      // 0x60BD
   //    DP_objFollowingErrorActualValue(0)   = temp_objFollowingErrorActualValue;   // 0x60F4
   //    DP_objPhysicalInputs(0)              = temp_objPhysicalInputs;              // 0x60FD
2) FoE 동안 Segment 수정
   DisplayState()
   
20220310
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.11 MDM99: 20220310
1) 로터리모터 사용시 CSP모드에서 CW운전후 서보오프하고 다시 서보온할때 모터 틔는 현상
   양산버전 V 1.02.00.01에서는 초기화하였고 로터리 구동 정상
   스페설버전 1.02.72.01 이후 모든 버전: 현재 위치 초기화를 주석처리함으로써 내부에 E.19 알람을 발생시킴.
   운전모드에 따라서 구분할 수 있도록 수정함.
      로터리모터 운전: 초기화함
      LMMT 운전시 : 초기화하지 않음.(근거 불분명함)

20220310
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.10 MDM99: 20220303
1) FoE Download 시 Version Check 는 Application FW 만 검사함.
2) DCT 는 OS20 으로 되어 있어 변경하지 않음

20220303
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.09 -> V01.10.90.10 MDM99: 20220303
1) MDM11 모니터링값에 서보알람을 보여지는 버그 수정
   원래대로 DC-Link voltage 로 보여지도록 코드 원복(스페셜 적용되었던 내용임)
2) MDM 16/17/18 U,V,W상전류 모니터링 기능 원래 코드로 원복
3) 이동평균 필터 탭수 98999 이상 설정시 부팅안되는 오류 발생-->디버깅중
   FW에서 탭수를 98990으로 설정하도록 해서 사용하도록 함.(IPcore 버전: V 1.02.95.02)
   ET1100 프로젝트는 100000L로 설정해도 정상동작함 (ET1100버전: V1.10.90.10)

20220228
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.07 -> V01.10.90.09 MDM99: 20220228
   09:엔코더 통합버전 08:(skip version)
1) 시간부족으로 RSware 끊기는 현상이 가끔 발생하여 로깅함수 삭제함.
   ->함수 자체를 실행하지 않도록 수정함.
2) 메인함수를 DRAM으로 이동하여 IRAM 메모리 확보
      Main.obj(.text) /* 20220225 */
3) DCT 기능과 일반 LMT 기능을 구분하기 위한 파라미터 추가
   배경: DCT 엔코더와 RSA 엔코더는 동작방식이 다르다. 따라서 홀엔코더 선택할 수 있도록 파라메터로 처리함.
   ft-1.56 0x2138	LMS_Hall Encoder ID Setting	LMS 홀엔코더 ID 설정
   	0: DCT Encoder
   	others : General(0: DCT, 1:GAON, 2: RSA/PBA)

   *엔코더 H/W 버전 읽기 지원
   Encoder Type(H/W) ft-1.56 gEncVer
   DCT		     0       x
   Gaon		     1 	    x
   RSA		     2       엔코더버전(Hword:fullturn,  Lword:FW정보), 0x11940101

4) 콜백변수 32비트 확장, save의 옵션기능을 추가하기 위해 양산버전에 도입한 것을 적용함.
   load의 옵션기능은 추후에 적용하도록 해야 한다.
   uI32  flgCallBack;   //20220225 //20140915
   예시) flgCallBack = 0x1010; --> flgCallBack = 0x801010xx;

5) SDO 사용하여 파라메터 저장시 RSware 먹통되는 현상 수정
   원인: 파렛오프셋을 저장할때 중복 플래쉬쓰기가 발생하여 FlashBlockPtr가 초기화가 안됨.
   	    CheckFlashStatus 에서 FlashBlockPt 14번 영역에 대해서 해제되도록 함.
   해결: 서버 파라메터와 파렛오프셋을 구분하여 저장함.
     0x1010:03 "save" 명령시 SaveEncoderOffset 호출, flgCallBack = 0x80101003
     0x1010:04 "save" 명령시 FlashBackup 호출, flgCallBack = 0x80101004
6) LMS용 부하율 시간 설정 기능 추가, LMS 이동평균 부하율 시간설정
   평균부하율에 대한 설정 시간동안의 이동평균값을 계산함.
   내부 버퍼크기 : 99999+1 로 설정
   ft-1.55 0x2137 LMMT_Current Load Moving Avg Time Setting, 0~99999, 단위:sec


20220221
///////////////////////////////////////////////////////////////////////////////
0) V01.10.90.07 -> V01.10.90.08 MDM99: 20220221
   8:(skip version)
1) 08버전은 LMS FW DCT버전과 일반버전의 분기점 관리를 위해 SKIP함.

20220216, 이현규
///////////////////////////////////////////////////////////////////////////////
0) V01.10.00.06 -> V01.10.00.07 MDM99: 20220216
1) 비정상 EEPROM 파일(예를들어 F/W 파일) 다운로드 시 F/W가 무한루프에 빠지는 문제 수정
  - 증상재현: EEPROM에 F/W 파일을 다운로드 하는 경우 F/W가 무한루프에 빠짐
    -> 이 경우 TwinCAT으로 재스캔 하여 정상 EEPROM을 다운로드 하면 정상 상태로 복귀 가능함(MMCE는 안됨)
    -> V01.02.100.21 이하 버전에 있던 문제는 V01.10.90.01를 통해서 수정되었다. 이후에 변수를 로컬화 하면서 다른 무한루프 문제가 발생한 것임.
  - 원인: HW_Init()에서 로컬변수 ms_eeprom_delay 관련 코드가 컴파일러에 의해서 최적화 되면서 의도하지 않은 동작을 함에따라  while 루프를 탈출하지 못하는 문제가 발생함
  - 조치: 최적화 방지를 위해서 HWTimer0_CounterUp 에 volatile 추가함
    -> ms_eeprom_delay 보다는 값이 계속 변경되는 HWTimer0_CounterUp 타입을 변경하는 것을 권장한다는 의견이 있었음(김상오팀장리뷰의견)
    -> HWTimer0_CounterUp 나 ms_eeprom_delay 중 어느것을 수정하더라도 같은 어셈블리 코드가 나옴
    -> 조치 후 E.210 에러 발생 확인함(PDI init error: "ETPDI")

20220214
///////////////////////////////////////////////////////////////////////////////
0) V01.10.00.05 -> V01.10.00.06 MDM99: 20220214
1) ET1100용 FW 버전이 업그레이드 안되는 현상 수정
   FW호환을 위해 설정한 정의문에 오타발생이 원임.
   PERMITTED_VERSION       0x011000000 -> 0x01100000  //20220118 ET1100 1.10.0.0

20220211
///////////////////////////////////////////////////////////////////////////////
0) V01.10.00.04 -> V01.10.00.05 MDM99: 20220211
1) FoE 다운로드시 Application FW 만 Version Check 하도록 수정

20220209, 천병훈
///////////////////////////////////////////////////////////////////////////////
0) 1.10.90.03 -> 1.10.90.04 MDM99: 20220209
1) 로터리모터 구동시 E.005발생 오류 수정
   전류피드백 스케일 2배 확장을 위해 추가한 변경 내용중 Gain_setup 함수 변경점이 적용되지 않았음.
   옵션을 넣어서 스케일 범위를 선택하여 컴파일되도록 변경(기본값 0)
   #define CC_SCALE_13BIT           0 //20220208 1:13bit, 0:12bit(default)
   CC_SCALE_13BIT 값이 1일때 13비트 분해능 동작시 정상동작
   CC_SCALE_13BIT 값이 0일때 12비트 분해능 동작시 정상동작
2) FOE 다운로드 받은 데이터 변수 이름 수정
   gD16 u16 -> u16Dat
3) Rsware 다운시 버전 변수를 static으로 변경
   static gD32 Dat32; //20220209

20220208, 천병훈
///////////////////////////////////////////////////////////////////////////////
0) 1.10.90.02 -> 1.10.90.03 MDM99: 20220208
1) FOE 다운로드 오류 수정

20220207
///////////////////////////////////////////////////////////////////////////////
0)1.10.90.01 -> 1.10.90.02 MDM99: 20220207
시리즈B 다운로드 방지 기능
1) 외부진입(ENC_NONE)시 위치래치 기능에 따른 pos_fbk값 설정 오류 수정
   - DCT엔코더 사용시 ft-1.52 D0 enable해서 사용하면 안된다. 주의!!
   RS엔코더 사용시 위치값 0 에서 시작
   DCT엔코더 사용시 싱글턴으로 시작
2) 서보알람 리셋시 오브젝트 0x603F 클리어 되도록 수정
   Run08_AlarmReset함수내에서 아래 코드로 처리함
   CiA402_LocalError(0);        /* 2021.09.17 */
3) ET1100 수정사항 모두 반영
   -V2.0이상만 ET1100 HW에 다운로드 가능하도록 함.
4) 시리즈A에 FW 공용 사용을 위해 패스워드입력시 다운로드 가능하도록 변경
   DCT FW 1.20 -> ET1100 양산모델에 FOE를 사용하여 다운로드할 때 필요함.
   검색어: 20220207

20220203
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.25 -> 1.10.90.01 MDM99: 20220203
  ET1100용 20220128 수정사항 반영
1) ET1100 사용 ControlBoard 지원을 위한 Hardware version 추가 수정
  - 0x1009 Hardware version 정보를 '0.0.5' 로 표기하도록 추가
  - 0x1009 Hardware version 사양정리
    ------------------------------------------------------------------------------
               Hardware       gControlBoardRev(=ASIC_VERSION)           bESC_TYPE
               version        (MDM 74)
    ------------------------------------------------------------------------------
    Ser.A      0.0.2          1 ( <> 0x20151124/FPGA Motion)            Don't Care
    Ser.A,B    0.0.3 (Rev.B)  2 ( =  0x20151124/ASIC Motion)            1(IP Core)
    Ser.A      0.0.4 (Rev.C)  2 ( Full Closed Function )
    Ser.B      0.0.3 (Rev.A)  2
    Ser.A      0.0.5 (Rev.D)  4 ( =  0x20151124/ASIC Motion, ET1100)    0(ET1100)
    Ser.B      0.0.5 (Rev.D)  4 ( =  0x20151124/ASIC Motion, ET1100)    0(ET1100)
    ------------------------------------------------------------------------------
    * 0.0.1 : Qual Hardware
    * 0.0.2 : 양산 Hardware
    * 0.0.3 : Motion ASIC
    * 0.0.4 : Full Closed Model, 사용안함
    * 0.0.5 : ET1100


20220128, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.15.90.00 -> 2.15.90.01 MDM99: 20220128
1) EEPROM_LOADED 신호 상태 체크 위치 수정: ECAT_Init() -> HW_Init()
  - ECAT_Init() 보다 앞서 실행되는 HW_Init()함수에서 ESC 접근이 시도되어 위치를 이동함.
  - Register 0x0000  체크 구문은 삭제함 (Register 0x0140 체크하는 구문이 이미 있음)
  - ms_eeprom_delay는 지역변수로 수정(에러 코드를 추가했고, 스코프 측정결과 범위내에 항상 들어오는 것을 확인함)
2) HW_Init()함수에서 PDI Control (0x0140) 데이터가 이상할 경우 무한루프를 돌게 되어있어 탈출 코드 추가
  - do..while 문을 200ms 이상 초과 시 탈출
3) ESC의 PDI 초기화에 문제가 생긴 경우 에러 발생 추가
  - PDI init error: "ETPDI", E.210
  - #define ErrCode_ET_PDI          0x9A  // E.210 ETPDI  // PDI init error.
  - ECAT PDI 초기화 에러가 발생된 경우에는 SetALStatus()에서 에러를 클리어 하지 않도록 수정

XML Maker
1) V2.14.0 파일로 업데이트함(20210819 Release by 천병훈수석)
  - 수정한것은 아니고 코드에는 최신버전으로 통합이 안되어 있어서 적용 하였음
  - RSA_CSD7_REV2_V2140_RELEASED_20210819.xml

20220119
///////////////////////////////////////////////////////////////////////////////
0) 2.15.90.00 -> 2.15.90.00 MDM99: 20220119
1) 1/19 이현규 책임 코드 리뷰 적용
3) 롬 파일 생산 전달

20220118
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.05 -> 2.15.90.00 MDM99: 20220118
1) ET1100 사용 ControlBoard 지원을 위한 Hardware version 추가 수정
  - 0x1009 Hardware version 정보를 '0.0.5' 로 표기하도록 추가
  - 0x1009 Hardware version 사양정리
    ------------------------------------------------------------------------------
               Hardware       gControlBoardRev(=ASIC_VERSION)           bESC_TYPE
               version        (MDM 74)
    ------------------------------------------------------------------------------
    Ser.A      0.0.2          1 ( <> 0x20151124/FPGA Motion)            Don't Care
    Ser.A,B    0.0.3 (Rev.B)  2 ( =  0x20151124/ASIC Motion)            1(IP Core)
    Ser.A      0.0.4 (Rev.C)  2 ( Full Closed Function )
    Ser.B      0.0.3 (Rev.A)  2
    Ser.A      0.0.5 (Rev.D)  4 ( =  0x20151124/ASIC Motion, ET1100)    0(ET1100)
    Ser.B      0.0.5 (Rev.D)  4 ( =  0x20151124/ASIC Motion, ET1100)    0(ET1100)
    ------------------------------------------------------------------------------
    * 0.0.1 : Qual Hardware
    * 0.0.2 : 양산 Hardware
    * 0.0.3 : Motion ASIC
    * 0.0.4 : Full Closed Model, 사용안함
    * 0.0.5 : ET1100
2) ET1100 보드 이전 FW 버젼 다운로드 방지 기능
   이전 FW 버젼 정의 < V02.15.00.00
   RSware 에서는 Ft-0.08 = 777 입력시 버젼 체크하지 않음

20220114, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.04 -> 2.14.00.05 MDM99: 20220114
1) ET1100 사용 ControlBoard 지원을 위한 PDI 상태 확인(EEPROM_LOADED, ESC Type) 기능 추가
  - GPIO 설정 변경: GP5[9]=EEPROM_LOADED_H -> INPUT
  - EEPROM_LOADED 신호 HIGH 상태까지 대기 코드 추가            -> ECAT_Init()
    -> EEPROM_LOADED: ET1100이 EEPROM의 PDI 설정 데이터를 로딩하고 PDI 사용이 가능한 생태
    -> 최대 200ms 까지 대기
  - ESC에서 ESC Type이 ET1100으로 읽히는지 확인하는 코드 추가 -> ECAT_Init()
    -> EEPROM_LOADED 신호와 함께 ET1100 PDI가 잘 되는지 확인하기 위한 목적으로 추가함
  - 위의 두 경우 에러는 띄우지 않고 ms_eeprom_delay 변수로 상태를 확인 가능
    -> 0x80000000: EEPROM_LOADED 신호 이미 들어와 있음 (Best)
    -> 0xX8000000: Register 0x0000 값 이상
    -> 0x0XNNNNNN: 지연 발생시간 NNNNNN
  - UpdateEEPROMLoadedState() 무용성에 대한 검토
    -> 레지스터 0x0502의 데이터를 검증하는 방식으로 되어있으나 0일때 정상으로 판단 하므로(PDI 가 정상동작 하지 않을 때) 정상동작으로 오판할 가능성이 있음
  - 테스트 결과 EEPROM_LOADED 상태를 체크하면 항상 HIGH가 되어있음을 확인함
    -> ms_eeprom_delay 어드레스 0x11837b0c를 OBJ 0x5011 Monitor Address에 쓴 뒤 0x5012 Monitor_A 값으로 확인함

20220113, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.03 -> 2.14.00.04 MDM99: 20220113
1) ET1100 사용 ControlBoard 지원을 위한 Hardware version 추가
  - 0x1009 Hardware version 정보를 '0.0.5' 로 표기하도록 추가
  - 0x1009 Hardware version 사양정리
    -----------------------------------------------------------------------
    Hardware version      gControlBoardRev(=ASIC_VERSION)       bESC_TYPE
    -----------------------------------------------------------------------
        0.0.2             1 ( <> 0x20151124/FPGA Motion)        Don't Care
        0.0.3             2 ( =  0x20151124/ASIC Motion)        1(IP Core)
        0.0.5             2 ( =  0x20151124/ASIC Motion)        0(ET1100)
    -----------------------------------------------------------------------
    * 0.0.4 : 사용안함, Full Closed Model 용


20220111, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.02 -> 2.14.00.03 MDM99: 20220111
1) ET1100 사용 ControlBoard 지원을 위한 코드 추가
  - GPIO 설정 추가
    -> GP3[6] - IN : ET1100(0) / IPCORE(1)
    -> GP3[15]- OUT: RUN LED
    -> GP4[10]- OUT: ERR LED
  - ET1100/IPCORE 인식 pin check 구문 추가(CheckEscType()로 읽어서 전역변수 bESC_TYPE 에 저장)
  - RUN/ERR LED 출력기능 분기처리 추가: ET1100의 경우 GPIO로 바로 출력
  - ESC 타입에 따라서 Device Name(0x1008)값을 처리하도록 변경

2) motion OBJ 임시저장 변수 temp_XXXX(2.14.00.01/02 참고) 초기화 구문 추가
  - InitTempMotionVariables() 함수를 추가하여 초기화함
  - Soft Reset을 하는 경우 temp_XXXX가 초기화 될 수 있도록 main() 내에서 초기화 하도록 함.

3) ft-3.20 (Following Error Limit) 기본값 변경 (V2.14.60.03, 천병훈)
  - 기존: 2147483647L -> 변경: 41943040L
  - 23비트 분해능 기준 5턴: 41943040(=2^23x5) (기존2.14에서 2147483647(=2^31))

4) 오브젝트 0x3000번대 동작 사양 변경 (V2.14.60.03, 천병훈)
  - 입력 범위 문제 수정: 쓰는 데이터의 값과 무관하게 해당 동작이 수행되는 문제 수정 -> min(0)/max(1) 속성 값 추가
    -> 범위변경 - 기존 : 0 ~ 2(모두 동작) -> 변경: 0(동작안함, 쓰기만 가능) ~ 1(동작)
    -> 문자열(예, ABCD)로 입력시 해당 오브젝트의 기능이 수행되는 현상 수정
    -> 0X3001 숫자 2이상 입력되는 오류, 문자열(예, ABCD)로 입력시 AT 수행되는 현상 수정
    -> 0 을 넣어도 enable로 동작하는 현상 개선
  - write 하는 값이 오브젝트에 저장되지는 않음(기존 사양)
  - EtherCAT Write 동작 변동사항
    1. 0x3001  Auto Tuning
       0x3009  Fault History Clear
       0x300A  Absolute Encoder Multi Turn Clear
       0x3010  Drive Reboot
      -> 변경 전 [입력 범위: 0~2]
        --> 0~2: 동작O
        --> 이외:0xee22220000에러
      -> 변경 후 [입력 범위: 0~1]
        --> 0: 에러X, 동작X
        --> 1: 동작O
        --> 이외:0xee22220000에러
    2. 0x3002  Smart Tuning [0~2]
      -> 변동사항 없음 [입력 범위: 0~2]
        --> 0: 에러X, 동작X
        --> 1: 동작O
        --> 2: 동작정지
        --> 이외:0xee22220000에러

5) CTT 테스트 오류 수정 (V2.0.42.0 기준)
  - 0x60B2 에러 수정
    -> 에러 내용 - "Online Dictionary: 0x60B2 PdoMapping flag  'None' expected: 'R'."
    -> 수정사항 - OBJACCESS_RXPDOMAPPING 속성 추가
  - 다음 문제들은 추후 XML이 수정되어야 할 문제들임 (V2.2.1.0 기준)
    -> DT10F3
      --> 에러 내용 - Failed to convert '00' to BOOLEAN: Failed to convert '00' to an Bool"
                   Slave Scan 시 인식오류 발생 + Tool Log Log error
      --> 방안 - DT10F3:04 의 New Message Available의 <DefaultValue> #x00 -> #0x0 로 수정필요
    -> 0x1008:0
      --> 에러 내용 - offline: 0x1008:0 length of default data '10' expected: '20' (all bytes shall be defined in the defaultdata)
      --> 방안 - <DefaultString>CSD7_01BN1</DefaultString> -> <DefaultString /> 로 수정필요
    -> 0x100A:0
      --> 에러 내용 - offline: 0x100A:0 length of default data '4' expected: '20' (all bytes shall be defined in the defaultdata)
      --> 방안 - <DefaultString>V1.0</DefaultString> -> <DefaultString /> 로 수정필요


20220107, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.01 -> 2.14.00.02 MDM99: 20220107
1) OBJ 0x606C PDO/SDO/RSWare read 문제 추가 수정 -> 2.14.00.03에서 보완
  - 2.14.00.01에서 temp 적용한 오브젝트를 내부 제어에 사용하는 경우 DP_XXXX를 temp_XXXX로 변경
    -> temp_objPositionActualValue         // 0x6064  - 12 points (PP, InitParaofTAS, command_SVR, etTouchProbe_task)
    -> temp_objVelocityActualValue         // 0x606C  -  1 points (PP)
    -> temp_objTouchProbePos1PosValue      // 0x60BA  -  1 points (SelectDisplayTouch)
    -> temp_objTouchProbePos1NegValue      // 0x60BB  -  1 points (SelectDisplayTouch)
    -> temp_objTouchProbePos2PosValue      // 0x60BC  -  1 points (SelectDisplayTouch)
    -> temp_objTouchProbePos2NegValue      // 0x60BD  -  1 points (SelectDisplayTouch)
    -> temp_objFollowingErrorActualValue   // 0x60F4  -  x points
    -> temp_objPhysicalInputs              // 0x60FD  -  1 points (PP)
  - pp_task() 함수 내 UpdateMotionObjects() 함수 실행 제거 (위 수정 조치로 인하여 불필요해짐)
    -> temp_objPositionActualValue 로 대체함

20220106, 이현규
///////////////////////////////////////////////////////////////////////////////
0) 2.14.00.00 -> 2.14.00.01 MDM99: 20220106
1) OBJ 0x606C PDO/SDO/RSWare read 문제 수정 -> 2.14.00.02/03에서 보완
  - 문제점: 32bit 데이터 인 0x606C "Velocity Actual Value"를 16바이트씩 업데이트하던 도중
    INT_Inner_Routine()이 실행되면서 old/new 16bit가 혼재하는 상황으로 인하여 데이터 이상 발생
       예를 들어 0x0000 0001 ~ 0xFFFF FFFF 값인 상황에서 0x0000 FFFF 나 0xFFFF 0000 같이 데이터가 읽히는 문제
  - 해결 방법: INT_Inner_Routine()에서 OBJ 값을 업데이트 하던것을 EtherCAT 처리 타이밍에 업데이트 하도록 수정
  - 업데이트 함수 UpdateMotionObjects() 실행 타이밍
    -> PDO Access: APPL_InputMapping() 호출 전 실행
    -> SDO/RSWare Access: OBJ_Read() 함수 내에서 맨처음 실행
    -> PP모드에서 사용되는 오브젝트 업데이트: pp_task() 함수 내에서 맨처음 실행 -> 2.14.00.02에서 제거함
  - 적용 대상 오브젝트 (0x6000 영역, RO, 32bit 데이터, TxPDO Mapping 가능한 오브젝트)
    -------------------------------------------
      Index   |  Index_Name
    -------------------------------------------
      0x6064  |  Position Actual Value
      0x606C  |  Velocity Actual Value
      0x6079  |  DC Link Circuit Voltage
      0x60BA  |  Touch Probe Pos1 Pos Value
      0x60BB  |  Touch Probe Pos1 Neg Value
      0x60BC  |  Touch Probe Pos2 Pos Value
      0x60BD  |  Touch Probe Pos2 Neg Value
      0x60F4  |  Following Error Actual Value
      0x60FD  |  Digital Inputs
    -------------------------------------------


20220128
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.24 -> 1.02.100.25 MDM99: 20220128
1) 평균부하율 계산식 오류 수정
  ft-4.23 입력값 단위를 초단위에서 ms 단위로 변경, 기본값은 10ms
  0x2417 오브젝트 범위수정
  DEFTYPE_UNSIGNED8  -> DEFTYPE_UNSIGNED16
  0 ~ 60000 [ms]
2) 0x60B2 PDO 맵핑 추가
3) 위치티칭값 변경시 속도와 가감속 즉시 변경 적용 (PDO 통신사용시)
4) FOE를 이용해서 FW 변경시 세그멘트상에 GO/DONE 표시됨(사용자 편이성)


20220126
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.23 -> 1.02.100.24 MDM99: 20220126
1) 100.23버전에서 통신끊김 발생하여 코드(100.21에서 ET1100수정사항만) 원복함.
2) 위치 Override 기능
   티칭 위치가 변경될 때 속도/가감속 값도 변경되어 적용
   LM_STATUS : 10(위치준비), 위치속도준비(50), 스텝초기(40)
3) 위치 등속도 이동중에는 변경되지 않음-> 등속 이동 중에도 변경요청(TBD)


20220126
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.23 -> 1.02.100.24 MDM99: 20220126
1) 100.23버전에서 통신끊김 발생하여 코드(100.21에서 ET1100수정사항만) 원복함.
2) 위치 Override 기능
   티칭 위치가 변경될 때 속도/가감속 값도 변경되어 적용
   LM_STATUS : 10(위치준비), 위치속도준비(50), 스텝초기(40)


20220125
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.22 -> 1.02.100.23 MDM99: 20220125
  --->>서보온 유지시 통신끊김 E.203 자주 발생하여 Timer0인터럽트 수정내용 코드 원복함!!!
1) 타이머 인터럽트(Timer0) 수정 (이전에는 값이 0이었음)
	기존: #define MACRO_INT_TIMER           (0x0000)
	변경: #define MACRO_INT_TIMER           ( MACRO_INT_TIMER0 | MACRO_INT_TIMER1 )
	다음 함수에 영향을 준다.
		INT_2ms_Routine, usb_isr
2) 하드웨어 버전정보 변경
   DCT모델의 제어보드 rev2이고 IPcore일 때 양산버전(0.0.3)과 달라서 동일하게 변경함(검사기 셋업시 주의!)
   FW 1.02.100.22 이후버전을 사용시 0.0.3으로 표시됨.
	기존: 0.0.2
	변경: 0.0.3
3) 실시간 위치 override 기능 (DCT 김희철 전임 요청사항)
   위치 티칭값 이동 중 티칭값을 바꾸었을 때 위치와 속도와 가감속 값이 모두 변경되어 적용되도록 수정.

20220124
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.21 -> 1.03.90.02  MDM99: 20220124
0)1.02.100.21 -> 1.02.100.22 MDM99: 20220124
1) ET1100 사용 ControlBoard 지원을 위한 Hardware version 추가 수정


20220117
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.20 -> 1.02.100.21 MDM99: 20220117
1) 파리미터 이름과 기능(최종)
   ft-1.52 D0 Incremental Pos Latch Enable
   ft-1.52 D1 Servo PalletOffset Apply Enable
   ft-1.52 D2 User Home Position Enable
   ft-1.52 D3 User Position Start Enable
2)파렛진입시 임의의 좌표로 시작하는 기능
   0x2135   ft-1.53 LMMT_Positive StartOffset
   0x2136   ft-1.54 LMMT_Negative StartOffset
2) 파렛모니터링 변수 읽는 명령 추가
   #ERD*
   $ERD20&0&301&-500&0&:


2)RFID로 검색된 파렛오프셋값을 읽는 코드 추가
   테이블 3자리 번호에 000을 입력하면 Pallet Offset Monitor(0x4000)의 값을 보여준다.
   ERD000*

20220110
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.19 -> 1.02.100.20 MDM99: 20220110
1) 100.18버전 시간부족으로 모터구동 안되는 경우가 발생함
   ->LMS_SimpleLogging 실행안되도록 return 처리함.
   ->MotionHistory_UnitTEST 삭제
2) 파렛오프셋 검색 기능 구현
   다음 파라미터 기능 변경
   0x2132   ft-1.50 , DINT, R : RFID 테스트변수
   0x2133   ft-1.51 , DINT, R : 홈밍 완료시 사용자 홈위치값
   0x2134   ft-1.52 , D1 : disable(기존), enable: valid시점에 검색
                      D3 : test : 홈밍 완료시 사용자 홈위치값

20220106
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.18 -> 1.02.100.19 MDM99: 20220106
1)RFID를 사용할지 말지 결정하는 플래그 추가
   ft-1.52 D1 Pallet offset Apply Enable
      0 (default): 기존처럼 DF6 적용
      1 : MOSI.14 (RFID Scan Enable) 동작시 DF6의 값을 파렛테이블에서 찾는 기능
          ft-1.22=1일때 Position_Offset 변수에 파렛테이블로부터 검색한 EA 혹은 EB 오프셋을 설정한다.
   ft-1.52 D3 Pallet offset Test Enable
   :RFID 리더기가 없을때 기능동작을 확인하기 위한 유닛테스트 활성화.
      ft-1.52 D1=0이고 ft-1.51=1일때 동작함.

   주의사항: Para[1][51].val==1 로 설정시 서보에서 ID 1234로 테이블 300번째 검색하는 기능(디버깅용)
      if (Para[1][52].val & 0x00F0) { //         if (Para[1][51].val == 1) { /* pallet table searching */
         if (LMMT_RFID_SF) { // 0이 아닐때만 update.
            Get_PalletOffset(LMMT_RFID_SF);
         }
      }
      else { //20211230 for TEST
         if (Para[1][51].val == 1) {
            Get_PalletOffset(1234); // pallet offset 300에 가상의 데이터 입력된 상태 (오프셋이 정상동작하는지 확인하기 위한 용도)
         }
      }
20211230
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.17 -> 1.02.100.18 MDM99: 20211230
1)RFID값을 확인하여 파렛 offset을 적용하는 기능
   엔코더는 RFID를 DF6에 1byte로 전송
   서보는 파렛저장 테이블에서 RFID에 해당하는 파렛과 오프셋을 찾음.
   (palletID 찾는 함수는 매번 호출되면 시스템 부하가 발생한다. MOSI.14 rising edge로 처리하도록 함.)
     ->MOSI.14: RFID scan 명령
     ->MISO.14: 파렛 오프셋이 테이블에 있는 경우 1(set), 없는 경우 0 (not configured)
     ->ID는 진입전에 검색하기 때문에 Enc VALID조건과 무관하게 동작해야 한다.(TBD)
   FW는 RFID에 할당된 오프셋을 찾고 Para[1][22].val==1 일때 할당된 오프셋을 위치명령에 적용한다.

   주의사항: Para[1][51].val==1 로 설정시 서보에서 ID 12345678로 테이블 검색하는 기능(디버깅용)

   -------- 테스트 방법--------------
   1) 엔코더 오프셋 테이블 입력 (RSware 시리얼 통신명령 사용)
      1-1)파렛테이블 입력
         파렛1 설정 (ID: 1111, CW offset:100, CCW_offset:-200)
         파렛2 설정 (ID: 2222, CW offset:300, CCW_offset:-500)
         #EWR001%1111&0&100&-200&0&0
         #EWR002%2222&0&300&-500&0&0
      1-2)플래쉬에 저장하기
         #EWR001*
      1-3)테이블 0으로 초기화 하기
         #EWR001@
      1-4)플래쉬에 저장된 값 읽기
         #EWR001$

   2) RFID 스캔동작
      MOSI.14 : 읽기명령 전송
      MISO.14 : 엔코더로부터 수신한 RFID가 테이블에 있으면 1, 없으면 0
      PalletOffsetMonitor.pallet_opt 변수의 상위Word에 값이 반영된다.
      PalletOffsetMonitor.pallet_opt 변수의 하위Word에 테이블 인덱스(번호)값이 반영된다.

   3) 정방향 위치이동시 오프셋 적용여부 판단
      MISO.14 = 1 && Para[1][22].val==1일때 위치 오프셋에 적용함
      2-1)CW 이동시 EA_ofs적용, 0x4000:03
      2-2)CCW 이동시 EB_ofs적용, 0x4000:04
   4) 저장 및 읽기
      0x1010:04 "save" 명령시 저장됨.
      0x1011:04 "load" 명령시 all's (초기값)

   *디버깅을 위해 추가한 내용
   1)Get_PalletOffset(12345678); // pallet offset 300에 가상의 데이터 입력된 상태 (오프셋이 정상동작하는지 확인하기 위한 용도)

2) 모니터링 변수 추가
   0x2A96	LMS Motor Utilization (TBD)
   0x2A97	LMS Drive Utilization (TBD)
   0x2A98	LMS_Torque Limit Ready
   0x2A99	LMS Enc Start Value, Valid ON되는 시점의 엔코더값 모니터링 추가
   0x2A9A   LMS_Extended Proximity Sensor

3) 확장 근접센서 변수 추가
   T-format DF7로 전송
   0x2A9A PDO 맵핑에 추가.

5)ft-1.51 파렛오프셋 설정값 범위 수정
   UDINT->DINT //Range:-2147483647 ~ 2147483647  // LMS_Pallet_HallOffset


20211203
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.16 -> 1.02.100.17 MDM99: 20211203
1) 자동위치 모드 기능 추가
   1)자동위치로 진행중 GO비트에 따라 기동정지와 시작 가능
      GO비트 OFF->ON될때 MOSI.1 설정에 의존하던 것을 무조건 자동위치대기(50) 상태로 천이함.
      T54_50
   2)자동위치 모드 진입 조건
      T0_50
      T3_50
      T10_50
      T13_50
      T14_50

20211118
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.15 -> 1.02.100.16 MDM99: 20211118
1) LMS_SimpleLogging 수정
2) ID 입력받아서 오프셋을 출력하는 기능
   :300번지에 uuid 12345678 쓰기
   #EWR300%12345678&-22222222&-33333333&-44444444&-55555555&-666666666
   :uuid 12345678로 등록된 배열과 오프셋값 가져오기
   #ERD!12345678
   응답: [uuid][배열][v1][v2][v3][v4][v5][v6]
   $ERD12345678&300&12345678&-22222222&-33333333&-44444444&-55555555&-666666666
3) 자동위치 전환기능 검증


20211110
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.14 -> 1.02.100.15 MDM99: 20211110
1) 0x603F 에러코드값 변경
   -기존: CiA402에 정의된 에러를 표시함.(매뉴얼에 CiA402 대응 에러코드 참조)
   -변경: built-in에서 보이는 외부에러코드를 표시함.
   -상위 바이트 0xFF 값은 제조사 에러임을 나타냅니다.
       예) Encoder Open 일 경우 E.030 의 외부 에러 번호를 가지므로
        0x603F = (0xFF | 0x1E) = 0xFF1E 로 표시됩니다.
        0x1001 = 1 로 표시됩니다.
        0x6041.비트3 = 1 로 표시됩니다.
2) 오브젝트 0x3000번대 오류 개선 (cimon PLC와 연결시 오류 수정사항 반영)
   : 문자열(예, ABCD)로 입력시 해당 오브젝트의 기능이 수행되는 현상
   : 0X3001 숫자 2이상 입력되는 오류, 문자열(예, ABCD)로 입력시 AT 수행되는 현상
   : 0 을 넣어도 enable로 동작하는 현상 개선
   : ABCD문자를 넣으면 메모리값이 0으로 넘어옴
      0x3001  Auto Tuning [0~1]
      //0x3002  Smart Tuning [0~2]
      //0x3007  Auto Current Fbk offset Calibration
      0x3009  Fault History Clear
      0x300A  Absolute Encoder Multi Turn Clear
      //0x300C  User Parameter Init
      0x3010  Drive Reboot
3) 0x2F09, Motion History Obejct 추가
   RecordInfo[시간:옵션:프로파일]
   motion history 읽기 #PDR002F090110
   subindex: max 32
   1	Mo Motion #1
   2	Mo Motion #2
   3	Mo Motion #3
   ...
   16   Mo Motion #16

4) Samik 90.12~19까지 수정사항 포함.
5) 모터 모델 (0x2001) Undefined로 보이는 현상 수정
6) 전원 off/on 시 현재 위치값 설정
   1.52 D0 disable: 싱글턴으로 설정
           enable : 현재 위치값으로 설정
   변경점
   pos_fbk = pos_fbk_latch;  //20211105 싱글턴이 추가로 더해지는 버그가 있음 -> old값을 초기화해서 해결함.
   LMMT_Enc_data_old = Enc_data; // 위치오차=0
   LMMT_pos_fdk_del = 0; // delta=0


20211101 스페셜 버전
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.13 -> 1.02.100.14 MDM99: 20211101
1) EWR 한번에 쓰기하는 기능 추가
   EWR300% V1 & V2 & V3 & V4 & V5 & V6
2) 전류피드백 스케일 2배 확장
   // tmshim. 21.02.08
   // tmshim. 21.03.11
3) 60Hz 3Phase AC to DC Bus Ripple = 360Hz
   // 21.10.18
4) Read_Encoder_Version 함수 호출시 E.106 알람 클리어 안되는 현상 수정
   ID 0x0D 명령에 대해 DCT 엔코더는 반응이 없고, 이후 ID 0x07 알람 리셋이 안됨.

20211013 내부 버전
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.12 -> 1.02.100.13 MDM99: 20211014
1) BlackBox 기능 추가 (2.14.00.01 참고)
   DIE 명령 추가 #DIE 01 00 [채널1 00->all row data]
2) PalletOffset 600개 정의
   명령어 추가 (ERD, EWR)
     1) ERD300*  : 300번 배열 저장장소 읽기
     2) EWR300*  : 전체 저장(save)
     3) EWR300$  : 전체 로딩(load)
     4) EWR300!  : default값으로 램저장장소 설정
     4) EWR300@  : all 0's
     5) EWR3004-1234  : 300번 배열, index 4번 위치에 -1234 값 쓰기
     6) ERD3004  : 300번 배열, index 4번 위치 값 읽기.
3) XML추가 (TBD)
   0x4001 ~ 0x4278,  [0..600]
4) PDO 맵핑 추가
      0x2117: LMMT_Positive Distance/Position
      0x2118: LMMT_Negative Distance/Position
      0x2119: LMMT_1st Velocity
      0x211C: LMMT_2nd Velocity
5) PDO에 DigitalOutput: 0x60FE0050 (잘못된 값)으로 등록될때 hardfault 발생하는 현상을 수정함.
   EPsoftwareHardFault함수 수정: 폴트발생시 LED로 상태표시
   시간지연 함수 추가: Delay_1us, Delay_10us 함수 추가, HWtimer.c

20211005
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.11 -> 1.02.100.12 MDM99: 20211005
1) 엔코더 offset 적용 이슈
   이력: 진입 후 양측 Encoder가 Valid 된 순간 적용되는 엔코더 offset값이 반대편 엔코더의 정보가 사용되는 현상
   변경: 진입 후 엔코더 Valid A & B 되는 시점의 엔코더offset은 이전 엔코더 기준으로 적용됨.

   예시:
   (* EncA보드 ID:3, EncB보드 ID:5 로 가정하고 설명함)
   1) polarity : Normal일때
     1-1)정방향 이동
        ID 3일때는 EncA 오프셋만 적용(계속 업데이트됨)
        ID 3->5 or 5인 구간은 EncA의 마지막 오프셋 적용됨.

     1-2)역방향 이동
        ID 5일때는 EncB 오프셋만 적용(계속 업데이트됨)
        ID 5->3 or 3인 구간은 EncB의 마지막 오프셋 적용됨.

   2) polarity : Inverted일때
     2-1)정방향 이동
        ID 5일때는 EncB 오프셋만 적용(계속 업데이트됨)
        ID 5->3 or 3인 구간은 EncB의 마지막 오프셋 적용됨.

     2-2)역방향 이동
        ID 3일때는 EncA 오프셋만 적용(계속 업데이트됨)
        ID 3->5 or 5인 구간은 EncA의 마지막 오프셋 적용됨.

2) W 스코프에 LMS_TorqueLimit_readyFlag 모니터링 임시 추가.


20210928 시험용 버전을 01.02.100.09와 동일하게 원복함.
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.10
1)1.02.100.11


20210928
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.09 -> 1.02.100.10 MDM99: 20210928 DCT전달
1)위치완료 후 GO비트 OFF 혹은 서보 온오프이후 GO비트 ON시 반대방향의 엔코더offset이 적용되는 문제
  82.01 버전과 동일하게 코드 원복함 (사내 재현은 안됨)


20210927
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.08 -> 1.02.100.09 MDM99: 20210927 내부 코드정리버전 DCT전달
->1.02.90.7과 일치시킴.
1)과전류 deadzone 설정하는 방법
2)진입시 valid 신호에서 현재 위치값을 엔코더값으로 시작하도록 코드 원복함.(100.05와 같음)
3)엔코더 버전읽기 추가
   uI32 Read_Encoder_Version(void); //20210924

20210917
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.07 -> 1.02.100.08 MDM99: 20210917 DCT전달
1) 속도/위치 전환 기능 오류 수정
   :위치모드상태에서 외부진입 안되는 현상 수정
   동작
      1) MOSI bit8 VP flag 정의
         1일때 속도운전으로 배출시키면 자동으로 속도위치대기(50)로 천이함.
         0일때 속도운전으로 배출시키면 속도대기(0)로 천이함(기존 동작임)
      2) 속도런(2)상태에서 배출시 VP enable이면 속도위치대기(50)로 천이함. 위치대기(10-11-12-13)과 유사하다.
            LMS_VP_READY  =  50,
            LMS_VP_INIT,
            LMS_VP_VEL,
            LMS_VP_POS,
            LMS_VP_STOP,
      3) VP disable이면 속도대기(0) 혹은 위치대기(10)로 천이함.
      4) VP_POS(53) 에서 속도이동으로 빠져나오는 방법
            MOSI bit8 ON->OFF->ON
            속도이동으로 이동중 속도런(2)상태에서 파렛을 빠져나올때 MOSI bit8 ON이면 VP 대기(50)으로 천이함.
            VP대기(50)에서 MOSI bit8이 OFF되면 속도 or 위치 모드로 이동한다. (대부분 속도기동상태이므로 속도이동함)

  예시) 파렛 1개, 서보1대로 동작 확인하는 방법
      1) 파렛을 코일 위에 위치시키고 서보온 명령을 전송한다. MOSI=0x01
      2) 위치 자동전환 비트(MOSI.8)를 켜고 속도 이동하여 진입방향(왼쪽, 역방향)으로 배출시킨다.
         (파렛이 코일밖으로 빠져나오도록 수동으로 밀어준다.)
            >>MOSI=0x010D
      3) LMS_STATUS 상태가 위치자동모드 대기상태(50)임을 확인한다.
      4) 서보의 위치운전 방향을 정방향을 바꾼다. MOSI=0x0105
      5) 수동으로 파렛을 코일로 진입시킨다. 그러면 파렛은 티칭된 위치에 정지한다.
            <<LMS_STATUS (53)
      6) 왼쪽으로 배출하고 다음에 진입하는 파렛을 정위치 시킬려면 ON되어 있는 MOSI.8을 OFF->ON 을 하면 위의 상태를 반복하게 된다.
      MOSI.8이 OFF되는 시점에 속도기동되고 속도기동 배출 후 MOSI bit8이 ON이 되어 있어야 위치자동모드로 진입한다.
      트리거펄스 형태로 주어야 하며 파렛이 Open상태가 되기전에 ON을 해주어야 한다.
      7) MOSI.8을 OFF 시키거나 기동명령을 off시키면 일반적인 속도 혹은 위치운전으로 동작한다.


20210916
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.06 -> 1.02.100.07 MDM99: 20210916
1) RFID값 위치조정
   엔코더의 DF6(1byte)를 읽어서 0x2A3B (LMMT_RFID_SF) 오브젝트로 보여줌
2) 오브젝트 변경정리
   0x2A3B, 0x2A3C 추가
   0x2A3D LMMT_TorqueLimit_flag
   0x212D LMMT_Zero Current Limit Band
            01: Zero Current Limit Enable 추가
3) DC sync 인터럽트 개선
4) 파렛이 없는 구간에 대한 처리
      w_rad_old = 0;
      IoFlag.CurLmted = FALSE;
      IoFlag.VelLmted = FALSE;
5) 위치정지 후 target pos 변경시 방향이 바뀌지 않는 현상을 개선
   다음 코드 추가: lms_pos_dir = LMS_DIR_VALUE;
6) 코일이 없는 구간에서 위치기동 시작비트 ON->OFF시 드라이브 오버로드 문제 해결
   ->전류명령 0 값으로 제어유지
   ->코일이 있는 구간은 속도감속값으로 정지하도록 상태 천이 추가, LMS_POS_STOP(14)
7) 속도이동중 캐리어 오픈상태가 되면 자동으로 위치 대기모드로 전환하는 기능 추가
   ->MOSI bit8 ON일때 속도모드에서 동작함

20210906
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.05 -> 1.02.100.06 MDM99: 20210912 (내부 관리용)
1) 위치 덜가는 현상 수정, (pos_fbk초기화 오류)
2) EncMask, 0xfff -> 0x1FFF 변경
3) TG-ON 신호 개선
4) 속도 적분게인 초기화 기능 추가
   1)코일이 없는 곳에서 속도기동 on->off 시 알람발생하는 것을 개선
   2)파렛이 위치/속도 모드로 진입시 전류과대현상 개선
     그러나 속도모드로 배출시에는 전류과대현상 남아 있음.
     ->이건 Deadzone으로 보완해야 한다.

20210903
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.04 -> 1.02.100.05 MDM99: 20210903
1) RFID값 신규오브젝트 추가
   엔코더의 SF(1byte)를 읽어서 0x2A3B (LMMT_RFID_SF) 오브젝트로 보여줌
   PDO 할당, XML 변경
   0x2A3B, 0x2A3C 추가
2) 서보오프상태에서 서보온되고 목표위치 이동시 목표위치 업데이트 반영
   GO비트가 살아있으면 위치이동을 한다.


20210830
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.03 -> 1.02.100.04 MDM99: 20210830
1) 스텝이동 변수 32비트 확장
2) 홈밍완료 상태(25)에서 Home모드 비트6 ON->OFF시 위치/속도운전으로 변경되도록 수정
3) 홈밍후 현재 위치 초기화 기능
4) GO 비트 ON상태에서 홈비트 1->0으로 변경시 속도/위치 비트값에 따라 동작하도록 변경



20210806
///////////////////////////////////////////////////////////////////////////////
0)1.02.100.02 -> 1.02.100.03 MDM99: 20210806
1) case ErrCode_PowerOvLoad:       CiA402_LocalError(ERROR_CONTINUOUS_OVER_CURRENT);break;                 //  #define ErrCode_PowerOvLoad     0x25  // E.104 CONOL
2) 용어변경
  82.01             83.01(100.03)
  DO_H_START        DO_HOME_MODE
  DO_Not_H_START    DO_Not_HOME_MODE

  IX_FLAG.START_IXG   LMMT_MOVE
  LMSFlag.H_START     LMSFlag.HOME_MODE
  LMSFlag.POSI_SET    LMSFlag.POSI_MODE

3) POS_POS상태에서 상태천이 처리방식 수정
  LMMT_MOVE, LMMT_MOVE_OLD 개별상태 변수로만 사용

4) 알람코드 변경, CheckFdbOverLoad_LMMT
  토크 70%이상 30초이상 발생시 알롬코드 변경
  ErrCode_ContTqFbkOl -> ErrCode_PowerOvLoad
  #define ErrCode_PowerOvLoad     0x26  // E.104 PWROL

5) 100.1~2 변경사항 적용




20210604
///////////////////////////////////////////////////////////////////////////////
0) 1.02.90.01 -> 1.02.90.02 MDM99: 20210608
1)스텝이동 오류개선
  1)속도이동후 스텝이동으로 전환시 목표위치 계산방식 수정
    :이전 목표위치+스텝이동 위치로 이동하면서 코일을 벗어나는 현상 발생
    ->target position+스텝이동량-> 모터현재위치+스텝이동량으로 설정함.
2)자동 서보-On에 준하는 조건에서 E.037 Fault 발생 문제 수정


20210604
///////////////////////////////////////////////////////////////////////////////
0) 1.02.82.01 -> 1.02.90.01 MDM99: 20210604
1) 파리미터 rangeover 발생시 추적기능
  : 최대 5개까지 볼수 있음. MDE920~924
2) ft-4.35 Main Current Regurator Max Bandwidth, [%] 추가, 100%=1Khz 기준
  : ft-4.26~34까지는 dumy로 추가됨.
3) 상태변수가 속도모드 시작으로 되어 있는 것을 ready로 변경함
  ->LMS_STATUS = LMS_VEL_INIT(1) -> LMS_VEL_READY(0)
4) 3rd party로 모터정보 등록시 전기각 오프셋 적용
  ->기본 0도 설정해야 한다.
5)ft-1.40 LMS Deadzone Current Level 범위 확장
  : [0~500] -> [1~2147483647] 스텝이동시 큰값 입력되지 않도록 주의!
6)MDE 명령추가
3) 모니터링
      Mon_AccDec = 0; // bhchun 20210604
      Mon_Jerk = 0; // bhchun 20210604


20210511
///////////////////////////////////////////////////////////////////////////////
0) 1.02.77.01 -> 1.02.77.10 MDM99: 20210511
1) 역방향 위치이동 안되는 오류 수정
2) E.058 발생시 원인 추적 코드 추가
  MDE 920 ~924
3)ft-435 전류제버 대역폭 0~100% 설정 추가
  #define WN_CC1  6283    // 1kHz
4)버전표기 추가, MDE 확장
5) 내부변수 모니터링 추가

36
37
38 Delta_acc
39 LMS_del_acc_pre[0]
  79  Mon_AccDec
  80  Mon_Jerk
  81  LMS_Trigger_pos
  82  Initial_pos
  83  LMS_Target_position
  84  Motor.wn_cc

