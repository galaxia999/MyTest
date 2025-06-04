===============================================================================
D8 LMMT F/W Revision History
===============================================================================

*버전관리
**bitbucket 회사 서버에 저장하고 "branch_날짜_버전표기"로 만들어서 팀멤버가 다운로드해서 사용하게 함.
(서버 저장소 이름: "DCT_LMMT" main)


KNOWN ISSUES:
1) Rangeover 알람 발생시 확인가능한 방법추가
2) D8 LMMT 공장초기화후 설정하는 방법 (NRSA)
	1) 공장초기화 실행
	2) D7+모터연결->정상
	3) D8 LMMT 로터리모터 구동
		#STR0001


## New Enc Version : 유진디스컴 적용(2022/11/29)
DF0/1 : Master Enc.
DF2   : Proximity Sensor Status (기존과 동일)
DF3   : Hall Sensor Status (기존과 동일)
DF4/5 : Slave Enc.
DF6   : RFID tag
DF7   : Ext Proxi. Sensor Status

## 이름 통일(D7, D8_M 공통)
	0x2A31 Axis-A LMMT_Encoder RX ID
	0x2A32 Axis-A LMMT_Digital Input Data
	0x2A33 Axis-A LMMT_Digital Output Data
	0x2A34 Axis-A LMMT_Sensor Data
	0x2A35 Axis-A LMMT_Status
	0x2A36 Axis-A LMMT_Encoder Data Difference
	0x2A37 Axis-A LMMT_Encoder Valid
	0x2A38 Axis-A LMMT_Initial Pos
	0x2A39 Axis-A LMMT_Trigger Flag
	0x2A3A Axis-A LMMT_Motor Jerk
	       
	0x2A3B Axis-A LMMT_RFID_DF6
	0x2A3C Axis-A LMMT_Pallet Offset
	0x2A3D Axis-A LMMT_Motor Inactive Flag
				Axis-A Pallet_CW_AOFF
				Axis-A Pallet_CW_BOFF
				Axis-A Pallet_CCW_AOFF
				Axis-A Pallet_CCW_BOFF
	0x2A3E Axis-A LMMT_ABS Offset
	0x2A3F Axis-A LMMT_Trigger Pos
	0x2A40 Axis-A LMMT_Homing Error
	0x2A41 Axis-A LMMT_Drive Enabled
	0x2A42 Axis-A LMMT_Encoder Version


## 상위 모션지령 조건
	MO = LM or RM
	1) LM = L & PHS 0x0F
	2) RM = R & PHS 0x3E

## 긴급정지 동작 알람.
  -서보온 상태에서 아래 알람 발생시 동작함.
  - 통신알람 범위 0x90~0x9F이면 double flash 동작
	#define ErrCode_ContTqFbkOl      0x25  // E.022 CONOL -> Motor Overload, CheckFdbOverLoad()
	#define ErrCode_PowerOvLoad     0x26  // E.104 PWROL -> Not enabled 20220531 83.01버전에서 추가됨.
	
	#define ErrCode_ET_NoSync        0x90  // E.200 NOSYN  // No Sync Error
	#define ErrCode_ET_SYNC          0x91  // E.201 SYNCH  // Synchronization Error
	#define ErrCode_ET_DCPLL         0x92  // E.202 DCPLL  // DC PLL Sync Error
	#define ErrCode_ET_SMWTD         0x93  // E.203 SMWTD  // Ecat Watchdog error.
	#define ErrCode_ET_INOUTMAPPING  0x94  // E.204 IOMAP  // Input output mapping error.
	#define ErrCode_ET_OPMODE        0x95  // E.205 OPMOD  // Unsupported Operation mode
	#define ErrCode_ET_ETCFG         0x96  // E.206 ETCFG  // EtherCAT Config Error.
	#define ErrCode_ET_COE           0x97  // E.207 ETCOE  // COE error.
	#define ErrCode_ET_FOE           0x98  // E.208 ETFOE  // FOE error.
	#define ErrCode_ET_PDO           0x99  // E.209 ETPDO  // PDO error and default applied.

## LMS용 이동평균 부하율 시간 설정 기능
  -평균 부하율에 대한 설정 시간 동안의 이동 평균값을 계산함.
  -내부 버퍼크기 : 99999+1 로 설정
  1)평균부하율 계산(D7, D8_M 공통)
  - ft4.23 Current Feedbvack Squared Integral Interval, 단위 [ms], LMMT special
  - MDM 44 0x2A2C Current RMS
  - MDM 45 0x2A2D Current RMS Max
  2)이동평균부하율 계산
  -LMS 이동평균 부하율 시간 설정(D7)
  - ft-1.55 0x2137 LMMT_Current Load Moving Avg Time Setting, 0~99999, 단위:sec
  - MDM 44 0x2A96	LMS Motor Utilization RMS
  - MDM 45 0x2A97	LMS Motor Moving Utilization RMS
  ----------- D8_M (A/B/C축) ------------
  - ft-3.44 0x232C Axis-A LMMT_Current Load Moving Avg Time Setting, 0~99999, 단위:sec
  - MDM 44 0x2A96	LMS Motor Utilization RMS
  - MDM 45 0x2A97	LMS Motor Moving Utilization RMS
  
## LMS 파라메터 specific
    sI32 pr118; //20220420 for TEST torque command 50(=5%, 0.1% unit)
    sI32 pr119; //20220420 for TEST torque command direction, 0:(+), 1:(-)

## CTT specific
   // CTT 테스트 시 예외처리 (CTT를 돌리는 경우 0x2008를 888로 설정하고 테스트한다.)
	// PASSWORD(0x2008==888) 일 경우 명령처리를 하지 않도록 함.
	// 이현규, 20210317
	// CTT 테스트 중 PASSWORD가 풀리는 문제 개선, 20210528, 이현규 
	if(Para[0][0][8].val == ID_Tester)
	    CTTTesterLock = 1;
	    
## RFID 사양
	DF6 필드 사용
	PDO 주소: 0x6004/0x6804/0x7004, 모니티렁 주소: 0x2A3B/0x3A3B/0x4A3B
	파렛 valid 신호 ON이고 ft-1.22=1인 경우에만 RFID값 엡데이트되고 다른 경우는 0 처리.
	
## 파렛오프셋값 적용 방식
	ft-1.22=1인 경우에 사용
	ft-3.41 D1 disable인 경우 엔코더 DF4,DF5 적용
	ft-3.41 D1 enable 인 경우 RFID 검색에 의해서 테이블에서 등록된 값을 적용
						만약 테이블에 등록되어 있지않으면 0 값이 적용됨.
	0x2A3C/0x3A3C/0x4A3C에 항시 적용됨.
	rsware는 상시 모니터링
	
## 근접센서 확장IO 적용 방식
	엔코더 DF3, DF7의 비트 5,6에 할당
	Valid 신호와 상관없이 항상 업데이트됨.
	0x2A9A/0x3A9A/0x4A9A에 엔코더A(비트 0,1), 엔코더B(비트8,9) 항시 적용됨.
	rsware 상시 모니터링

## 엔코더값 채널 할당
      0x60BA  |  Touch Probe Pos1 Pos Value | pAx->LMS_Vars.ID3_Enc
      0x60BB  |  Touch Probe Pos1 Neg Value | pAx->LMS_Vars.ID5_Enc
      0x60BC  |  Touch Probe Pos2 Pos Value | pAx->LMS_Vars.Master_Enc
      0x60BD  |  Touch Probe Pos2 Neg Value | pAx->LMS_Vars.Slave_Enc
    -------------------------------------------

## 파라메터 저장명령(0x1010:01,0x1010:04)시 파렛오프셋 자동저장 기능 추가(DCT요청사항, D7과 기능동작이 동일하도록 통일함) 
 	0x1010:01 "save" 명령시(Store all Parameters)           flgCallBack = 0x80101001, FlashBackup 과 SavePalletOffset 모두 호출함. 
    0x1010:03 "save" 명령시(Store cia402 Parameters)        flgCallBack = 0x80101003, SavePalletOffset 호출
    0x1010:04 "save" 명령시 (Store CSD7 specific Parameters) flgCallBack = 0x80101004, FlashBackup 호출

## 파라메터/오브젝트 list 정리
NAME									[CSD7         ] [D8           ][PDO]INDEX
-------------------------------------------------------------------------------
LMS_Positive Distance/Position			0x2117	Ft-1.23	0x2121	P0-1.33	Y	600A
LMS_Negative Distance/Position			0x2118	Ft-1.24	0x2122	P0-1.34	Y	600B
LMS_1st Velocity						0x2119	Ft-1.25	0x2123	P0-1.35	Y	600C
LMS_1st Acceleration time				0x211A	Ft-1.26	0x2124	P0-1.36	Y	600D
LMS_1st Deceleration time				0x211B	Ft-1.27	0x2125	P0-1.37	Y	600E
LMS_2nd Velocity						0x211C	Ft-1.28	0x2126	P0-1.38	Y	600F
LMS_2nd Acceleration time				0x211D	Ft-1.29	0x2127	P0-1.39	Y	6010
LMS_2nd Deceleration time				0x211E	Ft-1.30	0x2128	P0-1.40	Y	6011
LMS_2nd Positive Distance/Position		0x213B	Ft-1.59	0x2330	P0-3.48	Y	6012
LMS_2nd Negative Distance/Position		0x213C	Ft-1.60	0x2331	P0-3.49	Y	6013
// 0x6014 reserved
// 0x6015 reserved
LMS_EncA Base Deadzone Offset			0x2131	Ft-1.49	0x213B	P0-1.59			LMS_엔코더A 불감대 위치 오프셋
LMS_EncB Base Deadzone Offset			0x2132	Ft-1.50	0x213C	P0-1.60			LMS_엔코더B 불감대 위치 오프셋

Pallet Out Counter for CW Moving							0x222F:01	Ft-2.47 D0	0x222F	P0-2.47 D0	Y	6005:01
Pallet Out Counter for CCW Moving							0x222F:02	Ft-2.47 D1	0x222F	P0-2.47 D1	Y	6005:02
SyncEA Velocity Feedback Keeping Time for EB Trigger		0x2230		Ft-2.48 	0x2230	P0-2.48		Y	6016
SyncEB Velocity Feedback Keeping Time for EA Trigger		0x2231		Ft-2.49 	0x2231	P0-2.49		Y	6017
RFID Offset Apply Delay Time							 	0x2232		Ft-2.50 	0x2232	P0-2.50		Y	6018

---------미러 모니터링 6000번대
SyncEA_Status			      			0x2AA1	MDE-161	0x2AA1	MDE-161	Y 66A1
SyncEB_Status			      			0x2AA2	MDE-162	0x2AA2	MDE-162	Y 66A2
SyncEA_StartValue		      			0x2AA3	MDE-163	0x2AA3	MDE-163	Y 66A3
SyncEB_StartValue		      			0x2AA4	MDE-164	0x2AA4	MDE-164	Y 66A4
SM_SYNC_Flags[4]
	TRIGGER_EA		   				    0x2AA5	MDE-165	0x2AA5	MDE-165	Y 66A5:01
	TRIGGER_EB		   				    0x2AA5	MDE-166	0x2AA6	MDE-166	Y 66A5:02
	NET_PALLET_EA		   				                                Y 66A5:03
	NET_PALLET_EB		   				                                Y 66A5:04
SyncEA_Pallet_IN Counter				0x2AA7	MDE-167	0x2AA7	MDE-167	Y 66A7
SyncEB_Pallet_IN Counter				0x2AA8	MDE-168	0x2AA8	MDE-168	Y 66A8

LMS_ID3_Enc								0x2AA9 MDE-169 0x2AA9 MDE-169	Y 66A9
LMS_ID5_Enc								0x2AAA MDE-170 0x2AAA MDE-170	Y 66AA
LMS_Master_Enc							0x2AAB MDE-171 0x2AAB MDE-171	Y 66AB
LMS_Slave_Enc							0x2AAC MDE-172 0x2AAC MDE-172	Y 66AC
LMS_MOSI_GO								0x2AAD MDE-173 0x2AAD MDE-173	Y 66AD LMS_MOSI_GO	                 모션 기동 명령, MOSI.2		
SyncEA_Velocity Feedback				0x2AAE MDE-174 0x2AAE MDE-174	Y 66AE SyncEA_Velocity Feedback 동시이동 엔코더A 속도피드백		
SyncEB_Velocity Feedback				0x2AAF MDE-175 0x2AAF MDE-175	Y 66AF SyncEB_Velocity Feedback 동시이동 엔코더B 속도피드백		
Sync_VelFdk TimeTick					0x2AB0 MDE-176 0x2AB0 MDE-176	Y 66B0 Sync_VelFdk TimeTick	   동시이동 속도피드백 시간틱
-------------------------------------------------------------------------------

##5) LMS_TorqueLimit_readyFlag(토크제한 플래그) 동작 정의
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




to do list for FW enhancement
1) 반복되는 조건문 처리를 효율적으로 하는  방법 -> 엔코더 분해능 변수를 코드에 직접 적용하도록 수정하는 것을 검토필요함.
			if (pAx->EncMask  == 0x1fff) vel_cmd = (pAx->Kp_pc * 0.0001) * (FP32)pAx->pos_err;
			else                         vel_cmd = (pAx->Kp_pc * 0.00001) * (FP32)pAx->pos_err;
2) 엔코더 통신 4회이상 오류 발생시 알람 처리, 표준FW는 3회로 고정되어 있음
	비트 정의: D4:TIMOT, D3:SFOME, D2:FOME, D1:CRCE, D0:CONTE

       }else if (DMAReg5810[ax].RxDATA5 & 0x1F00){
           if ( EnCommErr_cnt[ax]>=4){
               ActionsOnAlarm(ax,ErrCode_EnCommErr);
               LMS_Valid_Direct[ax] = 0;
               LMS_Valid_Transf[ax] = 0;
               LMS_ENCV[ax][1] = 0;
               LMS_ENCV[ax][0] = 0;
           }
           else EnCommErr_cnt[ax]++;
       }else if (pAx->Flag.Err_reset){
       

신호 확인용:
1) DF3 bit6 pos valid
   DF3 bit5 z-phase

3)특이사항
	파라메터 변경 후 저장버튼 누른 상태에서 FW 업데이트하면 파라메터 정상
	파라메터 변경 후 엔터만 친 상태에서 FW 업데이트하면 파라메터는 변결되지 않는다.
	*파라메터 변경 후 엔터만 친 상태에서 RESET을 한 이후 파라메터 상태를 보면 정상.(RESET 발생시 flash backup을 하는듯함)

전류 BW 차이
	CSD7 LMMT의 속도 Bandwidth 는 1Khz 기준으로 ft-4.06 (H,M,Low)로 설정합니다.	
	ft-4.06 Low 설정시 BW는 33% ( ~ 333Hz)로 동작합니다.										 	

	D8 LMMT의 속도 Bandwidth 는 3Khz 기준으로 ft-4.06 (H,M,Low)로 설정합니다.
	ft-4.06 Low 설정시 BW는 33%는 1Khz가 됩니다.
	ft-4.35 는 ft-4.06 BW의 사용제한값입니다.	
	  0 : 100%, 10%설정시 BW는 대략 333Khz가 됩니다. 

속도 감속시점 조정 기능
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


************** 펌웨어 작성시 참고사항 **************
	BackGround(CPU0)
		//CPU0 파라미터, 파렛오프셋 공유
		pAx->pr246 = Para[ax][2][46].val;
		...
		pAx->pallet.pallet_id  = PalletOffsetMonitor[ax].pallet_id;
		sDrive_8000.aEntries[0] = PalletOffsetMonitor[ax].pallet_id; //CPU1->CPU0 변경된 오브젝트 업데이트
		
		g_Poffset[ax] = pAx->LMS_Position_offset; 					 //CPU1->CPU0 오브젝트 업데이트
		g_LMMT_ID3_Enc[ax] = pAx->LMS_Vars.MON_ID3_Enc;



TO_DO
1)동시이동 엔코더 스위칭 룰 사양
	1)

) 기타 수정할 내용
	1) 20230705
		엔코드 스위칭시 Direct(off), Tranf(on)되는 특수한 경우에 엔코더값이 0 으로 초기화되는 현상을 개선함
   		-동시이동과 같은 스위칭 조건시 발생할 수 있음. 
	    :CW방향 설정된 상태에서 CCW방향으로 손으로 파렛을 밀면 valid B->A&B->A (스위칭발생) 시점에 발생함.
   		-또는 valid A->A&B(스위칭발생) 발생시 Valid A(off), Valid B(on) 상태 발생함.
   		ID3에 대한 Valid 신호는 OFF, ID5에 대한 Valid 신호는 ON 상태가 될때가 있음. 
     	이때 enc_data값이 한주기 0으로 보임->속도리플 발생 가능성 있어서 개선함.	 

	2) 확장 센서 오브젝트 0x2A9A
		uI32 MON_ID3_ExtPHSensor; //사용함
		uI32 MON_ID5_ExtPHSensor; ///사용안함
		
	3) direct, transfer 조건 D7과 동일하게 통일
		if(!LMS_Valid_Direct[ax] && !LMS_Valid_Transf[ax]){ // none		
	4) 레지스터 수정
		//tmpEncValue = (sI16)Reg5810(ax).RxDATA3;
		tmpEncValue = (sI16)DMAReg5810[ax].RxDATA3; //20241211
		
		
2) 파라미터 추가
	D7          					D8            내부변수
	----------------------------------------------------------------------------------------------
	ft-2.47 D0, 0x222F:01, 		ft-2.47	LMS_palletOutCnt_CW  									CW방향으로 새로운 파렛 진입시 배출해야 할 파렛의 갯수
	ft-2.47 D1, 0x222F:02,		ft-2.47	LMS_palletOutCnt_CCW 									CCW방향으로 새로운 파렛 진입시 배출해야 할 파렛의 갯수
	ft-2.48,    0x2230,         ft-2.48	SyncEA Velocity Feedback Keeping Time for EB Trigger	EB 트리거시 EA 속도피드백 유지시간
	ft-2.49,    0x2231,         ft-2.49	SyncEB Velocity Feedback Keeping Time for EA Trigger	EA 트리거시 EB 속도피드백 유지시간
	ft-2.50,    0x2232,         ft-2.50	RFID Offset Apply Delay Time							RFID 오프셋 적용 지연 시간
	----------------------------------------------------------------------------------------------
	0x222F	Pallet Out Counter for SyncMotion	동시이동 파렛 배출 개수
		동시이동 파렛배출 카운터   CW	1	Pallet Out Counter for CW
		동시이동 파렛배출 카운터   CCW	2	Pallet Out Counter for CCW
		동시이동 파렛배출 카운터2 CW	3	reserved3
		동시이동 파렛배출 카운터2 CCW	4	reserved4
		
       모니터링 ----------------------------------------------------------------------------------------
	0x2A96 150 LMMT_Motor Utilization(RMS)			,0x2A2C Current Load Factor Feedback(RMS)와 같은값임.
	0x2A97 151 LMMT_Motor Moving Utilization(RMS)	,0x2A96의 ft-1.55시간동안의 이동평균값  
	0x2A98 152 LMMT_Torque Limit Ready
	0x2A99 153 LMMT_Enc Start Value, Valid ON되는 시점의 엔코더값 모니터링 추가
	0x2A9A 154 LMMT_Extended Proximity Sensor
	
	0x2A9B 155 PalletOffsetMonitor.pallet_id
	0x2A9C 156 PalletOffsetMonitor.pallet_opt
	0x2A9D 157 PalletOffsetMonitor.EA_ofs
	0x2A9E 158 PalletOffsetMonitor.EB_ofs
	0x2A9F 159 PalletOffsetMonitor.EA_ofs_dft
	0x2AA0 160 PalletOffsetMonitor.EB_ofs_dft
	
	0x2AA1 161 SyncEA_Status 동시이동 엔코더A 상태
	0x2AA2 162 SyncEB_Status 동시이동 엔코더B 상태
	0x2AA3 163 SyncEA_StartValue 동시이동 스위칭시 엔코더A raw값
	0x2AA4 164 SyncEB_StartValue 동시이동 스위칭시 엔코더B raw값
	0x2AA5 165 Sync_TRIGGER_EA : 동시이동 엔코더A 트리거 신호, EA Valid OFF->ON 시점에 SET, Valid 상태가 바뀔때 초기화(OFF)된다.
	0x2AA6 166 Sync_TRIGGER_EB : 동시이동 엔코더B 트리거 신호, EB Valid OFF->ON 시점에 SET, Valid 상태가 바뀔때 초기화(OFF)된다.
	0x2AA7 167 SyncEA_PalletInCnt 동시이동 CW 파렛 카운터, 동시모드에서 EA 홀센서의 Rising(0->1) 트리거 발생시 1씩 증가하는 증분형 카운터
	0x2AA8 168 SyncEB_PalletInCnt 동시이동 CCW 파렛 카운터, 동시모드에서 EB 홀센서의 Rising(0->1) 트리거 발생시 1씩 증가하는 증분형 카운터
	0x2AA9 169 LMS_ID3_Enc 엔코더A raw data ID3
	0x2AAA 170 LMS_ID5_Enc 엔코더A raw data ID5
	0x2AAB 171 LMS_Master_Enc 마스터 위치의 엔코더
	0x2AAC 172 LMS_Slave_Enc 슬레이브 위치의 엔코더
	0x2AAD 173 LMMT_MOVE MOSI.2			상위 기동명령에 대한 모션동작을 정확히 분석하기 위해 추가함.(MOSI.2)
	0x2AAE 174 SyncEA_Velocity Feedback 동시모드에서 EA 속도피드백
	0x2AAF 175 SyncEB_Velocity Feedback 동시모드에서 EB 속도피드백
	0x2AB0 176 Sync_Velfdk Timetick		동시모드 속도피드백 시간틱

----사용되지 않는 변수
	SYNC_EA_NEW_PALLET : EA Valid OFF->ON 시점에 SET, Valid 상태가 0아닌 값으로 바뀌어도 초기화되지 않는다.
	SYNC_EA_NEW_PALLET : EB Valid OFF->ON 시점에 SET, Valid 상태가 0아닌 값으로 바뀌어도 초기화되지 않는다.

	----------------------------------------------------------------------------------------------
	
## 스위칭 함수에서 Valid 오동작을 막기 위해 구동 방향 조건을 넣는 방법은 가능할까..


///////////////////////////////////////////////////////////////////////////////
0) V02.00.00.00 20250416 ECN 빌드용
1) Ft-5.57(LMS Overload Detection Level)추가 
  - Parameter Range 0 ~ 100
  - Ft-5.57값이 0이면 기존 검출레벨로 알람 발생.(Overload검출레벨: E.022 -> 정격전류의 110%이상 2초, E.104 -> 정격전류의 70%이상 30초)
  - Ft-5.57값이 0이아닐 경우 기존 검출레벨 * Ft-5.57 * 0.01을 검출레벨로 적용함.

///////////////////////////////////////////////////////////////////////////////
0) V02.00.00.00 20250404 ECN 빌드용
1) 오브젝트 관련 수정 
   -오브젝트 타입과 사이즈 mismatch 수정
   -오브젝트 PDO 맵핑 속성 PdoMapping flag 삭제함, 0x2AA1 ~ 0x2AB0 동시이동 각축
2) 블랙박스의 입력신호 범위를 46에서 999으로 변경함.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.67 20250326 ECN을 위한 수정사항 추가
1) 오실로스코프 채널값 기본값 변경
	CH C Velocity Feedback -> Current Feedback
	CH D Velocity Feedback -> Current Command
2) LMS 인코더의 홀센서 상태값 모니터링과 오실로스코프에 추가
	LMS Hall Value, MDM 23
3) LMS 제어용 모니터링 변수 맵핑
	MDM 20 속도루프에 실제로 적용되는 속도에러 		 : Analog velocity command voltage
	MDM 21 파렛이동 진입시 실제로 적용되는 속도피드백  : Analog current command voltage


///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.66 20250214 위치명령 업데이트 오류로 비정상적인 구동 현상 수정
1) 홈밍 기능 수행시 홈밍 후 정지위치가 홈오프셋으로 동작하지 않고 제1/2위치값으로 동작하는 현상
  -59버전에서 수정한 Update_TargetPositionWithDirection 함수를 LMMT control에서 항상 호출하도록 변경한 부분의 side effect
  -홈밍모드에서는 master position이 홈밍 오프셋값으로 적용되도록 수정함.
2) 위치정지 + GO OFF 상태에서 구동방향(MOSI.3) 바꿀 때 파렛이 이동하는 현상
  -59버전에서 수정한 Update_TargetPositionWithDirection 함수를 LMMT control에서 항상 호출하도록 변경한 부분의 side effect
    원인:
    정지 상태에서 방향을 바꾸면 Update_TargetPositionWithDirection 함수에 의해서 target position이 변경된다.
    이때 LMS_STATUS는 변경되지 않은 상태에서 위치에러(pos_err)만 발생하여 속도명령이 생성되어 파렛이 이동하게 됨.
   수정:
    정지상태에서 방향을 바꾸면 LMS_STATUS(10) 준비상태로 천이함.
    정지상태에서 위치값을 바꾸면 LMS_STATUS(10) 준비상태로 천이하는 것과 동일하게 처리함.
  Update_Direction_And_StatusChanging(ax) 추가하고 Update_TargetPosition_with_PDOupdateTiming 함수와 함께 사용함.
  -적용되는 상태: 위치완료(13/53/63) 상태에서 구동방향 변경시 위치준비(10)

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.65 20250213 S커브 적용 개선 (DCT전달)
1) S커브 실시간 적용 개선하면서 로터리모터 구동안되는 오류 수정
2) 속도가감속과 S커브를 속도/위치/자동위치/동시이동에 적용
	적용단계: 속도/위치/자동위치/동시이동 초기화 단계(LMS_VEL_INIT/LMS_POS_INIT/LMS_VP_INIT/LMS_SM_INIT)
	호출함수: Update_velocityAccDec(ax), ApplicationPara(208, ax);
3) 위치티칭값 변경 있는 경우 NEW PALLET ON 설정하여 바로 위치정지 가능하도록 함.(10->60->61->63)
   -신뢰성 시험 후 결과로 판단하여 결정함.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.64 20250207 동시이동 2P1C Unit Test 결과를 반영한 FW 수정사항 적용함.
1) 위치값 갱신하는 방법 결정 (D7: 송신ID(TxID) 기준, D8 : 수신ID(rxID) 기준으로 스위칭시 위치값 초기화함)
   -특이사항: 
     송신ID 기준 시험 결과: 목표위치 덜 가서 정지하는 현상 발생함.
     위치 초기화하는 함수는 Update_PosFbk_EncOld 사용함.
2) 속도피드백 유지시간 기능 추가
3) 인코더 개별 속도피드백과 속도피드백 필터, 위치값 계산 추가
4) MOSI명령을 LMS 제어부와 동기화 처리 like D7
	MOSI.2 필터링 처리함. 동기화하지 않음 like D7
	MOSI.3 LMS 제어부와 동기화. like D7
	MOSI.5 LMS 제어부와 동기화. like D7
	MOSI.7 LMS 제어부와 동기화. like D7
	MOSI.13 LMS 제어부와 동기화. like D7
5) 기타
   -속도에러 표시 단위 오류 수정, unit [mm/s]
   -동시이동 위치정지 상태(63)에서 기동명령 MOSI.2 off->on시 위치준비(10)로 이동하지 않고 동시이동 위치준비(60)로 천이함.
       상태 10과 60에서 처리하는 코드가 동일하여 FW 동작에는 영향을 주지 않음.
   -MDM 166 Sync_TRIGGER_EB 표시 오류 수정

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.63 20250121 동시이동  Seed FW 완성본, 동시이동 모드에서 RFID 오프셋 적용 오류 수정(D7 88버전) 
1) 동시이동 모드에서 RFID 오프셋 적용시 오프셋 검색완료 되지 못하여 오프셋 적용안되는 현상 수정
   -일반모드에서 RFID 오프셋 지연 기능은 정상 동작함.
	-타이머 시간틱 32비트 변수로 확장 (최대 10000ms 입력시 *16하여 160000 입력됨)
   -동시이동 파렛 진입시 진행방향에 따라 트리거 신호를 구분하여 처리함
	  정방향 CW 구동시 EA측 외부 진입 시간 지연틱 1부터 시작.
	  역방향 CCW 구동시 EB측 외부 진입 시간 지연틱 2부터 시작.
   -RFID 오프셋 검색완료되는 시점에 파렛오프셋(Pallet Offset)을 한번 갱신하여 위치명령이 업데이트되도록 한다.
   -RFID 오프셋 지연 시간 설정값은 파렛A의 Trigger EA ON되는 시간과 다음 파렛B의 
 	 Trigger EA ON되는 시간보다 작게 설정해야 한다. (설정 가능한 최대 시간 범위 이내 사용해야 함.)
   -RFID_OFFSET_APPLYED_FLAG 플래그(CPU0 처리결과를 CPU0에서 오프셋 한번만 적용하기 위한 플래그)
   	  위치명령에 오프셋이 반영하고 경우 ON 설정함.
   	  전원 리셋시, 파렛 오픈시 OFF

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.62 20250115 동시이동  Seed FW 완성본
1) HW/FW 비상정지 개선
   -파렛 정지 속도 판단 수식을 수정하여 감속시간이 정확하게 나오도록 개선함.
       속도0 정지 명령 판단 기준: 0.001mm/s 이하
   -적용되는 운전모드 확대
       개선전: 속도/위치/홈밍 모드에서만 비상정지 동작
       개선후: 속도/위치/홈밍/스텝/자동위치/동시이동 등 비상정지 모드외 상시 동작되도록 수정

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.61 20250110 동시이동용 PDO 맵핑으로 기본값 설정되도록 XML 변경
1) 위치 정지시 영속도 제어로 정지하는 기능 확대 적용
     서보온상태에서 파렛이 진입할 때  위치 정지 명령이면 프리런되는 현상 개선
     자동위치(50)/동시이동(60) 대기 상태에서 적용됨.
     외부에서 Servo On -> Pos Mode -> 파렛 진입 시 Free Run 되는 현상 수정
     대기(READY) 상태에서 기동시 속도명령(1/2 Velocity) 변경시 가감속값 업데이트함.	
2) 운전모드 우선순위 조정
     동시이동(SM) > 자동위치(VP) > 속도위치(POS)=스텝이동(STEP)=홈밍(HM)
3) 스텝모드에서 기동시 속도명령 변경에 따라 실시간 가감속 업데이트되도록 동작 개선
     기동시 속도명령(1/2 Velocity) 변경시 가감속값 업데이트함.
     대기상태에서 위치값(제1/2 위치값) 변경시 적용되도록 수정
4) HW(input1) 비상정지시 적용되는 감속값(ft-2.07)이 정상적으로 동작하도록 수정함.
   ft-2.06, ft-2.07 로터리기준으로 되어 있는 것을 리니어 타입 구분하도록 수정함.
    기존에는 단위는 m/s^2인데 기본값이 로터리모터 단위로 41.667 rev/s^2로 크게 되어 있었다.
   default 감속도값=41.667[m/s^2] 단위일 때 정지시간은 대략 ~20ms(at 0.4 m/s)이내 빠른 감속으로 정지하였다.
    구동속도 : 0.2 m/s
    수정 전 FW: 41.667 적용시 감속시간 16ms 정도 측정됨.
    수정 후 FW: 0.2 [m/s^2] 적용시 감속시간 1[sec]로 정확하게 계산됨. 
	          0.4 [m/s^2] 적용시 감속시간=속도/감속도= 0.2[m/s]/0.4[m/s^2]=0.5[sec]
5) 통신끊김시 비상정지 동작하지 않는 오류 수정
   -통신 SafeOP 변경시 비상정지 기능이 동작중이면 즉시 서보으프하지 않도록 ecat_fault_delayStatus 상태값 추가하여 수정함.
   -속도명령 0과 정지상태를 비교시 정수화하여 비교함.(sI32)(pAx->w_ref)
6) HW 비상정지시 감속값을 리니어 모터 단위로 처리하도록 개선함. 
   -IO 비상정지시 적용되는 감속값(ft-2.07)이 로터리모터 기준(rev/s^2)에서 리니어 단위(mm/s^2)로 적용되도록 수정함.
7) 매크로 수정	
   abs ->labs
   
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.61 20250103 20250110 동시이동 FW 변경점 시작, 동시이동 미러 오브젝트 추가
1) 동시이동 미러 오브젝트 추가 (0x663B ~ 0x663D,  0x6696 ~ 0x669A, 0x66A1 ~ 0x66B0 각축)
   - 사용 영역
         미러 오브젝트 [0x6000 ~ 0x60FF] : 파리미터 변수 변수 256개 맵핑 (각축)
         미러 오브젝트 [0x6600 ~ 0x66FF] : 모니터링 변수 256개 맵핑 (각축)
   - 0x2A3B ~ 0x2A42 동시이동 A축 XML 제조사 영역 PDO맵핑 속성 삭제
   - 0x663B ~ 0x6642 동시이동 A축 XML 미러 오브젝트
2) 미러 오브젝트 사용을 위해 제조사영역 모니터링 오브젝트에 OBJACCESS_TXPDOMAPPING 속성 삭제 
   -(0x2A3F ~ 0x2A42), (0x2A9B ~ 0x2AA0)
3) 동시이동용 PDO 맵핑으로 기본값 설정되도록 XML 변경      
   
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.60 20241231 동시이동 FW 변경점 시작, 위치정지 동작 사양 개선, 동시이동 로직 추가. 모니터링 추가
1) 위치/자동위치/동시모드의 위치정지 상태에서의 동작 사양
	(1)위치모드 진입시  MOSI.2 OFF시 현재 위치 유지
	(2)자동위치모드 진입시  MOSI.8, MOSI.2 동시 OFF시 현재 위치 유지(53)
	(3)동시이동모드 진입시  MOSI.9, MOSI.2 동시 OFF시 현재 위치 유지(63)
2) MOSI.15 통신 비상정지 알람비트 상태워드에 신규 반영함.
3) RFID 오프셋 적용 시점을 지연시키는 기능을 파라미터(ft-2.50)로 추가 (실제로 동작하는 부분 변경)
4) 동시이동 control task 추가, Set_SyncMotion_task(0); //MOSI.9
5) HW 비상정지시 감속 속도 시작은 속도명령을 기준으로 한다. 
   -현재 속도를 적용하면 리플이 많아서 가감속될 수 있음.
6) 자동위치모드에서 제2위치값이 적용안되는 것을 개선함
   -Update_PositionOld_PositionNegative 함수로 대체
7) 모니터링 변수 추가, (0x2AA1 ~ 0x2AB0, 161~176)
7) 기타사항
	LMS_VEL_STOP 상태에서 자동위치/동시이동 모드로 전환가능하도록 상태 추가, T3_50 속도정지(3)->자동위치대기(50)


///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.60 20241218 동시이동 시작 버전2, 관련 파라미터 추가
1) 토크제한 플래그(0x2A98 LMS_TorqueLimit_readyFlag)를 Status Word 비트14 에 할당함.
2) RFID 오프셋 적용 시점을 지연시키는 기능을 파라미터(ft-2.50)로 추가
   RFID Offset Apply Delay Time							 	0x2232		Ft-2.50 	0x2232	P0-2.50		Y	6018
3) 미러링 오브젝트 0x600A ~ 0x6013 SDO 쓰기 오류 수정
   PDO read/write는 모두 정상, SDO read는 정상, write시 메세지 오류하여 쓰기 안됨.
    원인은 WR_COMMAND 함수처리가 잘못되는 것으로 확인하여 WR_COMMAND_LMMT 로 신규 추가함.
4) 미러링 오브젝트 6005/6016/6017/6018 각축 추가
   222F/2230/2231/2232 각축 추가
   COE 카테고리 열거형 변수에 미러링 영역 추가
      CO_INDEX_LMMT_6000                             = 0x6000,	     //20241218 동시이동 미러맵
      CO_INDEX_LMMT_6800                             = 0x6800,	     //20241218 동시이동 미러맵
      CO_INDEX_LMMT_7000                             = 0x7000,	     //20241218 동시이동 미러맵
      CO_INDEX_LMMT_7800                             = 0x7800,	     //20241218 동시이동 미러맵
5) 내부 변수 이름 변경
      LMS_VEL_BUF -> LMS_VELPOS_BUF
6) XML 신규 오브젝트 추가
      222F/2230/2231/2232 각축 추가
      6005/6016/6017/6018 각축 추가

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.59 20241211 동시이동 시작 버전
1) RFID 오프셋 적용 시점을 일정시간 지연시키는 기능 추가(D7에 적용된 기능을 D8에 똑같이 적용함)
	일반모드/동시모드 400ms 시간 지연 후 오프셋 적용함.
	파렛오픈시 적용된 파렛오프셋 0으로 초기화함 (D8: BG처리, D7:Extint1처리)
	Update_TargetPositionWithDirection 함수를 LMMT control에서 항상 호출하도록 변경
2) 동시이동 관련
	상태머신, 엔코더룰 추가
3) 기타 수정사항
	1)내부 변수 이름 변경
	//Sync_PalletInMon_EA -> SyncEA_PalletInCnt
	//Sync_PalletInMon_EB -> SyncEB_PalletInCnt
	enc_normal_alarmreset -> IsDirectON_alramreset
	Set_EncDiffCompensator-> Set_Update_PosFbk_with_EncDiff
	Update_posfdk_latch   -> Set_posfdk_latch
	2) pr114 알람 삭제
	3) 세타값 설정하는 위치 조정
	4) 함수 모듈화
		void PROCESS_Homing_SwitchingRule(uI16 ax); 				//20241211 홈밍 스위칭룰
		void PROCESS_SyncMotion_SwitchingRule1(uI16 ax); 			//20241211 동시이동 스위칭룰1
		void PROCESS_SyncMotion_TriggerCondition(uI16 ax); 			//20241211 동시이동 트리거 조건 검사 블럭
		void PROCESS_SyncMotion_Enc_Delcnt_EA_EB(uI16 ax);			//20241211 동시이동 인코더 개별 속도피드백 계산
		void PROCESS_SyncMotion_PositionUpdate_EA_EB(uI16 ax); 		//20241211 동시이동 개별 인코더 위치값 업데이트
4) RFID사용시 파렛오프셋 검색 루틴의 조건 1.52 조건 추가
	if (pAx->pr341 & 0x00F0) { // ft-1.52 D1 Servo PalletOffset Apply Enable

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.58 20240903 위치정지 상태 + GO OFF 상태에서 위치값 변경시 속도명령 발생하여 파렛이 움직이는 현상 수정(D7 V1.20.10.81 대응) 
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
	코드 변경점: 위치명령의 변경 여부를 확인하는 코드를 통합함.         
	변경전:
   	  Update_TargetPosition_And_StatusChanging(0);
      lms_pos_dir = LMS_DIR_VALUE;
      Update_PositionOld_PositionNegative(0);
	변경후:
      Update_TargetPosition_with_PDOupdateTiming(0);

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.57 20240731 D7&D8 FW 기능 통합 버전(DCT전달)
1) 55, 56버준 수정사항중에 자간거리관련 잘못 수정된 사항을 원래대로 원복함.
2) XML버전을 FW 57버전과 통일함.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.56 20240625 55버전의 수정사항 중 필요없는 부분 삭제.
1) 모터정보의 자간거리 유효자리수 처리 원복함.
   D8에서는 3rd Party일때 다운로드된 모터정보를 사용하도록 command_SET 함수가 이미 수정되어 있음
   LMMT FW만 special하게 처리되어 있어서 수정함.(D5/D7 표준 모델은 정상)
2)부동소숫점 타입 자간거리변수로 정수형 ppr을 구할때 최소 mm단위로 스케일링하던 것을 사용하지 않도록 원복함
			  
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.55 20240618 모터의 자간거리값에 스케일 적용시 양자화 오차 발생되는 현상을 개선함.
1) 모터의 자간거리값에 스케일 적용시 양자화 오차 발생되는 현상을 개선함.
	->UDB로 모터 정보를 저장하고 로드를 반복하는 경우 발생 
	    입력값 30->29.9로 보임, 29.9를 저장하여 다시 Write하면 29.8로 보임.. 이렇게 0.5 보다 작을때 까지 계속 변함.
	->자간거리(Motor.Cycle_len)는 내부에서 1/10000배 스케일 변경하고 반올림 적용하여 정수로 보이도록 함.
2) ppr 계산시 최소 mm 단위로 스케일링하여 계산하여 FullTurn 값이 잘못 계산되는 현상을 개선함.
	->모터의 자간거리값에 스케일 적용시 양자화 오차 발생되는 현상을 개선하면서 발생한 side effect임.
   	->3rd Party 모터로 사용시 ppr 소숫점값에 의해 FullTurn 계산값 오차 발생하고 속도피드백 틔는 현상으로 나타남.
	->수정한 코드 
		pAx->Motor.Enc.ppr = (sI32)((pAx->Motor.Enc.Line_cnt * (uI32)(pAx->Motor.Cycle_len * 1000)) / 1000); // 최소 mm단위로 변경, x1000
		HalfTurn 변수 실수값으로 계산하도록 수정
			pAx->HalfTurn = (uI32)((FP32)(pAx->FullTurn+1)/2. + .5)
		Del_cnt 계산시 HalfTurn을 초과하는 경우 (FullTurn+1)값을 더하도록 계산식 수정.
			pAx->ENCODER.Del_cnt = pAx->ENCODER.Del_cnt - SIGN(pAx->ENCODER.Del_cnt)*(pAx->FullTurn+1)

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.54 20240613 DCT향 베트남 하이퐁 튜닝이슈 개선사항 적용외 최적화
1) 속도 감속시점 조정 기능(위치/자동위치 모드에 적용함)
	ft-1.54 (0:disable, 0 아닌 위치값을 입력시 등속에서 감속되는 시점을 임의 조정 가능)
	-속도 프로파일 생성기는 목표위치에서 위치스위칭 임계값 (position switching threshold)을 뺀 가상의 위치를 기준으로 
	  주어진 속도와 감속시간을 고려하여 감속 시점을 결정한다.
	-최종 목표위치 대비 남은 위치가 이 임계값보다 작을 때  위치모드로 전환된다.
2) 자동위치 모드에서 파렛이동을 위한 속도 피드백 선택 방법 누락된 부분 추가, 0x222A subindex1 (ft-2.42 D0)
3) 기타 사항
     홈밍변수 HomingEntry_direction 버그 수정
     엔코더 분해능에 따라 위치모드 전환 기준 변수. EncRes_25mm
     엔코더 분해능에 따라 게인 스케일 변수, EncRes_gainScale

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.53 20240524 자동위치모드 동작사양 개선 (DCT 남기혁, KNS대응) 
1) 자동위치 정지 후 PLC에서 비트8을 OFF시켰을 때 속도모드로 젼환되어 이동하는 현상을 막기 위한 개선 요청
       상태(13)에서 MOSI.1을 OFF하여도 현재 위치를 유지하는 것과 동일한 사양으로 동작하도록 해야 혼란이 없기 때문.
	기존: 상태(53)에서 MOSI.8 OFF시 MOSI.1의 값에 따라 속도(3)/(13)위치로 전환되었음.
	변경: 상태(53)에서 MOSI.8 OFF시 MOSI.1과 2의 조건에 따라 동작함.
2) 자동위치 모드  동작사양 수정 (LMS_POS_POS(13) 상태와 동일하게 동작하도록 수정함.)
      자동위치 정지(53) 후 MOSI.8=OFF && GO 비트의 on/off(에지) 발생시 MOSI.1 조건에 따라 속도(0)/위치(53) 상태로 이동함. (홈밍(20) 설정시 우선함)
      자동위치 정지(53) 후 MOSI.8=OFF && GO=ON 유지 조건일때 MOSI.1 조건에 따라 속도(0) 혹은 현재 위치(53) 상태 유지함.
      자동위치 정지(53) 후 MOSI.8=OFF && GO=OFF 조건일때 현재 위치(53) 유지
3) 자동위치 모드에서 파렛오프셋이 master postion에는 업데이트되지 않는 현상 수정 (99.50 수정사항을 자동위치 모드에서도 동일하게 적용)
       수정 : 자동위치 준비/초기 (50/51)단계에서 위치값 갱신
4) 자동위치 모드에서 위치명령 업데이트 안되는 오류 수정   
       코드상 A축만 정상 동작하고 B/C축 적용할 때 A축으로 잘못 적용되어 오류를 수정함.
   Update_TargetPosition_And_StatusChanging(ax);
   Update_PositionOld_PositionNegative(ax);

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.52 20240419 자동위치모드 개선, 홈밍 가감속 개선(DCT KNS대응) 
1) 자동위치모드 기능 동작 수정 요청
       위치완료 상태에서 자동위치모드로 변경시 파렛이 이동하지 않는 현상으로 후행 파렛 충돌현살 발생
       원인: 위치완료 상태(13)에서 자동위치모드 시작(MOSI.8+GO)명령을 전달하면 바로 자동위치모드의 위치완료(50-51-52-53)로 진행함.
      T13_50   위치완료(13)->자동위치모드 준비(50), STATUS 13은 서보온+GO 위치완료 상태이므로 T13_53으로 자동위치모드 위치정지 상태로 정상동작함.
       동작 개선:
           자동위치모드가 켜져있다면 항상 속도이동하여 현재 파렛을 배출함.
     MOSI.8=1인 경우 MOSI.1(Vel/Pos)에 상관없이 무조건 현재 파렛을 속도이동하여 배출하고 open시 LMS_STATUS 50으로 진행함.
       동작 수정(3가지 CASE):
      VP13_01 위치완료(13)->속도모드초기(01)->자동위치모드 준비(50)
      VP10_01 위치준비(10)->속도모드초기(01)->자동위치모드 준비(50)
      VP02_50 속도이동(02)->자동위치준비(50)
2) Homing Mode 수정
       홈밍트리거 이후 Offset값에 따라 속도 가감속이 발생하지 않는 현상 수정
  - LMS_STATUS 추가
     LMS_HOME_VEL_TRIG_STOP,  //25 홈트리거를 만났을때 이미 위치를 지났으면 정지하는 단계
     LMS_HOME_OFFSET_INIT,    //26 대기시간 100ms
     LMS_HOME_OFFSET_RUN,     //27 정지 후 Offset값 만큼 이동하는 단계
  - 홈트리거를 만난 후 Offset위치가 5mm (감속거리)이하이면 바로 위치 완료(28)로 진행함.
  - 예시
    CCW 방향(HomingEntry_direction=-1)
 	*조건식1 (5mm이내, A구역) : 위치명령>(트리거 - 5mm) 이면 감속정지(25)하고 위치완료(28)
 	*조건식2 (5mm이내, B구역) : 위치명령-(트리거 - 5mm) < (25mm이내) 이면 바로 위치완료(28) 아니면 속도이동(24) 하다가 25mm이내 들어오면 위치완료(28)
  2) MISO.3에 자동위치 모드(50~54)일때 상태 표시함.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.51 20240415 파렛오프셋 SDO으로 저장시 저장안되고 지워지는 오류 수정(DCT KNS대응)
1) 플래쉬 메모리에 cia402 오브젝트 저장 영역(16Kbyte)을 잘못 지우는 현상을 수정
   FlashErase 함수는 APP/FPGA 바이너리 대용량파일을 처리하기 위해서 만든 함수로 최소 0x10_0000 크기를 처리되어
   Cia402_Obejct 영역을 지울 때 뒤에 있는 Pallet_Offset 영역까지 지우게 된다.
   4page(16kybte) 단위로 설정된 Cia402_Obejct, Pallet_Offset 영역을 올바르게 지우기 위해 아래 함수로 대체함. 
   FlashErase -> EraseFlashPage
    -SIinitParamSave / SIinitEraseDictionary / SIinitDeletePartialDictionary 함수에 적용됨.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.50 20240321 파렛진입시 오프셋값이 항상 적용되도록 사양 보완(DCT V1.00.99.50_7.bin로 전달)
1) 위치모드에서 RFID 인식되어 파렛오프셋 계산되었으나 master postion에는 업데이트되지 않는 현상 수정 
   원인: Valid ON시점에 CPU1에서 CPU0으로 오프셋 검색 요청
	오프셋을 찾은 후 LMS_Control 시작하면 위치값 반영되지만 LMS_Control이 먼저 시작하면 오프셋 미반영됨
   수정 : 속도 준비/대기 (0/1)단계에서 위치값 갱신, 위치 준비/대기 (10/11)단계에서 위치값 갱신
       RFID_READ_OK 플래그는 데이터 복사가 끝난 후 ON 설정함. 

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.49 :20240104 0x8000 모니터링 안되는 현상 수정 (DCT전달)
1) 0x8000 모니터링 안되는 현상 수정
2) 파렛오픈시 적용된 파렛오프셋 0으로 초기화함

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.48 :20231228 RFID 처리에 따른 파렛오프셋값 표시안되는 현상 수정 (DCT전달)
1) 파라메터 pr160 용도 변경, RFID 디버깅 용으로 사용함 
     기존: 토크 제한기능 개선, Set_TorqueLimitFlags
     	파라메터 pr160 용도를 토크제한 음의 영역 설정값으로 변경, pallet ID -> LMS_Deadzone Distance Neg_Offset
2) RFID 인식되어 파렛오프셋 계산되었으나 master postion에는 업데이트되지 않는 현상 수정
                       
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.47 :20231206 RFID값이 0일때 예외처리 추가(DCT전달)
1) RFID값이 0으로 수신되는 경우 파렛오프셋이 미적용되도록 모두 0으로 초기화함.

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.46 :20231020 3rd party 모터 설정시 엔코더 마스킹 범위 확장 (42mm, 45mm RSA 엔코더 대응)
1) 엔코더 마스크 확장(10um기준 0xfff->1fff로 변경), NRSA제공

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.45 :20231014 DCT제공
1) 사용자맵 구성시 PDO 데이터 오프셋 버그 수정

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.44 :20231013
1) PDO 맵핑에 속도변수 추가후 속도변경 안되는 현상 수정(99.42버전의 누락분 추가)
                        
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.43 :20231012-1
1) 엔코더 A/B raw 싱글턴 데이터 PDO 맵핑(CSD7과 사양 통일)

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.42 :20231012
1) 파라메터 저장명령(0x1010:01,0x1010:03)시 파렛오프셋 자동저장 기능 추가(DCT요청사항, D7과 기능동작이 동일하도록 통일함)
2) 속도.가감속 PDO 맵핑 기능 추가 (0x2123~0x2128, 각축)

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.41 :20230925 3rd party 전기가 설정 지원
1) LMS custom 리니어 모터로 설정시 홀오프셋 0값으로 적용되는 버그 수정
2) Monitor -> Mechanical Angle에 홀옵셋 값을 표시하도록 수정

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.40 :20230920 
1) EtherCAT Z상출력 관련 수정, EtherCAT_Z
   -EtheCAT으로 출력되는 시간 수정, 3.2ms->2ms (D7과 동일함) 
   -ft0.08=777 입력시 SRDY 신호로 출력함

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.39 :20230703 속도피드백 필터사용 옵션 검증완료, DCT 제공
   LMS control 처리부에서 필터 초기값 설정은 LPF를 MAF필터로 대체함.
   LMS control 처리부에서 속도/위치에서 필터 초기값이 설정된 이후 구동시점에 필터계산을 수행함.
1) 속도피드백 주파수 1~3Hz의 작은 값에 대해서 최소값 4Hz로 적용
2) 파렛이동을 위한 속도 피드백 선택 방법, 0x222A subindex1 (ft-2.42 D0), Analog current command 에 신호맵핑됨
 	0 : raw data (속도명령)
 	1 : Moving Average Fiilter
 	2 : 1st LPF-> MAF적용
 	3 : 2nd LPF-> MAF적용
 	4 : Velocity Cmd Following Method (속도명령 추정)
3) 속도제어기에 사용되는 속도피드백 선택 방법, 0x222A subindex2 (ft-2.42 D1), Analog velocity command 에 신호맵핑됨
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

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.38 :20230630 속도피드백 속성 변경
1) 동작중에도 속도피드백 필터를 변경할 수 있도록 함.
	- ft-2.40/ft-2.42/ft-2.43/ft-2.44 

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.37 :20230627 Rsware와 용어 통일 
1) 0x2137 서브인덱스 이름은 "LMS" 키워드를 붙이지 않음. 파라메터는 기본 "LMS"를 붙인다.
2) 20230628 D7에서 D8로 오브젝트 번호 변경되면서 런펑션의 위치 잘못되는 오류 수정
   0x3000~0x3FFF -> 0x2C00~0x2FFF/3C00~3CFF/4C00~4CFF    


///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.36 :20230626 제2위치 티칭값 B/C축 설정잘못되는 오류 수정 
1) 1위치/2위치 상호 변경시 old 값을 초기화시킴
2) 속도피드백 2차 LPF 시정수 수정, BQ_SetCoef 함수 호출시 MPI->MPI2

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.35 :20230623 XML 수정 완료
1) OBJ 0x2114/0x2115/0x2116 각축 모두
2) OBJ 2137 D2 D3 각축 추가
   OBJ 2332/2333 각축 추가
   OBJ 0x2209 ~ 0x220F 각축 추가
   OBJ 0x2228 ~ 0x222C 각축 추가
3) 속도에러 표시방법 가변
	pr242 D1에 따라 속도제어에 적용되는 속도피드백 값을 표시함. ((속도명령-필터링된 속도피드백)
4) 속도피드백 시간상수 수정 
   BQ_LmsVnLpf: TS_VEL*2 ->TS_VEL

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.35 :20230621 (XML 수정전 임시버전)
1) 신규 파라메터 추가 및 최대값 확장
    /* LMS 신규 기능 대응 from pr240 ~ pr246 20230621 */      
	{/*Pr-2.40*/      0,      0, {0, 0,0,0,0,0,0,48}},//Range:0_10000         //Velocity Feedback Filter Cutoff Frequency
	{/*Pr-2.41*/      0,      0, {0, 0,0,0,0,0,0,48}},//Range:0_10000         //Jerk Acc/Dec Time
	{/*Pr-2.42*/ 0x0000, 0x0000, {1, 1,0,0,0,0,0,16}},//Range:0_0xffff        //Velocity Feedback Select Method
	{/*Pr-2.43*/      0,      0, {0, 0,0,0,0,0,0,27}},//Range:0_100           //Velocity Cmd Following Threshold
	{/*Pr-2.44*/      0,      0, {0, 0,0,0,0,0,0,27}},//Range:0_100           //Encoder Delta Threshold for Motor Stop Detection
	{/*Pr-2.45*/ 0x0000, 0x0000, {1, 1,0,0,0,0,0,16}},//Range:0_0xffff           //Smart Tuning Method1
	{/*Pr-2.46*/ 0x0000, 0x0000, {1, 1,0,0,0,0,0,16}},//Range:0_0xffff           //Smart Tuning Method2
	Pr2Max	35->46
	Pr3Max  49->51
2) 오브젝트 수정사항
    -타입 수정
	0x213B/0x213C UDINT->DINT, EA/EB 불감대 위치 오프셋
	0x313B/0x313C
	0x413B/0x413C	
	-신규
	LMMT_Encoder Version    	    MDM-66	0x2A42 gEncVer_Object
	LMMT_VC_Proportional Monitor    MDM-67	0x2A43
	LMMT_VC_IntegralSum Monitor     MDM-68	0x2A44
3) 기타 수정
	임시로 사용하던 파라메터 pr117 -> pr242 D0 로 대체
	임시로 사용하던 파라메터 pr118 기능삭제하여 원복함
	임시로 사용하던 파라메터 pr119 -> pr244로 대체
4)엔코더 버전읽기 추가
	pr345 Encoder ID가 2(RSA encoder only) 일 때 Read_Encoder_Version 함수에서 처리
	0x2A42 에 반영됨.
4) 위치 정지시 0속도 제어로 정지하도록 개선함
   -서보온상태에서 파렛이 진입할 때  위치 정지명령이면 프리런되는 현상 개선

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.34 :20230619 
1) 역주행 방지 로직1/2 추가, 
    -로직1: 스위칭동안 엔코더 1주기 이전의 증분값을 적용하는 방식
    -로직2: 스위칭동안 속도계 1주기 이전의 값을 적용하는 방식
2) FOE를 이용한 FW 다운로드시 빌트인 표시 개선
   -다운로드시 시작시 "GO", 완료시 "done", 다운로드 에러 발생시 "Error"
   -FOE를 이용한 다운로드시 자동 드라이브 리셋이 되지 않으므로 새로운 버전을 적용할려면 반드시 전원 on/off를 해야 한다.
   -FW 업데이트가 한번이라도 완료된 상태이고 전원을 끄지 않았다면 "done" 표시는 유지된다.
   -FW가 업데이트되었고 전원을 꺼지 않았은 경우 사용자가 알수 있도록 오브젝트(터치프로브 함수)에 표시함
   -FOE로 FW업데이트 되었음을 알려주는 상태 플래그, MB_firmwareUpdatedFlag = FALSE;
3) Z상 출력 기능 추가, pAx->EtherCAT_Z
4) 모니터변수 이름 변경
			sI32 MON_Master_DF3;
			sI32 MON_Slave_DF7;
			sI32 MON_ID3_Enc;
			sI32 MON_ID5_Enc;
			sI32 MON_Master_Enc;
			sI32 MON_Slave_Enc;

			uI08 MON_enc_posvalid;
			uI08 MON_enc_zphase;

			uI08 MON_ID3_halluvw;
			uI08 MON_ID5_halluvw;

			uI32 MON_ID3_ExtPHSensor;
			uI32 MON_ID5_ExtPHSensor;

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.33 :20230420 
1) 속도피드백 평균횟수 변경 : 4->8 ((DCT요청사항) 

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.32 :20230321 
1) 확장센서는 고정자리에 맵핑되도록 사양 결정 (DCT요청사항)
2) 0x600A A만 이상동작하는 현상
3) XML 문법오류 수정
		
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.31 :20230214
1) PDO에 등록되지 않은 오브젝트가 있을 경우 값이 0으로 보이는 현상 수정
   -6000번지대 PDO 등록시 APPL_GenerateMapping, APP_VarOutputMapping 함수에서 처리해야 함.
 
///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.30 :20230213
1) 2차 위치값이 PDO로 동작할 때 LMS_STATUS가 잘못 동작하는 버그를 수정함.
2) 오브젝트 인식안되는 현상 수정, 0x3331
3) 위치값 오브젝트 6000번대에 신규 추가
4) 내부변수 추가, pr348, pr349 
3) 엔코더값 모니터링 변수 D7과 동일하게 처리함
   ID3_Enc, ID5_Enc

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.29 :20230119
1) ft-1.22=1인 경우에만 RFID값 엡데이트되도록 수정함. 
2) 2차 위치값  파라메터 추가, ft-3.48, ft-3.49
	MOSI.13 비트 (0: 1차 위치값, 1: 2차 위치값)
3) PDO 맵핑 최종: A축기준
      0x2121: LMMT_Positive Distance/Position
      0x2122: LMMT_Negative Distance/Position
      0x2123: LMMT_1st Velocity
      0x2124: LMMT_1st Acceleration
      0x2125: LMMT_1st Deceleration
      0x2126: LMMT_2nd Velocity
      0x2127: LMMT_2nd Acceleration
      0x2128: LMMT_2nd Deceleration
      0x2330: LMMT_2nd Positive Distance/Position
      0x2331: LMMT_2nd Negative Distance/Position

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.28 :20230117
1) 엔코더 A/B을 값을 PDO로 업데이트 추가
	0x2AA9 LMS_ID3_Enc   
	0x2AAA LMS_ID5_Enc   
	0x2AAB LMS_Master_Enc
	0x2AAC LMS_Slave_Enc 

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.27 :20230117
1) 속도피드백 계산방식 개선, 이동평균값 적용
   ft-1.17=1일때만 적용
   cpu1에서 속도평균값 계산
2) cpu1에서 평균함수 쓸수 있도록 추가
	MAF_Init/MAF_Create/MAF_Update
3) 정지상태 판단 변수 타입 수정
   uI16-0>sI16 Pos_oscill_cnt

///////////////////////////////////////////////////////////////////////////////
0) V1.00.99.26 :20230116
1) 확장근전센서 초기화 오류 수정, Ext_Proximity_Sensor
2) 토크 제한기능 개선
     파라메터 pr160 용도를 토크제한 음의 영역 설정값으로 변경, pallet ID -> LMS_Deadzone Distance Neg_Offset
   EA 진입/배출시 설정값: ft-1.59 
   EB 진입/배출시 설정값: ft-1.60
   0x213B	Axis-A LMS_EncA Deadzone Offset	A축 LMS EA 기준 불감대 위치 오프셋, DINT
   0x213C	Axis-A LMS_EncB Deadzone Offset	A축 LMS EB 기준 불감대 위치 오프셋, DINT

             
///////////////////////////////////////////////////////////////////////////////
20230106, 천병훈
0) V1.00.99.25 :20230106
1) 현재 위치값이 위치 티칭값과 같은 값으로 보이도록 수정, DCT요청사항 고정도
     현재 위치(0x6064)=현재 위치값(티칭값 기준) - 파렛오프셋값
     타켓위치(0x607A)=위치 티칭값
2) PHS 근접센서값을 DF2에서 읽어오도록 처리방법 변경(DCT요청사항)
   기존)ID 3이면 DF2, ID5이면 DF6에서 읽는데 RFID가 DF6에 할당되어서 PHS 신호가 보이지 않음.
3) 확장 근접센서(Ext PHS) 오브젝트 0x2A9A 사양추가
   엔코더A 접점: DF3 bit6/5, 0x2A9A bit1/0
   엔코더B 접점: DF7 bit6/5, 0x2A9A bit9/8

///////////////////////////////////////////////////////////////////////////////
20221207, 천병훈
0) V1.00.99.24 :20221207
1) Encoder 선택옵션 추가, ft-3.45
   0: DCT encoder, 1: others
2) 모션 변경점, ft-3.45 설정에 따라 DCT special처리된 부분을 D7 & D8_M 동일하게 적용
	1)속도준비상태에서 스텝기동시 현재 정지 위칭위치를 기준으로 함.
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
	2)속도기동 정지후 스텝이동시 현재 위치로 기준점 변경
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
	3)위치기동중 정지후 스텝이동시 현재 위치로 기준점 변경
		이전: LMS_Trigger_pos = pos_fbk;
		new: LMS_Target_position = pos_fbk;
3) 엔코더 양간오차 적용 기능 버그 수정
4) z상 출력 Flag.Zout
5) 위치모드에서 속도운전중에는 속도변경되도록 수정. (DCT 김혜찬 전임 요구사항)

20221202, 천병훈
0) V1.00.99.23 :20221202
1) ESI파일 수정
   DT2A3D 수정, CTT Error: 'DataTypesKey' Keyref의 키 시퀀스 'DT2A3D'이(가) 다른 키를 참조하지 못했습니다.,
   0x3A3E 수정, Offline dictionary! (동일한 키를 사용하는 항목이 이미 추가되었습니다.);
   0x2A38/0x3A38/0x4A38 이름변경: LMMT_Deadzone flag -> LMMT_Initial Pos
   0x413C 수정, UDINT->DINT


20221130, 천병훈
0) V1.00.99.22 :20221130
1) TxPDO default map 구성 변경
	{{9, {0x60410010,0x60010010,0x60640020,0x606C0020,0x60770010,0x60610008,0x10010008,0x60040020}},                /*TOBJ1B00*/ //20221130 add RFID_DF6
2)부하율 관련 추가사항
   평균부하율은 ms단위로 계산되도록 수정
   이동평균은 sec단위로 계산되도록 추가
  MDM 44 0x2A96	LMS Motor Utilization RMS
  MDM 45 0x2A97	LMS Motor Moving Utilization RMS

20221129, 천병훈
0) V1.00.99.21 :주석 20221129
1) RFID값 Valid ON 구간에만 적용하고 오픈상태는 0으로 초기화함.

20221128, 천병훈
0) V1.00.99.20 :주석 20221128
1) PDO 업데이트 방식 : DC sync 인터럽트에서 동작 (검색어: PDO_UPDATE_ISR)
  1.00.99.19에 0으로 임시 적용된 것을 1로 변경함.
2) 자동속도위치 모드 추가
	LMS_VP_READY  =  50, // VP mode
	LMS_VP_INIT,
	LMS_VP_VEL,
	LMS_VP_POS,
	LMS_VP_STOP,
3) warning 제거
	extern void free(void *ptr);
	extern void *malloc(size_t size);
4) 3rd-party 모터 설정시 Motor Model(0x2001)이 undefined로 보이는 현상 개선
     사용자 입력 문자로 보이도록 수정함


20221126, 천병훈
0) V1.00.99.19 :주석 20221126
1) 전류제어기 설정 레지스터 수정: Theta in FW

20221125, 천병훈
0) V1.00.99.18 :주석 20221125_1
1) PDO_UPDATE_ISR 0로 설정: 
   PDO_UPDATE_ISR 1로 설정: 효과가 없어서 원복함. 
2) RFID 쓰레기값 버그 수정
	
20221125, 천병훈
0) V1.00.99.17 :주석 20221125
1) RFID 쓰레기값 처리 :ft-3.41 D1 disable인 경우는 0 으로 초기화시킴
2) MOSI 명령에 대한 loopback 오브젝트 추가: 0x6002/0x6802/0x7002
3) 사용자맵에 LMMT_RFID_DF6 신규추가.    
	0x6004/0x6804/0x7004
          
20221124, 천병훈
0) V1.00.99.16 :주석 20221124
1) 네트워크 검색시 LMMT Mode로 구성되도록 수정
   0xF050 Detected Module Ident List : 0x00019800 -> 0x0069800
2) 파렛오프셋 모니터값 읽을때 A/B/C 전축 보여지도록 수정
	#ERD*
	

20221122, 천병훈
0) V1.00.99.15 :주석 20221122 DCT협의 사항 반영 (w/d DCT 남기혁 책임)
1) ID 인식 failed 된 경우
      RF ID값은 맞지만 테이블에 등록이 안된 경우
      RF ID값을 수신하지 못했거나  잘못된 경우
      -> 외부 진입시 ID 검색 실패하면 오프셋 0 적용(초기화)
2) 단위 수정
   토크부하율, 0x6077 단위 재확인
      CSD7 BNM : 0.001[Amps] (기존 장비의 호환성을 위해 수정 못함)
      D8, D8_M : 0.1[%]
   속도피드백, 0x606C 단위 변경
      CSD7 BNM : [pps]
      D8_M : [mm/s] , 속도명령 파라메터 단위와 피드백을 맞추기 위함
     
20221122, 천병훈
0) V1.00.99.14 :주석 20221122
1) LMS_Control 수정 2차
     급정지 기능 추가

20221120, 천병훈
0) V1.00.99.13 :주석 20221120
1) LMS_Control 수정 1차

20221120, 천병훈
0) V1.00.99.12 :주석 20221120
1) 0x8000 subindex 18개로 확장(A/B/C 각각 6개씩 할당)

20221119, 천병훈
0) V1.00.99.11 :주석 20221119
1) pallet monitor 3축 확장 사양변경
   #ERD* <--- 축번호 적용
   XML 0x8000에 서브인덱스 영역으로 축 구분
   		01 ~ 06: A축
   		07 ~ 12: B축
   		13 ~ 18: C축

20221118, 천병훈
0) V1.00.99.10 :주석 20221118
1) Homing시 원점 위치 클리어되도록 사양 변경
2) RFID PDO 맵핑 추가
	uint32 g_RFID[3]={0,0,0};
	int32 g_Poffset[3]={0,0,0};

20221118, 천병훈
0) V1.00.99.09 :주석 20221118_1  XML수정
1) IO 맵핑 추가


20221118, 천병훈
0) V1.00.99.08 :주석 20221118 XML수정
1) 모니터링 오브젝트 수정/추가
    0x2A3B ~ 0x2A42        
    0x2A96 ~ 0x2A9A
      이름 변경: LMMT_Dead Zone Flag ->LMMT_Initial Pos

20221117, 천병훈
0) V1.00.99.07 :주석 20221117 RFID 테이블 검색 동작 확인.
1) EWR 음수처리 버그 수정
2) ID 검색기능 테이블 인텍스 잘못되는 버그 수정
3) PO-1.60 값에 따라 RFID 검색 기능 구분
4) 모니터링 변수 수정

20221116, 천병훈
0) V1.00.99.06 :주석 20221116
1) 공유메모리 파라메터 추가
   po-1.51~po1.60, po3-40~po3.47
2) 엔코터 위치값 무효인 경우 알람 발생 추가 (DCT special)
	po1.14 = 0 일때 발생함
	#define ErrCode_EncEEPROM        0x38  // E.031 ENCPE
3) RFID 기능 추가
	Get_PalletOffset
4) LMS_Position_offset 처리 기능 추가

20221115, 천병훈
0) V1.00.99.05 :주석 20221115
1) 변수 정리, 공유메모리 사용하도록 변경
   LMS_Enc_data_old
     서보off 체크함수 수정
   pos_fdk_latch
   PALLET_L ->LMS_Valid_L
   PALLET_R ->LMS_Valid_R
2) E.112 조건 추가 (DCT special)
   - ft-1.51 D0 Enable일때 근접센서 조건으로 발생함
   - #define ErrCode_EmerStop        0x54  // E.112 ESTOP

-------------------------------------------------------------------------------------------------
20221114, 천병훈
0) V1.00.99.04 :주석 20221114
1) LMS_STATUS_VARS 구조체 정의
     FW에서 사용하는 변수 추가 
3) ESI_Builder.c 수정
     파렛오프셋 오브젝트 추가 : 0x8000 ~ 0x8258

20221114, 천병훈
0) V1.00.99.03 :주석 20221114
1) 파렛오프셋 플래쉬 저장/읽기 기능 추가

20221111, 천병훈
0) V1.00.99.02 :주석 20221111 hot_fixed
1) CW 위치정지 후 CCW 홈밍시 반대방향으로 진행하면서 이탈하는 현상 수정
   Backward_switch 명령 처리를 LMS_Control에서 수행하도록 변경함
2) E.106알람발생 후 리셋하고 서보온+기동운전시 모터구동 불가 현상 수정
   RUN08 호출시 Enc Type 설정하는 부분 추가함. 
   LMS_SetDCTEncoder() 함수 추가


20221111, 천병훈
0) V1.00.99.01 :주석 20221111
2) 파렛오프셋 기능을 위한 구조 추가
   PALLET_OFFSET 정의문 추가
   ETcia402_lms.c, ETcia402_lms.h 추가 

-------------------------------------------------------------------------------------------------
20221011, 천병훈
0) V1.00.00.00 :주석 20221011 양산버전
1) 긴급정지관련 파라메터 영역만 추가함.(RSware와 호환성을 위해서 넣음)
   Pr-3.46, 0x2139 "LMMT_Emergency Stop Deceleration" Range:0 ~ 2147483647, mm/s^2
   Pr-3.47, 0x2140 "LMMT_Emergency Stop Torque Limit" Range:0 ~ 500 [%], %
-------------------------------------------------------------------------------------------------

20220628, 하대경, 천병훈
0) V0.1.99.05 :주석 20220628
1) DCT Object 예약영역 확보
   A axis : 0x2A31~2A42
   B axis : 0x3A31~3A42
   C axis : 0x4A31~4A42
2) 상위 제어 속도 모드에서 부하 정지판단시 속도명령이 0에서 시작하도록 변경.
   -> In Positon만으로 부하정지판단 후 곧 이어 속도 명령 인가했을 때, 0이 아닌 속도 Feedback이  명령 생성에 영향을 미치는 것을 방지하기 위함.
   -> Ft 1.19로 정지판단을 위한 위치 흔들림 크기 폭 설정 가능 (Ft 1.19 Default값 20->5 변경)
3) PWM 가용 폭 조정 ( E.057 방지 )
   -> Peak, Valley 추가 PWM 가용 영역을 사용하지 않고, 기존 Min-Max 영역으로 제한

20220621, 하대경, 천병훈
0) V0.1.99.04 :주석 20220621
1) 제품코드 변경: 500->600


20220614, 하대경, 천병훈
0) V0.1.99.03 :주석 20220614
1) ft-4.23 단위와 범위 수정
	0~60000, unit [ms], DINT타입
2) 오브젝트 관련 수정 (A/B/C축 모두 해당함)
	DT2329, ro->rw
	0x213C, UDINT->DINT
	0x2133
	0x2137
	0x2329
	0x6000, INT16->UINT16
	0x6001, INT16->UINT16
	0x3A02 누락된거 추가

20220607, 하대경, 천병훈
0) V0.1.99.02 :주석 20220607
1) 속도제어루프 별도 생성
2) 파렛1개로 동작시 진입후 배출때 전류명령 부호가 반대로 나타나는 현상을 수정함.
     파렛없을때 적분게인 초기화 수행.

20220606, 하대경, 천병훈
0) V0.1.99.01 :주석 20220606
   V1.02.20.14를 버전만 바꿈
   DCT 사내 테스트 시작버전용

20220603, 하대경, 천병훈
0) V1.02.20.14 : 주석 20220604
1) 물리신호 출력기능 추가: TG_ON,SRDY, P_COM, V_COM 
      LMS_IOMapping_TG_ON(ax); //MISO.1
      LMS_IOMapping_SRDY(ax); //MISO.2
      // LMS_IOMapping_P_COM(ax); P_COM 신호는 LMS_Control에서 업데이트함.
      LMS_IOMapping_V_COM(ax);
      LMS_IOMapping_NEAR_AND_PHS(ax); //MISO.8 ~ MISO.13 and MISO.14
      LMS_IOMapping_SALM();
2) 0x6060: -13 일때 LMMT모드로 자동 동작하도록 xml파일 수정
      InitCmd 에 HexBinary 타입으로 F3(0xF3= -13dec)
      MMCE로 스캔시 0x6060은 USINT값으로 243dec 으로 등록됨.

20220602, 하대경, 천병훈
0) V1.02.20.13 : 주석 20220602
** A축의 0x6060=-13이고 ft-0.0=13일때 A/B/C모든 축이 LMS 동작모드로 자동설정된다.
   A축 기준으로 LMS 동작모드를 설정해야 정상적으로 동작한다. 
      - B축/C축을 개별적으로 설정하더라도 A축을 따라감 
1) EtherCAT 0x6060(Modes of operation) 동작 모드와의 관계
   LMS(-13)모드 추가 : ECAT_PROFILE_LMS_MODE, master scan시 기본값으로 자동 설정됨(by XML)
                                  빌트인에 "n" 표시
   LMS 설정 오브젝트가 맞지 않는 경우 알람 발생, E.207
   UINT8-> SINT8 타입 변경
   - SINT8 타입 추가(ETecat_def.h)
   - Drive.h 수정, sI08 objModesOfOperationDisplay
   - ETcia402appl.h수정
       SINT8      objModesOfOperation
       SINT8      objModesOfOperationDisplay
	0x6502 supported operation mode는 수정하지 않음.(코드상 예외처리)
2) 신호수정
	MISO.4 : LMS_Valid_LM->LMS_Valid_MO
	MISO.14: LMS_EcatOut[ax].BIT.Near 삭제할 수 있는지 협의


20220531, 하대경, 천병훈
0) V1.02.20.12 : 주석 20220531
1) 기본모터 설정시 토크상수와 최대전류 0으로 나타나는 현상 수정
   - 3rd party로 선택시 ft-0.01의 모터호출 함수 수정
   - ft-1.20!=77일때 ft-0.01 기본모터 정보값(0x90000-> 0x11047:CSMT-04BR\) 설정되도록 수정
2) EtherCAT 0x6060(Modes of operation) 동작 모드와의 관계
   master scan시 기본값 CSV(9)모드
   CSP(8): LMS 운전 가능, 빌트인에 "F" 표시
   CSP(9): LMS 운전 가능, 빌트인에 "n" 표시
     다른 동작모드(CST/HM/PP)는 ft-0.0값이 변경되고 지원하지 않음. 
3) LMMT 알람기능 추가
   E.114, 과전류 알람: 전류피드백이 드라이브 최대전류를 초과하면 제한
   E.022, 과부하 알람: 순시전류값 정격의 110% 2초 유지시, 순시전류값 정격의 70% 30초 유지시 발생 


20220530, 하대경, 천병훈
0) V1.02.20.11 : 주석 20220530
1) ECAT 수정사항
     0x2D000010-> 0x60000010	LMMT MOSI
     0x2D010010-> 0x60010010	LMMT MISO
2) 긴급정지 정지상태의 속도 명령값(w_ref) 정수화 처리
    

20220527, 하대경, 천병훈
0) V1.02.20.10 : 주석 20220527
1) LMS_Control 83.01버전으로 변경

20220526, 하대경, 천병훈
0) V1.02.20.09 : 주석 20220526
1) 제품코드 변경: 0x02020001->0x02050001
2) 모델이름 변경: "_M" 추가
3) 긴급정지 기능 추가

20220521, 하대경, 천병훈
0) V1.02.20.08 : 주석 20220523
1) ApplicationPara 누락부분 추가
2) Valid A/B 신호 잘못된부분 수정
3) 기타
   LMS_Control에서 속도명령 변수 대체: w_command->w_ref
		pAx->w_ref = pAx->w_command; //20220520 속도명령 변수 대체
   PDO상 속도피드백 w_rad_flt 값 적용
     중복변수 삭제
      LMS_Dir_change (x) LMS_DIR_Change (OK) 


20220520, 하대경, 천병훈
0) V1.02.20.07 : 주석 20220520
1) Ecat IN/OUT 추가
   LMS_EcatIn, LMS_EcatOut 변수 선언
2) 위치/속도/토크 피드백 오브젝트 계산
3) jog direction수정
   정방향: CW(0)일 때 Dir=-1
   역방향: CW(0)일 때 Dir=1
4) CSV(동기속도모드) 일때만 LMS 모드로 구동됨
   0x6060(Mode of Operation)=0x09
    빌트인의 축A/B/C 문자앞에 "n"으로 모드 표시됨.   
2) 기타수정
   축변수 변경: MACRO_CIA402_SV_ON(counter) -> MACRO_CIA402_SV_ON(nAxis)

20220512, 하대경, 천병훈
0) V1.02.20.06 : 주석 20220516
1) LMS_Status 초기화 추가
2) Motor 최대속도 계산 추가
3) Acc/Dec Monitoring 계산 추가

20220512, 하대경, 천병훈
0) V1.02.20.05 : 주석 20220512
   XML : RSA_D8_M_Series_V1.2.20.5_00_20220513.xml
1) ECAT 수정사항
  1-1) LMMT 기본맵 구성 정보
     - user mode 사용(0x1700, 0x1B00), CSD7과 동일함
     - User Mode->LMMT Mode 이름변경
     - DefCiA402ObjectValues에 정의된 값 수정

     RxPDO (0x1700)
        0x60400010	Control Word
        0x2D000010	LMMT MOSI
     TxPDO (0x1B00)
        0x60410010	Status Word
        0x2D010010	LMMT MISO
        0x60640020	Position Actual Value
        0x606C0020	Velocity Actual Value
        0x60770010	Torque Actual Value
        0x60610008	Modes of Operation Display
        0x10010008	Error Register
        0x603F0010	Error Code
  1-2) A/B/C축 슬롯에 대한 ModuleIdent 개별할당
  	Axis-A : x619800
  	Axis-B : x719800
  	Axis-C : x819800
  1-3) PDO 업데이트 방식 : DC sync 인터럽트에서 동작 (검색어: PDO_UPDATE_ISR)
  	A: 0x2D00, 0x2D01
  	B: 0x3D00, 0x3D01
  	C: 0x4D00, 0x4D01
  1-4) SDO 오브젝트 범위
  	A: 0x2133 ~ 0x2137, 0x2328 ~ 0x232D
  	B: 0x3133 ~ 0x3137, 0x3328 ~ 0x332D
  	C: 0x4133 ~ 0x4137, 0x4328 ~ 0x432D
2) TESTRUN 정상동작

20220510, 하대경, 천병훈
0) V1.02.20.04 : 주석(20220510)
1) JOG/AT/TESTRUN 기동시 LMS_Dir_value 값이 속도명령에 따라 결정되도록 변경

20220504, 하대경, 천병훈
0) V1.02.20.03 : 주석(20220504)
1) LMS 파라메터 추가
	ft-1.33 ~ ft-1.60
	ft-3.40 ~ ft-3.45
2) A/B/C축 엔코더 스위칭 동작
3) Jog 모터 구동 동작
   가감속은 안됨.



20220427, 하대경, 천병훈
0) V1.02.20.02 : 주석(20220427)
1) ft-7.02 500
2) 함수 변경: abs함수 ->labs로 변경
3) 엔코더 통신부 안정화 처리
     CPU1 인터럽트 Exception 현상 해결
     B축 ID 5 위치에서 3/5 교대되는 현상 수정

20220420, 하대경, 천병훈
0) V1.02.20.01
1) 테스트중

20220415, 하대경, 천병훈
0) V1.02.00.00 -> V1.02.20.01
1) Ft 0.00 Min-Max값 수정
2) LMS Motor 인식 전 Mode.DriveType을 Ft0.00으로 update하도록 수정.
3) Reset 시, 자동으로 Ft0.00값 1(Follow None)로 만들어주는 Code 주석처리.
4) Ft 1.20,21 Black Box관련 값 update구문 주석처리
5) position overflow 임시 삭제
6) 엔코더 A에서 A&B 구간될때 Exception 발생하는 현상 해결
   DMAReg5810 대신 Reg5810을 읽도록 수정하여 정상 동작확인함.
7) CPU1에 ExceptionHandler 추가
   -RegisterHandlers
8) LMS_latch_pos_flag 추가
   E106 엔코더 관련 알람리셋시 enable되고 inner_routine에서 동작함.
9) 알람리셋 할때 SetNewMotor가 호출되므로 모터파라메터 관련함수도 호출되도록 추가함.
   -LMS_SetMotorParaRelatedVariable
10) 위치 모니터링 변수 수정
   MDM   4: position feedback, following position
   MDM  24: Motor Feedback Position
11) UltraEdit를 사용자를 위한 프로젝트 추가
   -CSD8_M_LMS_Ultraedit_project.prj
   -D8_M_LMS_ctag.txt
   
20220406, 함년근
////////////////////////////////////////////////////////////////
0) File name : D8_LMS_V01_02_00_00_20220406_1
1) 컴파일 에러 발생부분 디버깅(해결완료)

20220330, 함년근
////////////////////////////////////////////////////////////////
0)LMS 초본생성
1)LMS F/W 설계 및 적용
1)LMS Motor drive 부분 Migration


20211214, 최연범
////////////////////////////////////////////////////////////////
0) V1.01.00.02 -> V1.02.00.00 MDM99: 20211214
1) while(FlashBlockPt[ax]) 수정, CheckFlashStatus()함수 호출 위치 변경에 따른 조치

20211207, 최연범
////////////////////////////////////////////////////////////////
0) V1.01.00.01 -> V1.01.00.02 MDM99: 20211207
1) FlashBackup() 함수 내  2ms 활성화 확인 조건문 삭제
2) Run09_AlarmHistoryClear() 함수 내 FlashBackup()함수 중복 호출 삭제
3) User PDO 4byte Data Access 방법 수정(CSD7_S)
4) SDO Data Access 방법 수정(CSD7_S)
5) Command PDR Access 방법 수정(CSD7_S)

20211123, 최연범
////////////////////////////////////////////////////////////////
0) V1.01.00.01 -> V1.01.00.01 MDM99: 20211123 (MDM만 변경)
1) Smart Tuning Mode 2,3 Velocity Limit 이상 현상 수정

20211122, 최연범
////////////////////////////////////////////////////////////////
0) V1.01.00.00 -> V1.01.00.01 MDM99: 20211122
1) CheckFlashStatus()함수 호출 위치 변경 : 2ms ISR -> Background
2) Smart Tuning Mode 1 Velocity Limit 이상 현상 수정

20211103, 최연범
////////////////////////////////////////////////////////////////
0) V1.00.99.03 -> V1.01.00.00
1) F/W Release

20211103, 최연범
////////////////////////////////////////////////////////////////
0) V1.00.99.02 -> V1.00.99.03
1) KMTK BT Encoder 내부 코드 변경

20211101, 최연범
////////////////////////////////////////////////////////////////
0) V1.00.99.01 -> V1.00.99.02
1) RSMA, RSMP 관련 수정
   - 전류 방향 누락 수정
   - Encoder 방향 누락 수정

20211026, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.52 -> V1.00.99.01
1) RSWare Oscilloscope Buffer 변경 : 300 -> 1000(FW Version 1.01.00.00 이상부터 가능)
2) Parameter Range 변경
   - FT-2.30
   - FT-2.32
   - Ft-2.34
   - Ft-2.35
   - Ft-3.34
   - Ft-4.37
3) CMD_SF1, CMD_SF2 추가
   - CMD_SF1 수신 시 E.124 발생
   - CMD_SF2 수신 시 E.123 발생 후 Reset
4) TBL-i II 모델 추가
5) RSMA, RSMP 모델 추가
6) BlackBox 기능 수정
   - Ft-0.1.30 = 0 : 기존과 동일
   - Ft-0.1.30 = 1 : Auto Trigger Mode, E.009, E.030, E.037, E.111, E.123, E.124 저장하지 않음
7) Ft-6.04 Default Value 수정
   - 0x0508 -> 0x050A


20210901,신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.51 -> V1.0.0.52 MDM99: 20210831
1) FOE Download 시 부팅 에러 방지
   - ECappDownloadSelect()에서 lvalue, lvalueEnd 값을 Address1, Address2 둘 다 비교하도록 수정
2) FOE Error 표시 DispStr(0,"ERRxxx") 수정 및 추가


20210830,채정훈
////////////////////////////////////////////////////////////////
0) V1.0.0.50 -> V1.0.0.51 MDM99: 20210830
1) 64bit Absolute Homing 관련 Object 추가
   - 0x2534 Axis-A ECAT Abs Origin 64bit Offset(HIGH DWORD) 추가  (0x3534, 0x4534 동일)
   - 0x2535 Axis-A ECAT Abs Origin 64bit Offset(LOW DWORD) 추가 (0x3535, 0x4535 동일)

20210826, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.49 -> V1.0.0.50 MDM99: 20210827
1) TestRun ANF 수정
   - TestRunCnt 오타 수정
2) P0-6.04 Default 수정
   - 8 -> 0x0508 (D3~D2, D1~D0 분리해서 사용 중)


20210825, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.48 -> V1.0.0.49 MDM99: 20210826
1) OPTime 저장 조건 변경
   - 기존 : 5V Monitor 신호 확인
   - 변경 : Ctrl Power Loss 신호 확인
2) Read32BitOPTimeFromFlash() 함수 bug 수정
   - 한 개의 Section 만 사용하던 부분 수정


20210824, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.47 -> V1.0.0.48 MDM99: 20210824
1) Power Off 시 파라미터 저장 기능 삭제(SEMES 고속 OHT 대응)


20210820, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.46 -> V1.0.0.47 MDM99: 20210820
1) Vdc_fdk_flt(DC Link Volatage) Fitler 중복 사용 부분 제거
   - 기존 : OV, UV, Pre-Charge 기준 전압 = 50Hz LPF + 10Hz LPF (전류 제어기 제외, 전류 제어기 전압은 50Hz LPF 통과 후 전압 만 사용)
   - 변경 : OV, UV, Pre-Charge 기준 전압 = 50Hz LPF (전류 제어기 포함)
2) Pre-Charge Relay Off 기준 전압 수정 : 200V -> 185V
3) Pre-Charge Relay On Delay Time 수정 : 5ms -> 500ms (CSD7)   


20210820, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.45 -> V1.0.0.46 MDM99: 20210820
1) AqB Linear Hall Sensorless 일때 P0-2.25(Motor Overspeed Level) 설정을 command_LPS()에서 command_MEP() 함수로 이동


20210820, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.44 -> V1.0.0.45 MDM99: 20210820
1) PER_SW_26, PER_SW_52 수정
   - RSWare Oscilloscope 사용 시, Trig.Position의 값이 270이 되었을때 Command_MOX()함수가 호출되는 부분 제거


20210813, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.43 -> V1.0.0.44 MDM99: 20210813
1) Command_PDW() 버그 수정


20210812, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.42 -> V1.0.0.43 MDM99: 20210812
1) PER_FW_184 수정
   - SendBlockBox()에서 BlackBox_buf[+6] -> [+10] 수정


20210812, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.41 -> V1.0.0.42 MDM99: 20210812
1) Monitoring Object 0xnA00 ~ 0xnAFF 속성 변경   (n = 2,3,4)
   - NULL -> 0xffff (CSD7 동일)
2) Command_PDR() 함수 수정
   - 불필요한 비교문 삭제
   - Monitoring Object 비교문 Bug fix


20210811, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.40 -> V1.0.0.41 MDM99: 20210811
1) Bode Plot 재 측정 E.018 발생 현상 수정
   - RS Ware 수정 필요
2) V1.0.0.40의 0x2A53 bug fix


20210811, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.39 -> V1.0.0.40 MDM99: 20210810
1) RSWare Dvice Model, 0x1008 AA0로 표시 안됨 
 - COE_VarInit()에서 %01d->%01X로 수정
2) VendorSpecificObjDic[]에서 Build 시 Warning 제거 
 - 0x2A50~4까지 0xFFFF->Null 수정
 

20210810, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.38 -> V1.0.0.39 MDM99: 20210810
1) RSWare Oscilloscope Trigger Signal 에서 Digital Input#4,#5 동작 수정 
 - RollandTriggerMode()에서 (ExtInput[0].WORD)>>7)& 0x0018), ExtInput[0].WORD&0x0007로 수정 


20210809, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.37 -> V1.0.0.38 MDM99: 20210809
1) OHT 특주형 RS OscilloScope 기능 추가
   - Ft-0.1.30 = 1 인 경우 적용 가능
      - 최소 샘플링 주기 : 1ms
      - 최대 측정 시간 : 1min


20210809, 윤호성
////////////////////////////////////////////////////////////////
0) V1.0.0.36 -> V1.0.0.37 MDM99: 20210809
1) PER_FW 102 수정
   - ECAT Fault History 업데이트 이상 현상 수정
   - pAx -> Error 부분 error 변수로 수정


20210809, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.35 -> V1.0.0.36 MDM99: 20210809
1) PER_FW 165 수정
   - 0x1C32, 0x1C33의 subindex 3의 BitLength는 32bit이나 Data Type이 0으로 되어 잘못된 번지 인식
   - BitLength 참조 루틴 추가
2) COE_GetDicEntry() 함수내 불필요한 for문 삭제
3) Oscilloscope Size 300개로 원복


20210809, 신동석
////////////////////////////////////////////////////////////////
0) V1.0.0.34 -> V1.0.0.35 MDM99: 20210809
1) PER_FW_169 수정 : SelectDisplayData()에서 ~RegSW_GP_IN.WORD를 ExtInput[ax].WORD로 수정
2) PER_FW_177 수정
 - COE_GetIndexMapping(), COE_GetDicEntry()에서 if(pDiCEntry->Index < 0xF000 || pDiCEntry->Index > 0xF050) 추가
 - command_PDR(), command_PDW()에서 if(pDiCEntry->Index < 0xF000 || pDiCEntry->Index > 0xF050) 추가


20210809, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.33 -> V1.0.0.34 MDM99: 20210809
1) Detect Excessive Resonance Fault Disable 기능 누락 부분 수정
2) CSP Mode 사용시 Status Word의 12번 Bit 기능 수정 (TwinCAT NC 이상현상 수정)
   - 일반 모드 인 경우                  : 항상 High(CSD7 동일)
   - OHT Mode(Ft-1.30 = 1) : OHT Homing 완료 Bit로 사용
3) ANF_Core0.c 사용하지 않는 부분 삭제


20210805, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.32 -> V1.0.0.33 MDM99: 20210805
1) Ft-1.32 Tuningless Fine Gain 추가(SEMES OHT향 특주 기능)
2) Ft-2.09 기능 제거(임시 적용, Prest) -> Ft-1.30 으로 대체


20210805, 함년근
////////////////////////////////////////////////////////////////
0)V1.0.0.31 -> V1.0.0.32 MDM99: 20210805
1)FOE Error 발생 후 부팅 Error 방지하기 위해 ECappDownloadSelect()에서 Sector 마지막인 lvalueEnd == 0x77777777 확인 추가
2)ECappDownloadProcessing()에서 Flash Wtire 확인 후 0x77777777 Write 하도록 수정
3)PER_FW_170 수정 : Block Box 기능으로 Pr-6.04에서 BitField '1' 로 수정


20210805, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.31 -> V1.0.0.32 MDM99: 20210805
1)UsingParaSetup()에서 ApplicationPara(556, ax) 추가


20210726, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.30 -> V1.0.0.31 MDM99: 20210730
1) SEMES OHT향 특주 기능 사용시 Feedback Value Polarity 변경
2) SelectDisplayData() 함수 내 누락 부분 수정(EtherCAT 관련)
3) CheckAcPower220V() 함수 내 Bug 수정
4) Logic Version Upgrade(V01.00.00.00 -> V01.00.00.01), 지난 배포 시 Version Up 누락


20210730, 이현규
////////////////////////////////////////////////////////////////
0)V1.0.0.29 -> V1.0.0.30 MDM99: 20210730
1) 0x1010:2, 0x1011:2 이름 변경 (원래 지원안하는 오브젝트)
  - 0x1010:2 Store Communication Parameters   -> Reserved2
  - 0x1011:2 Restore Communication Parameters -> Reserved2
2) 0x10F0 이름 변경 (오브젝트의 기능을 명확히 하기위함)
  - 0x10F0   Backup Parameter Handling -> Backup CiA402 Parameter Handling
  - 0x10F0:1 Backup Parameter Checksum -> Backup CiA402 Parameter Checksum
  - 0x10F0:2 Backup Parameter Changed  -> Backup CiA402 Parameter Changed


20210727, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.28 -> V1.0.0.29 MDM99
1)210727 HDF Update 
2)pp_task_init()에서  Status 초기화 부분 수정


20210727, 이현규
////////////////////////////////////////////////////////////////
0) V1.0.0.27 -> V1.0.0.28 MDM99: 20210727
1) OBJ 0x2538 name에 Subindex name 추가 (CTT Warning 수정)


20210726, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.26 -> V1.0.0.27 MDM99: 20210726
1)pp_task_init()에서  Status 초기화 부분 추가
2)Digital I/O Default 값 수정 P0-0.16(0x0000), P0-0.21(0x0054)
3)P0-5.56(Analog Output Axis) 추가 및 P0-5.05, P0-5.06 수정
4)A_Moni.CH 에서 4,5,6,24 Position값 0.01배로 출력되도록 수정
5)P0-7.02 Drive Power Rate 이름 수정
6)CntEdmOut(STO Counter) 30(3ms)->200(20ms) 수정


20210726, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.25 -> V1.0.0.26 MDM99: 20210726
1) Parameter 추가
   - Ft-1.30 : SEC Enable(OHT 특주 기능)
   - Ft-1.31 : SEC Sliding Surface Slope(OHT 특주 기능)
   - FT-3.36 : In Position Hold Time
2) Smart Tuning Auto Setup Bug fix
3) 20kHz PWM Switching Frequency 적분 상수 계산식 수정
4) Bode Plot 측정 시퀀스 수정


20210721, 이현규
////////////////////////////////////////////////////////////////
0) V1.0.0.24 -> V1.0.0.25 MDM99: 20210721
** 7/16 파라미터 검토회의 결과에 따른 수정 **
1) ParaRangeTbl[41], ParaRangeTbl[83]이 같은 0_3000 범위로 되어있어 41로 통일함
  - ParaRangeTbl[83]은 0_3000 -> 0_30000 변경 (Ft-0.04에 사용)
  - ParaRangeTbl[83]를 사용하는 Ft-1.26(0x211A, 0x311A, 0x411A)은 ParaRangeTbl[41]로 변경함
2) Ft-0.04: max: 6000 -> 30000
3) 0x2004, 0x3004, 0x4004: max: 6000 -> 30000, ParaRangeTbl 45 -> 83
4) Ft-0.22[D2]: default 3 -> 1
5) Ft-3.00: default 0x1123 -> 0x1100
6) Ft-5.55: max 오류 수정 1600 -> 65535 (ParaRangeTbl 89 -> 54)
7) Ft-5.27~29: YUDO 관련 기능으로 사용안함으로 표시함 (관련 오브젝트는 존재하지 않음)
8) Ft-0.03 D3: max 9 -> 0
9) 0x211D, 0x311D, 0x411D 추가
10) 0x221B name 수정
11) 0x231F, 0x331F, 0x431F 추가(원복)
12) 0x2516, 0x3516, 0x4516 Subindex 정의
13) 0x2517, 0x3517, 0x4517: max 100 -> 200, ParaRangeTbl 27 -> 94

20210720, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.23 -> V1.0.0.24 MDM99: 20210720
1) PER_FW 161 수정
   - Application내 case 334 추가
2) PER_FW 162 수정 
   - VelocityCommandLimit()의 전달인자 수정
3) Red LED 동작 수정
   - 기존 : A Axis Error만 모니터링
   - 수정 : 전축 Error 모니터링(한 축이라도 Error 상태이면 Red LED 점멸)
4) Command_PDR 수정
   - 0x171n, 0x172n, 0x1B1n, 0x1B2n Bug 수정
5) Alarm Clear시, 0x603F 값이 Clear되지 않는 부분 수정
6) SEMES OHT대응 Code 추가(적용방안 검토 필요)


20210719, 함년근
////////////////////////////////////////////////////////////////
0)V1.0.0.22 -> V1.0.0.23 MDM99
1)INT_2ms_Routine() 이전에 main.c에서 FlashBackup() 실행으로 Init7,Init11에서 정지
 - FlashBackUp()에서 if(!Ative_2ms_routine) 추가 및  수정


20210719, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.22 -> V1.0.0.23 MDM99
1)P0-5.23 0~100 -> 0~200 수정 (100% 너무 작아 200% 변경)
2)P0-5.30 0~4 -> 0~1 수정 (20kHz, 10kHz 만 사용)
3)Motor.c 에서 PWM 20kHz에서 wn_cc 6kHz -> 4.5kHz : pAx->Kp_cc *= 1.5f, pAx->Ki_cc *= 0.75f 수정


20210719, 최연범
////////////////////////////////////////////////////////////////
0)V1.0.0.22 -> V1.0.0.23 MDM99
1)DigitalIO.c에서 pDrive->max_axis == 3 추가 (완제품 검사기 수정)


20210716, 이현규
////////////////////////////////////////////////////////////////
0)V1.0.0.21 -> V1.0.0.22 MDM99: 20210716
1) 0x2604/0x3604/0x4604 Axis-X Fault Detail Sampling Period 속성 오류 수정
  - 한 오브젝트에 두가지 기능이 포함되어 있어 속성을 변경 함(함년근 수석 협의 완료)
  - USINT(8) -> UINT(16)
  - Min: 1 -> 0, Max: 100 -> 0xFFFF, Default: 8 -> 0
  - ESI / OBJ&Para List 수정
2) User PDO Mapping OBJ 0x17x0, 0x1Bx0 초기화 값 수정
  - UserPDO Mapping OBJ 인 0x1700/0x1710/0x1720, 0x1B00/0x1B10/0x1B20 값이 Default 값으로 초기화 되지 않고 0으로 읽히는 문제가 있음
  - DefCiA402ObjectValues에 정의된 값 수정


20210713, 함년근
////////////////////////////////////////////////////////////////
0)V1.0.0.20 -> V1.0.0.21 MDM99
1)Outer_Handler에서 bBootMode == FALSE 추가 (Boot Download 중 Ponwer Off에서 충돌 방지)


20210713, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.20 -> V1.0.0.21 MDM99
1)단거리 자극검출 - Biss에서 지원 안 함 (command_CPD, command_BAS, command_BVF에서 P0-5.39 D0(1)면 Cancelled Return)
2)CAP_LIFE_LIMIT_HOUR(10년->8년), FAN_LIFE_LIMIT_HOUR(8년->6년)으로 수정


20210712, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.19 -> V1.0.0.20 MDM99: 20210712
1) Command_PDR() 수정 
   - 입력한 object의 데이터 표시
   - 축 정보는 무시함


20210712, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.18 -> V1.0.0.19 MDM99
1)단거리 자극검출 수정사항 반영


20210712, 함년근
////////////////////////////////////////////////////////////////
0)V1.0.0.18 -> V1.0.0.19 MDM99
1)완제품 검사기 ErrCode_STOinput(E.111) 발생 - CntEdmOut 12에서 30(3ms)으로 변경 (완제품 검사기 2ms에서 Fault 발생)
2)RSWare Digital Input 4,5 표시 안 됨 - SelectDisplayData(15) >>2)& 0x300 -> >> 7)& 0x18 수정
3)ETutility.c에서 0x2F09 & 0xffff 추가 (완제품 검사기 수정 전 대비)


20210709, 최연범
////////////////////////////////////////////////////////////////
0) V1.0.0.17 -> V1.0.0.18 MDM99: 20210709
1) Smart Tuning Mode 3 Position Error 계산 방법 수정
2) Command_PPE() 수정 : 다축 인식 가능
3) Core 1 ANF Bug 수정(최소 공진 주파수 Big fix)


20210709, 함년근
////////////////////////////////////////////////////////////////
0)V1.0.0.16 -> V1.0.0.17 MDM99
1)BinaryMaker ROM Image CheckSum 계산 추가


20210709, 신동석
////////////////////////////////////////////////////////////////
0)V1.0.0.16 -> V1.0.0.17 MDM99
1)P0-5.39 D0 (0:Method1(Normal),1:Method2(Short)) 기능 추가


20210708, 채정훈
////////////////////////////////////////////////////////////////
0)V1.0.0.15 -> V1.0.0.16 MDM99
1)V1.0.0.14 리뷰 사항 반영 수정
                            기존            변경
  Parameter 5.51 -> 5.52 ECAT Abs Origin 64bit Offset (High DWORD)
  Parameter 5.52 -> 5.53 ECAT Abs Origin 64bit Offset (Low DWORD)
  
2)Pr5Max 52->53 수정
3)Parameter List(엑셀 파일) 수정

20210708, 채정훈
////////////////////////////////////////////////////////////////
0)V1.0.0.14 -> V1.0.0.15 MDM99: 20210708
1) pAx->objVelocityActualValue 계산 관련 Linear Motor 사용 시 Command Velocity 와 Actual Velocity가 다름.
  -> 기존 계산 방법 (sI64)(-pAx->w_rad_flt * pAx->Kc_RAD2PULS / 60.f / pAx->fGearBox * (FP32)pAx->Full_resolution );
            바꾼 계산 방법 (sI64)(-pAx->w_rad_flt * pAx->Kc_RAD2MPS / pAx->fGearBox * (FP32)pAx->Full_resolution );
      라디안 피드백 속도를 MPS 단위로 변환 후 나눠줌.
      
20210707, 채정훈
////////////////////////////////////////////////////////////////
0)V1.0.0.13 -> V1.0.0.14 MDM99: 20210707
1)64비트 슬레이브 호밍 관련
  -Parameter 5.51 EhterCAT Origin Offset2 HIGH DWORD  추가
             5.52 EhterCAT Origin Offset2 LOW DWORD   추가
             Pr5Max 51->52 로 수정
  -64비트 슬레이브 호밍 관련 코드 추가  gOriginFbkOffset_64 이용 (CSD7 코드 Merge)
  -Paramert List(엑셀 파일) 수정

20210705, 최연범
///////////////////////////////////////////////////////////////
V1.0.0.12 -> V1.0.0.13 MDM99: 20210705
1) SEMES OHT향 Homing Function 추가
   - 0x2C1C, 0x3C1C, 0x4C1C Object 추가
   - OHTHoming Run Function 추가
   - CSP 구동 시  Status Word의 12번 비트를 항상 High로 만드는 부분 수정(표준 확인 필요) 
2) 보간 이상 수정(기어비 적용 시 이상 현상 확인), 안쓰는 변수 삭제
3) Run26_BlackBox_Start 선언 삭제(Run27_BlackBox_Start 추가)
4) XML 수정
5) MCprofile.c의 세미 클론이 연속으로 있는 부분 삭제(;;)
6) Smart Tuning Mode 3 적용 시 DDC Gain의 변경이 반영되지 않는 부분 수정
7) DC Sync와 Inner ISR, Outer ISR 간격 수정(Inner 0us, Outer 50us) -> 추후 테스트 필요
   (기존 Inner +50us, Outer 0us)
8) Smart Tuning Mode 1 멤버 변수 기존값으로 복원
 

20210705, 신동석
///////////////////////////////////////////////////////////////
V1.0.0.11 -> V1.0.0.12 MDM99: 20210702
1) 210701 HDF Update 
2) PER_FW_156 Biss 리니터에서 운전 시 간헐적으로 튀는 증상 발생
 - Biss 통신설정에서 MCLK 20MHz에서 50MHz로 변경으로 기존 주파수에 맞도록 수정
 - Delay 시간 추가
 

20210702, 신동석
///////////////////////////////////////////////////////////////
V1.0.0.10 -> V1.0.0.11 MDM99: 20210702
1) PER_FW_152 낮은 용량 모터 Mismatch 비교에서 Standard 모터는 용량으로 3rd 모터는 CSMA 정격전류기준으로 수정
 - MinimalMotorCurrent[0](Standard W), MinimalMotorCurrent[1](3rd Reated Current Apeak)로 나누어서 사용


20210701, 최연범
///////////////////////////////////////////////////////////////
V1.0.0.9 -> V1.0.0.10 MDM99: 20210701
1) PER_FW 135 : Programmed run or Smart Tuning Autosetup의 Precision 문제 해결을 위한 Hysteresis 추가 
2) PER_FW 128 : Tuningless Mode의 Velocity Override 문제 수정
3) PER_FW 154 : Actual Velocity 이상 현상 수정
4) 리니어 모터, DD 모터의 경우 전류 제어기 B/W를 3kHz 수준으로 변경
5) Ft-1.25를 이용하여 가속도 Threshold 값 변경 적용(Smart Tuning Parameter Estimation)


20210630, 최연범
///////////////////////////////////////////////////////////////
V1.0.0.8 -> V1.0.0.9 MDM99: 20210630
1) DMA_Read(), DMA_Write() 함수 내 DMA의 완료를 확인하는 루틴 추가
   - DMA_Read() 함수 전/후 DummyIdling() 삭제
   - DMA 완료 여부는 Interrupt Active Signal Check
 

20210625, 신동석
///////////////////////////////////////////////////////////////
V1.0.0.7 -> V1.0.0.8 MDM99: 20210628
1) PER_FW_153 리니어 모터 Reset 후에 Ki_vc 10배 되는 문제 수정
 - ApplicationPara(103,ax) 계산에서 리니어 모터일때 0.1에서 0.01로 수정


20210628, 채정훈
////////////////////////////////////////////////////////////////
V1.0.0.6 -> V1.0.0.7 MDM99: 20210628
1)64bit 포지션 변수 관련 OverFlow 문제 수정
  CSD7 최신 F/W V2.13.99.18 에 적용되어있는 코드와 머지 후 수정
  변경 사항: sI64 pos_fdk -> sI32 pos_fdk
         sI64 pos_cmd -> sI32 pos_cmd
         sI64 pos_cmd1 -> sI32 pos_cmd1
         sI64 pos_cmdOffset -> sI32 pos_cmdOffset
         sI64 PcmdI_flt -> sI32 PcmdI_flt
         FP64 fGearBox -> FP32 fGearBox;
         objPositionActualValue 계산방법 변경
         
2)Homing 탈출 시 OperMode가 Jog 에서 Normal로 바뀌지 않는 현상 수정
   Homing 완료 후 다시 Homing 할 수 있도록 수정한 코드 원복 (2021.01.08 dsshin)
   기존 CSD7과 같이 Homing 완료 후 다시 Homing 하려면 모드를 바꾸고 돌아와야 하는 방법사용.
  
20210625, 함년근
///////////////////////////////////////////////////////////////
V1.0.0.5 -> V1.0.0.6 MDM99: 20210625
1) PER_FW_134 Overload 조건일때 Servo OFF에서 줄어드는 부분 실행되도록 수정
 - CheckFdbOverLoad()에서 return 수정


20210625, 신동석
///////////////////////////////////////////////////////////////
V1.0.0.5 -> V1.0.0.6 MDM99: 20210625
1) PER_FW_72 0x60E0, 0x60E1 User PDO 10배로 저장되는 문제 수정
 - nPDORx1700Status[ax] 추가하여 User PDO 설정되면 데이터 업데이트
2) PER_FW_148 Tuning with Programmed Run에서 리니어 속도 맞지 않는 부분
 - PreCalPRF()에서 리니어 모터일때 속도 변환 수정
3) PER_FW_114 AqB Converter Inner Alarm E.039를 3개로 분리 
 - E.039(CONPE) 수정 및 E.120(CONHE), E.121(CONRE) 추가
4) PER_FW_121 DD Motor일때 Offline Auto Tuning 시 Inertia 추정 되도록 수정
 - DD Motor 맞게 Gain 및 속도 줄임
5) 드라이브와 모터 용량 저용량 Mismatch일때 CAP 경고에 추가
 - 200W(50W미만), 400W(100W미만), 800W(200W미만), 1kW(400W미만)
 - 3rd(CSMA 정격전류 0.9배 미만) : 0.9는 마진


20210624, 채정훈
///////////////////////////////////////////////////////////////
V1.0.0.4 -> V1.0.0.5 MDM99: 20210624
1) PER_FW_147 Overflow 관련 디버깅
   V1.0.0.4 에서 수정내용 코드리뷰결과 반영하여 변경.


20210624, 채정훈
////////////////////////////////////////////////////////////////
0) V1.0.0.3 -> V1.0.0.4 MDM99: 20210624
1) PER_FW_147 OverFlow 관련 디버깅
   위치 명령 OverFlow 발생 한 상태에서  Servo Off -> Servo On 할 경우  pos_fdk 튀는 현상 개선 


20210622, 이현규 
////////////////////////////////////////////////////////////////
0) V1.0.0.2 -> V1.0.0.3 MDM99: 20210622
1) Gain Switching 관련 기능 미지원으로 오브젝트 속성 변경 및 주석처리(PER_FW_59)
  - B축/C축 OBJ/Parameter도 아래 A축과 동일하게 변경됨
  - 0x2005:3 / P0-0.5 D2 Gain Change Enable: Reserved로 변경
  - 0x2006:3 / P0-0.6 D2 Mode of Gain Switching: Reserved로 변경
  - 0x210A / P0-1.10 Delay Time of Gain Switching: 오브젝트 주석처리
  - 0x210B / P0-1.11 Level of Gain Switching: 오브젝트 주석처리
  - 0x210C / P0-1.12 Hysteresis of Gain Switching: 오브젝트 주석처리
  - 0x210D / P0-1.13 Position Gain Switching Time: 오브젝트 주석처리
  - 0x210E / P0-1.14 Axis-A 2nd Velocity Regulator P Gain: 오브젝트 주석처리
  - 0x210F / P0-1.15 Axis-A 2nd Velocity Regulator I Gain: 오브젝트 주석처리
  - 0x2110 / P0-1.16 Axis-A 2nd Position Regulator Kp Gain: 오브젝트 주석처리
  - 0x2111 / P0-1.17 Axis-A 3rd Velocity Regulator P Gain: 오브젝트 주석처리
  - 0x2112 / P0-1.18 Axis-A 3rd Velocity Regulator I Gain: 오브젝트 주석처리
  - 0x2113 / P0-1.19 Axis-A 3rd Position Regulator Kp Gain: 오브젝트 주석처리
  - 0x2114 / P0-1.20 Axis-A 4th Velocity Regulator P Gain: 오브젝트 주석처리
  - 0x2115 / P0-1.21 Axis-A 4th Velocity Regulator I Gain: 오브젝트 주석처리
  - 0x2116 / P0-1.22 Axis-A 4th Position Regulator Kp Gain: 오브젝트 주석처리
2) 오브젝트/파라미터 공통/축별 속성 처리 사양 정리에 따른 수정
  - 0x2002:4, 0x2005:4, 0x2006:1 공통 속성에 따른 수정
    -> 0x3002:4, 0x3002:4 미사용으로 Reserved4로 변경
    -> F/W에서도 해당 니블은 Para[0][0][2]만 사용됨
  - 0x2008 Password 축별로 원복 
    -> Password -> Axis-A Password
    -> 0x3008/0x4008 원복
    -> 다음 기능은 축별 Password가 아닌 Para[0][0][8]을 기준으로 처리함을 확인함
      --> 정수 초기화 실행
      --> CTT 테스트 시 예외처리
      --> PDW 명령 실행
  - 0x2424, 0x3424, 0x4424: 공통->축별처리로 변경
  - 0x2425, 0x3425, 0x4425: 공통->축별처리로 변경
  - 0x2504, 0x3504, 0x4504: 축별->공통처리로 변경
  - 0x2505~0x2508, 0x3505~0x3508, 0x4505~0x4508: 축별->공통처리로 변경
  - CheckAcPower() 함수 삭제 처리: Para[0][5][11], Para[0][5][13]을 축별처리하지 않는 함수이지만 사용하지 않는 함수여서 삭제 처리함.
  - 0x2518~0x251A (0x3518~0x351A, 0x4518~0x451A): 축별 -> 공통으로 변경
3) V0.2.3.2에서 수정이 안된 #x2317 #x3317 #x4317 삭제 
4) aName0x16XX~aName0x1BXX 수정
  - aName0x16XX 에서 초과하여 정의된 SubIndex 삭제
    -> 0x1601~0x1603, 0x1611~0x1613, 0x1621~0x1623: Subindex 003 삭제
    -> 0x1604, 0x1614, 0x1624: Subindex 002 삭제
    -> 0x1610, 0x1620, 0x1630: Subindex 008, Subindex 009 삭제
    -> 0x1A02, 0x1A12, 0x1A22: 이름없는 Subindex 005 삭제
  - SubIndex간 구분 기호 오류 수정: \000\ -> \000 (EtherCAT Slave Stack Code Tool에서 생성된 코드로 확인함)
5) 대소문자/띄어쓰기 불일치 수정
  - 0x1000 Device type -> Device Type
  - 0x1001 Error register -> Error Register
  - 0x1008 Device name -> Device Name
  - 0x1009 Hardware version -> Hardware Version
  - 0x100A Software version -> Software Version
  - 0x1010 Store parameters -> Store Parameters
  - 0x1010:1 Store all parameters -> Store All Parameters
  - 0x1010:2 Store communication parameters -> Store Communication Parameters
  - 0x1010:3 Store CiA402 parameters -> Store CiA402 Parameters
  - 0x1011 Restore parameters -> Restore Parameters
  - 0x1011:1 Restore default parameters -> Restore Default Parameters
  - 0x1011:2 Restore communication parameters -> Restore Communication Parameters
  - 0x1011:3 Restore CiA402 parameters -> Restore CiA402 Parameters
  - 0x1018:2 Product code -> Product Code
  - 0x1018:4 Serial number -> Serial Number
  - 0x1C32:4 Synchronization Types supported -> Synchronization Types Supported
  - 0x1C33:4 Synchronization Types supported -> Synchronization Types Supported
  - 0x2002:1 Fault And Disable Braking -> Fault and Disable Braking
  - 0x2315:1 TouchProbe Enable -> Touch Probe Enable
  - 0x2315:2 TouchProbe Mode -> Touch Probe Mode
  - 0x2315:3 TouchProbe Source -> Touch Probe Source
  - 0x2315:4 TouchProbe Edge -> Touch Probe Edge
  - 0x2515:1 Converter Eeprom Write Enable -> Converter EEPROM Write Enable
  - 0x6099:1 Speed For Switch -> Speed for Switch
  - 0x6099:2 Speed For Zero -> Speed for Zero


20210622, 채정훈
//////////////////////////////////////////////////////////////
0) V1.0.0.1 -> V1.0.0.2 MDM99: 20210622
1) 상위 Servo On 시 Halless Linear Commutation Angle Search 안되던 현상 수정
  -> etHoming_StateMachine 함수 호출 시 Homing 수행하지 않고 나올 때 OP_Normal로 지속적으로 변경시켜 Angle Serach 수행 못함
     etHoming_StateMachine 함수 나올 때 OP_Normal로 변경시키는 부분 제거.

20210621, 채정훈
////////////////////////////////////////////////////////////////
 0) V1.0.0.0 -> V1.0.0.1 MDM99: 20210621
 1) DMA delay 를 위한 DummyIdling(92/2)->DummyIdling(46)으로 변경 나누기 연산 제거
 2) Initsys.c 에서 Object 업데이트가 기존방식으로 사용시 분해능이 떨어져 서 매 주기 업데이트 하도록 변경. 
 3) AC Power Fail 3축 동시에 알람 뜨도록 변경
 4) HDF(210621) Update

20210617, 신동석
///////////////////////////////////////////////////////////////////
0) V0.9.0.4 -> V1.0.0.0 MDM99: 20210617
1) DMA_Read() 앞뒤에 DummyIdling(92/2) 추가
2) HDF(210617) Update 
3) CUR_MAX_10 29.7 -> 28 수정
4) CheckAmpAndMotorCapacity()에서 pAx->I_drive_max CSD7과 동일하게 수정


20210615, 이현규
///////////////////////////////////////////////////////////////////
0) V0.9.0.3 -> V0.9.0.4 MDM99: 20210615
1) PDO Mapping 값 이상(offset이 잘못 적용되거나 적용되지 않는 버그) 수정
  - 0x1001 Error Register, 0x0000 Padding 에 Offset이 적용되는 버그
    -> 0x6XXX(>= 0x6000)가 아닌 PDO Mapping 오브젝트들은 Offset을 적용하지 않도록 수정
  - 0x16X5 및 0x1AX5에도 Offset을 적용하도록 수정
  
  
20210614, 신동석
//////////////////////////////////////////////////////////////////
0) V0.9.0.2   -> V0.9.0.3 MDM99: 20210613
1) 0x2C1B,0x3C1B,0x4C1B BlackBox Start 추가 


20210614, 함년근
//////////////////////////////////////////////////////////////////
0) V0.9.0.2   -> V0.9.0.3 MDM99: 20210613
1) Vesion 관련 수정
2) FOE Bootloader Header 관련 수정
3) RSWare로 설정 변경 시 FlashSaveFlagAtPowerOff로 FlashBackup 결정
4) 0x2C1B,0x3C1B,0x4C1B BlackBox Start 추가 


20210611, 최연범
//////////////////////////////////////////////////////////////////
0) V0.9.0.0   -> V0.9.0.2 MDM99: 20210613
1) Read DMA Size 변경 : 41 -> 50 

20210611, 이현규(채정훈)
//////////////////////////////////////////////////////////////////
0) V0.9.0.0   -> V0.9.0.1 MDM99: 20210610
1) V0.2.5.13 -> V0.2.5.14 수정사항을 V0.9.0.0에 머지함, 이현규 
2) PER_FW_137 수정, 채정훈
  - DC Sync를 사용하지 않는 상위제어기에서 위치보간시 첫 번째 축만 동작하는 현상 
  - nUpdateDelayedFlag, nUpdateFlag 다축화해서 처리.


20210610, 신동석
//////////////////////////////////////////////////////////////////
* MMCE install package에 포함된 버전
0) V0.2.5.13   -> V0.9.0.0 MDM99: 20210610
1) ECAT_ReadErrorHistory Error 조건 추가
  - MAX_ERR_GROUP 넘어가는 현상 발생
2) PER_HW_12 Drive Inner Temperature 값이 튀는 증상 수정
  - sI16Temper 16bit 비트 연산에서 이상한 값으로 변경
3) Version 관련사항 수정
4) Black Box Save 위치 수정
5) 리니어 Biss 0x6064(Position Actual Value) 올라오지 않는 문제 수정
  - BiSSOfsRmvdAbsTurnDataInv[ax] 추가
6) PER_HW_12 Drive Inner Temperature 값이 튀는 증상 수정
  - sI16Temper 입력값 비트 변환 수정


20210609, 윤호성
///////////////////////////////////////////////////////////////////
0) V0.2.5.12 -> V0.2.5.13 MDM99: 20210609
1) PER_FW_68 수정
   ->Tamagawa 스탠다드 모터 ABSA Data 처리를 CSD7과 동일하게 CCW -> CW 로 변경
   
20210608, 최연범
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.11 -> V0.2.5.12 MDM99: 20210608
1) DMA Map 수정
2) Servo On/Off Macro 수정(DMA 값 Read)
3) ANF 호출 위치 수정
4) Logic Version Check 추가(추후 변경 시 변경 요망)
5) Overload Fault(I2C) 가중치 누락 부분 수정
6) Inner ISR 주기 변경(50us -> 100us)

20210608, 함년근
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.10 -> V0.2.5.11 MDM99: 20210608
1) BlackBox 기능 수정
2) Build Data 자동 Update
3) BinaryMaker 기능 수정


20210608, 신동석 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.9 -> V0.2.5.10 MDM99: 20210607
1) External Shunt 설정 수정
  - Para 5.25 External Shunt 설정 25%로 수정
  - Para 5.24 External Shunt 저항 최소값 18ohm 수정
  - Para 0.06(D0) Internal(0), External(1) 만 사용하도록 수정
2) 5.22 D1 : CSD7 Serise B와 동일하게 수정
  - 역기전력 방향 : 0(CW), 1(CCW)
3) LED Display 1,2,3 -> A,B,C 수정
  - DispMonitorNumber(), DispParaNumber(), DispRunModeNumber(), DisplayState() 수정
4) (PER_FW_92,93) EtherCAT TxPdo C축 마지막 Object Touch Probe Function 값이 안 올라오는 현상 수정
  - PDO_OutputMapping()에서 16bit 단위로 하면 마지막 안 쓰여짐 32bit 단위로 수정
  - PDO_InputMapping()도 동일하게 수정
5) (PER_FW_95) Encoder EEPROM CRC 틀릴 경우 E.031이 아닌 E.100 발생
  - EEPROM 데이터 처음 확인할때 CRC도 확인하기 때문에 CRC 틀릴 경우 3rd Party로 인식
  - EEPROM 데이터 Standard 모터에서 CRC 확인 하므로 위에는 삭제 
6) (PER_FW_129) 리니어 모터에서 자극 검출 동작 안됨 (Normal Mode)
  - Run26_EncOffsetAngleSearch() 자극검출 Offset 초기값 설정부분 수정
7) (PER_FW_130) 3rd Party Motor에서 Motor Model 설정 안해도 Reset 후에 Ready 상태로 됨
  - 3rd Party 일때 P0-0.01 (0x90000) 설정되는데 Reset 하면서 저장됨
  - P0-0.01 설정 Reset 때 저장 안되도록 수정 (RSWare에서 저장되도록 수정)
8) (PER_FW_133) Encoder Open Err에서 Fault Clear 후에 Encoder 연결하고 Fault Clear 하면 E.100 발생
  - CheckLinearBissEnc() 에서 아닐 때 설정 초기화 부분에서 이번에 추가한 0x200 초기화 추가


20210608, 함년근
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.8 -> V0.2.5.9 MDM99: 20210608
1) F/W Update 파일 CSD7 일때 Update 안 되도록 수정
2) Power Off 시 Flash 저장  되도록 수정
3) Para Range 넘어갈때  Max, Min 값으로  저장하도록 수정


20210608, 이현규
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.7 -> V0.2.5.8 MDM99: 20210608
1) 백업할 파라미터의 개수 오류 수정
  - Pr1Max: 28 -> 29 (오브젝트 기준으로 잘못 수정한 것 파라미터 기준으로 원복함, V0.2.5.2)
2) 잘못 수정/삭제한 0x2C01, 0x3C01, 0x4C01, 0x2C02, 0x3C02, 0x4C02 (원복, V0.2.5.2)
  - 0x2C01, 0x2C02 축이름 추가
    -> 0x2C01 Axis-A Auto Tuning
    -> 0x2C02 Axis-A Smart Tuning
  - 0x3C01, 0x3C02, 0x4C01, 0x4C02 삭제한것 원복
    -> 0x3C01 Axis-B Auto Tuning
    -> 0x3C02 Axis-B Smart Tuning
    -> 0x4C01 Axis-C Auto Tuning
    -> 0x5C02 Axis-C Smart Tuning
  - 관련 XML 수정
3) 삭제에서 누락된 0x4702, 0x4708,0x4709 추가 삭제 (V0.2.5.2 수정시 누락된것)
4) 중복선언 변수 삭제: INT_2ms_Routine() 내부 변수 uI16Temper 삭제


20210608, 윤호성
///////////////////////////////////////////////////////////////////
0) V0.2.5.6 -> V0.2.5.7 MDM99: 20210608
1) Homing Error 코드 관련 에러 없을 시 0x0000으로 변경
2) PER_FW_68 수정
   ->Tamagawa 스탠다드 모터 ABSA Data 처리를 CSD7과 동일하게 CCW -> CW 로 변경
3) PER_FW_102 수정
   - Fault history #8이 삭제되지 않는 현상 해결(FlashBackUp.c) 7->8 변경
   - 1축 fault 발생 시 0,2축 fault history가 이상 값으로 업데이트 되는 현상 해결
   - 부팅후 히스토리가 EtherCAT상에 표시되도록 루틴 추가

20210607, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.5 -> V0.2.5.6 MDM99: 20210607
1) Capacitor, Fan 수명정보, Drive 온도 OBJ 0x2A50~4 추가 
  - 0x2A50 Built-in Fan Remaining Life Ratio, UINT (16), hour,    1s주기 업데이트
  - 0x2A51 Built-in Fan Remaining Life Time,  UDINT(32), 0.01%,   1s주기 업데이트
  - 0x2A52 Capacitor Remaining Life Ratio,    UINT (16), hour,    1s주기 업데이트
  - 0x2A53 Capacitor Remaining Life Time,     UDINT(32), 0.01%,   1s주기 업데이트
  - 0x2A54 Drive Inner Temperature,           INT  (16), degree, 2ms주기 업데이트
2) Capacitor, Fan 최대수명 변경(김재형수석, 함년근수석 합의) 및 define 문으로 변경
  - FAN_LIFE_LIMIT_HOUR, CAP_LIFE_LIMIT_HOUR
  - CAP 수명: 10년 -> 10년 (동일, 87600)     10 * 365 * 24 = 87600 hour (10년) 
  - FAN 수명: 10년 ->  8년 (87600 -> 70080)  8 * 365 * 24 = 70080 hour (8년)
3) 수명이 다했을 때 0으로 유지하도록 수정
  - ElapseLifeTime() 수정


20210604, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.4 -> V0.2.5.5 MDM99: 20210604
1) OS 버전정보 읽기 이상 수정
  - 데이터 타입 이상으로 특정 값 이상 버전은 이상처리되는 문제 수정
  - uI08 -> uI16
  - 관련 Binary maker 수정
  

20210603, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.3 -> V0.2.5.4 MDM99: 20210603
1) (PER_FW_103) Absolute Homing Completed (6041 Status Word의 bit8, 2006:2) 동작 이상 수정 
  - 증상: 2006:2 오브젝트 동작 안함
  - 증상: Absolute / Incremental Encoder를 함께 사용하는 경우 
          서로 간섭이 발생해서 6041 Status Word의 bit8에 잘못된 값이 저장됨.
  - 원인: Absolute Homing Completed 상태값을 처리하는 포인터(ptrPara006)를 Para[ax][0][6]으로 초기화 하는 코드가 누락되어
          모든 축의 데이터를 Address 0에 기록하는 문제가 발생함
  - 수정사항: ptrPara006 초기화 코드 추가 (Main.c)
2) 기타 주석오류 수정 및 보완


20210602, 윤호성
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.2 -> V0.2.5.3 MDM99: 20210602
1) 몇명의 Fault 항목이 Fault History에 저장되지 않는 현상 수정
   - void main()문이 시작되고 발생하는 Fault들이 저장되지 않음
   - 하기 Flag를 while(1) 시작 전, 초기화 하여 저장하지 않는 현상으로 해당 구문 삭제
      pAxis->Flag.AlarmBackUp =0;
      (pAxis+1)->Flag.AlarmBackUp =0;
      (pAxis+2)->Flag.AlarmBackUp =0;


20210531, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.1 -> V0.2.5.2 MDM99: 20210531
1) 2XXX, 3XXX, 4XXX 오브젝트 중 축별 정보가 아닌 드라이브 전체에 대한 오브젝트는 2XXX 만 남기고 3XXX, 4XXX는 삭제, Name에서 'Axis-A' 삭제
  - 0x2008 Axis-A Password -> Password
  - 0x2021 Axis-A Alias ID -> Alias ID
  - 0x2702 Axis-A Product Code -> Product Code                 -> V0.2.5.8에서 0x4702 추가 삭제, 20210608
  - 0x2708 Axis-A Serial No. High Word -> Serial No. High Word -> V0.2.5.8에서 0x4708 추가 삭제, 20210608
  - 0x2709 Axis-A Serial No. Low Word -> Serial No. Low Word   -> V0.2.5.8에서 0x4709 추가 삭제, 20210608
  - 0x2A1F Axis-A FPGA Version -> FPGA Version
  - 0x2A24 Axis-A Power Time Hour -> Power Time Hour
  - 0x2A25 Axis-A Power Time Min Sec -> Power Time Min Sec
  - 0x2A49 Axis-A OS Version -> OS Version
  - 0x2A4A Axis-A Product Revision -> Product Revision
  - 0x2A4B Axis-A Product Type -> Product Type
  - 0x2C01 Axis-A Auto Tuning -> Auto Tuning   -> V0.2.5.8에서 원복함, 20210608
  - 0x2C02 Axis-A Smart Tuning -> Smart Tuning -> V0.2.5.8에서 원복함, 20210608
  - 0x2C10 Axis-A Drive Reboot -> Drive Reboot
  - 0x2F00 Axis-A Reserved #0 -> Reserved #0
  - 0x2F01 Axis-A Reserved #1 -> Reserved #1
  - 0x2F02 Axis-A Reserved #2 -> Reserved #2
  - 0x2F03 Axis-A Reserved #3 -> Reserved #3
  - 0x2F04 Axis-A Reserved #4 -> Reserved #4
  - 0x2F05 Axis-A Reserved #5 -> Reserved #5
  - 0x2F06 Axis-A Reserved #6 -> Reserved #6
  - 0x2F07 Axis-A Reserved #7 -> Reserved #7
  - 0x2F08 Axis-A Reserved #8 -> Reserved #8
  - 0x2F09 Axis-A Reserved #9 -> Reserved #9
  - 위 오브젝트에 대응하는 B/C축 오브젝트는 삭제함.
  - 구현되어있지 않고 XML에도 없는 aName0x2C13 삭제 
  - 관련 XML 수정

20210528, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.5.0 -> V0.2.5.1 MDM99: 20210528
1) (PER_FW_96) 모듈 교체시 오브젝트 0xF030:0x 및 0xF050:0x 값이 바뀌지 않는 문제 수정 
  - Write0xF030() 함수에서 동작에 장애가 되는 조건문들을 제거하고 간략화함 (bCompleteAccess 조건 삭제: 지원하지 않음, Esi mailbox attribute 참고)
  - APPL_StartInputHandler() 함수에서 SubIndex0 값 체크 구문 삭제 (2 또는 3으로 설정됨)
  - 0xF030 Default 값을 0으로 수정함 (0xF030을 설정했는지 확인하기 위함)
  - OBJ_Write()에서 subindex 체크 예외처리 추가 
2) 오브젝트 0xF010, 0xF030, 0xF050 Subindex name 수정 
  - Module Profile 1st Terminal -> Axis-A
  - Module Profile 2nd Terminal -> Axis-B
  - Module Profile 3rd Terminal -> Axis-C
  - 관련 XML 수정 
3) (PER_FW_111) 0x2C10 Drive Reboot 이나 RSWare를 이용한 Reset 동작할 때 자동 재접속이 안되는 문제 수정
  - Reset 전에 PHY 를 먼저 Reset 하도록 추가 
  - 2CXX, 3CXX, 4CXX, 2FXX, 3FXX, 4FXX 속성 수정
    -> MMCE에서 wo인 경우에는 값이 써지지 않아 ACCESS_WRITE -> ACCESS_READWRITE 로 다시 수정함
  - CTT 관련 테스트 완료 
  - 관련 XML 수정 
4) (PER_FW_104) 0x2F09 A축 예약 명령 #9 (Axis-A Reserved #9) 기능 CSD7과 동일하게 구현함 
  - 생산 공정(PTS)에서는 0x7777 설정으로 EEPROM에 데이터(제품코드, 개정번호, 시리얼넘버)를 쓰는 목적으로 사용됨
  - 데이터 타입에 맞게(RunData16->RunData32) 수정함
  - 0x7777이외의 기능은 사용 여부가 명확하지 않아 추가하지 않음
5) 버전정보 관련 수정
  - 0x1018:3 Revision과 0x2A4A Product Revision 이 같도록 수정 
  - 버전 String으로 표기시 'V'추가 (0x1009 Hardware Version, 0x100A Software Version)
6) Revision 판정 알고리즘 개선 (추후 수정해서 사용)
7) CTT TF-2301 AL CoE Object Dictionary 오류 수정 
  - Unexpected name 'Axis-X Home Current Time' expected: 'Axis-X Forced Homing Flag': 0x2533, 0x3533, 0x4533
  - Backup flag 'False' expected: 'True': 0x6068, 0x607D 607F, 0x6081, 0x6083, 0x6084, 0x6085, 0x6086, 0x60A3, 0x60A4, 0x60C5, 0x60C6 
8) 불필요한 파일 삭제를 위한 프로그램 파일 추가: Clean.bat, cmd 바로가기
  - .bak, .old, .pui 삭제하는 프로그램임

20210524, 최연범
///////////////////////////////////////////////////////////////////////////////
0) V0.02.04.04 -> V0.02.05.00
MDM99 : 20210526
1) ENC ID 5를 사용하는 경우 Encoder Reset 이상현상 수정
2) 210524 HDF 적용
3) PER_FW_77 전류 피드백 시점 변경
4) PER_FW_125 Tuninlgess 속도 모드에서 속도 오차 Display 누락 수정
5) Ft-6.04 Range 변경(지난 Merge 시 누락)


20210524, 채정훈
///////////////////////////////////////////////////////////////////////////////
V0.2.4.3 -> V0.2.4.4    MDM99: 20210524
1) PER_FW_127 PP Mode 구동 중 Homing Mode로 변경 시 낮은 속도로 계속 모터가 구동되는 현상 수정
2) PER_FW_15  Homing Mode ErrorCode Update 안되던 현상 수정
3) PER_FW_82  저속(6RPM)에서 PP 구동중 Homing으로 모드 전환시 모드 전환이 되었다고  MCST에 표시하지만 (0x6060, 0x6061이 Homing으로 표시) 실제 설정한 Homing Velocity로 동작 X
              => 이상태에서 다시 PP모드로 모드전환시 모터튐 현상 발생
              Stop Velocity Check를 기존 6RPM 에서 3RPM으로 변경
4) PER_FW_23  CSP -> PP Mode 변경 시 모터 튐 현상 기존 버전에서 수정한것에서 oldtargetpos를 사용하는 다른 방식으로 변경
5) PER_FW_110 MMCe 연결 후 RSWare에서 서보 오프 시 MMCe에 ErrorStop 상태되는 현상 수정 -> OP 일 때 RSWare에서 Servo Off 못하도록 변경


20210524, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.4.2 -> V0.2.4.3 MDM99: 20210524
1) (PER_FW_75) Alias ID 설정 및 표시 방식 변경
  - EEPROM을 통해서 Alias ID 설정 시 7-Segment에 4자리이상 표기가 안되는 것을 표시 방식(nID-NNN)을 변경하여 5자리까지 표시되도록(A.NNNNNN) 수정함.
    -> 1234로 설정하는 경우 C34로 표기되는 문제 해결 
    -> 6자리까지 표시되도록 수정     
    -> ETG.2010에서는 Unsigned16으로 정의되어 있으므로 0~0xFFFF(0~65535)까지 설정 가능해야 함.(MMCE에서는 UI상 0~32767로 제한하고 있는 문제 있음)
    -> 현재의  7-Segment 표시 방식 변경
      -->  기존: nID-NNN
      -->  변경: A.NNNNN (A는 Alias 또는 Address의 의미, CSD7에서는 An을 사용하였음)
  - OBJ 2021으로는 0~255까지 밖에 설정 못하던 것을(EEPROM을 통해 쓰는 범위보다 매우 작음) 0~65535까지 설정 가능하도록 개선
    -> 변경 사유: MMCE의 축 ID 생성이 Alias ID + 0, + 1, + 2로 생성됨에 따라서 Alias ID를 10단위로 설정해야 할 필요성이 있음. 이럴 경우 0~255 범위는 너무 작음
    -> OBJ 2021의 데이터 타입을 USINT(8bit) -> UINT(16bit)로 변경
    -> Para[x][0][33]속성에서 ParaRangeTbl index를 77 -> 54 (0~255 -> 0~65535)으로 변경 
    -> sEntryDesc0x2021: DEFTYPE_UNSIGNED8 -> DEFTYPE_UNSIGNED16, 0x08 -> 0x10
    -> VendorSpecificObjDic[] 에서 0x2021 Attribute 수정
      -> ParaRangeTbl[77].min -> ParaRangeTbl[54].min
      -> ParaRangeTbl[77].max -> ParaRangeTbl[54].max
    -> Para[0][0][33] Attribute 변경
      --> Range: 77 -> 54 (255 -> 65535)
      --> Change: 0 -> 2 (Always -> Power Cycling)
      --> Rep_para: 0 -> 1 (SET/STR 처리시 B/C축 데이터 대신 A축 데이터에 읽기/쓰기)
    -> 관련 변수 데이터 타입 검토 및 변경
      --> Para[0][0][33].val: sI32로 변경이 필요하지 않음 
      --> OBJ 0x10E0:3: USINT -> UINT 로 변경 
      --> sAliasID.u8Value   : UINT8 -> UINT16, u8Value -> u16Value 로 변경
      --> sAliasID.u8Reserved: UINT8 -> UINT16, u8Reserved -> u16Reserved(u16Value 변경에 따라 변경) 로 변경
    -> OBJ 0x3021, 0x4021로는  Alias ID 변경이 안되는 문제 수정
      --> OBJ 0x3021, 0x4021에 접근시 Para[0][0][33]를 사용하도록 수정
  - ESI(XML)에서 관련 오브젝트 수정
2) (PER_FW_14) 0x5030~2 Axis-X Forced Homing Flag 를 축별 오브젝트 영역 0x2533, 0x3533, 0x4533으로 각각 옮김
  - Axis-X Forced Homing Flag 는 연관된 Axis-X Home Current, Axis-X Home Current Time 오브젝트로 다음으로 옮김  
  - ESI(XML)에서 관련 오브젝트 수정
  

20210524, 채정훈
////////////////////////////////////////////////////////////////////////////////
V0.2.4.1 -> V0.2.4.2 MDM99: 20210524
1)PER_FW_6, PER_FW_23 CSP -> PP Mode 변경 시 모터 튀는 현상 디버깅
2)PER_FW_91 수정
3)Auto Setup 관련 테스트 코드 제거


20210521, 채정훈
V0.2.4.0 -> V0.2.4.1 MDM99: 20210521
1)SEMES OHT 전처리기 값 0으로 변경
2)EMULATOR 전처리기 주석처리


20210521, 최연범
V0.2.3.5 -> V0.2.4.0 MDM99: 20210521
1)In Position 조건 변경(상위 명령 확인)
2)PER_FW67, PER_FW70, PER_FW73, PER_FW74,  수정
3)필요없는 Parameter 삭제(Ft-6.12 ~ Ft-6.14)
4)세메스 OHT 용 전처리기 추가
5)DMA Read Time 변경
6)VelocityActualValue 계산 방법 수정


20210520, 윤호성
///////////////////////////////////////////////////////////////////////////////
V0.2.3.4 -> V0.2.3.5 MDM99: 20210520
1) (PER_FW_107) ZVD Filter 2차 Damping Ratio 적용 시, Master Position 대비 Follower Position 못가는 현상 수정

20210520, 채정훈
////////////////////////////////////////////////////////////////////////////////
V0.2.3.3 -> V0.2.3.4 MDM99: 20210520
1)PP Mode Target Reached 관련 PER_FW_71, PER_FW_80, PER_FW_84 수정
2)AC PowerFail 관련 PER_FW_120 수정

20210520, 신동석 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.3.2 -> V0.2.3.3 MDM99: 20210520
1) PcmdBuffer1_Size 1601로 수정 (PER_116)
2) Utilities_Core0.c에서 SAG 다축 수정부분 반영
3) Para 5.39 반영 안 되도록 수정 (PER_115)
4) Motor.c에서 Gain_setup(), CC_I_SCALE() 수정 (PER_85)
5) Backup.h에서 CUR_MAX_02 8.34로 수정 (PER_109)
6) MCascInterface.c에서 pp_task_init() now_controlword, old_controlword 초기값 추가 (PER_64)
7) ETcia402drive.c에서 0x60C2 Subindex 0를 DP_objInterpolationTimePeriodSubindex()로 수정

20210518, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) V0.2.3.1 -> V0.2.3.2 MDM99: 20210518
1) PDO Mapping Name 수정
  - user rxPDO Mapping -> User Receive PDO Mapping
  - user txPDO Mapping -> User Transmit PDO Mapping
  - 3th -> 3rd
2) 0x1C32:11/0x1C32:12/0x1C33:11/0x1C33:12 이름 변경 
  - 0x1C32:11 SM-Event Missed -> SM-Event Missed Counter 이름 변경
  - 0x1C33:11 SM-Event Missed -> SM-Event Missed Counter 이름 변경
  - 0x1C32:12 Cycle Time Too Small -> Cycle Time Too Small Counter 이름 변경
  - 0x1C33:12 Cycle Time Too Small -> Cycle Time Too Small Counter 이름 변경
3) 0xF000/0xF010 이름 변경(단어 첫문자 대문자로)
4) 0xF030:00 PreOP에서 Write 가능하도록 수정
  - ETG 규격에 맞게 수정(예전에 잘못 수정한 것을 원복함)
5) (PER_FW_118) 지원되지 않는 오브젝트 삭제 
  - CSD7 에서는 지원되나 D8에서는 지원되지 않는 OBJ가 남아있음
  - #x2007 #x3007 #x4007 Axis-X Drive Address
  - #x2317 #x3317 #x4317 Axis-X Fully Closed System Selection -> 삭제 안된 것 V1.0.0.3에서 삭제, 20210622, 이현규
  - #x2318 #x3318 #x4318 Axis-X AqB Scale Resolution
  - #x2319 #x3319 #x4319 Axis-X Load Side Encoder Type
  - #x231A #x331A #x431A Axis-X Load Side Encoder Feedback Forward Direction
  - #x231B #x331B #x431B Axis-X Conversion Ratio Denominator
  - #x231C #x331C #x431C Axis-X Conversion Ratio Numerator
  - #x231D #x331D #x431D Axis-X Smart Compensator Frequency
  - #x231E #x331E #x431E Axis-X Position Difference Error Level
  - #x231F #x331F #x431F Axis-X Position Smoothing Filter
  - #x2320 #x3320 #x4320 Axis-X AqB Scale Lines Per Meter
  - #x2321 #x3321 #x4321 Axis-X Motor Side Overspeed Level
  - #x2A2B #x3A2B #x4A2B Axis-X EtherCAT Version  
6) (PER_FW_108) OBJ 0x60B2, 0x68B2, 0x70B2 PdoMapping 속성에  "| OBJACCESS_RXPDOMAPPING" 추가 (XML에 맞춤)
  - TF-2301 > General Offline Dictionary plausibility > Compare the offline dictionary and the online dictionary 에서 에러 발생함
  - XML에서는 "R"로 되어 있으나 Online 에서는 Flag가 없음
  - XML : RSA_D8_Series_V0.2.2.1_20210330.xml
  - EEPROM : D8 Series_V0.0.4_20210317
  

20210317, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) 00.01.00.00 -> 00.01.00.01 MDM99: 20210315
1) ECAT OBJ 0x1008 Device name 값 처리 코드 수정 (ETcoeappl.c, ETecat_def.h)
  - RMD_xxxBN1 -> D8_xxxBxN1
  - 2축/3축 구분코드 추가 'D'/'T'
  - 세부모델명 변경
    -> RMD_220BN1 -> D8_220BDN1
    -> RMD_440BN1 -> D8_440BDN1
    -> RMD_AA0BN1 -> D8_AA0BDN1
    -> RMD_222BN1 -> D8_222BTN1
    -> RMD_444BN1 -> D8_444BTN1
    -> RMD_842BN1 -> D8_842BTN1 
2) 세부모델명 변경에 따라 XML 수정(ESI_Builder.c, RSA_RMD_Series_V0.0.6_20210317.xml)
3) 세부모델명 변경에 따라 EEPROM 수정(D8_xxxBxN1_EEPROM_V0.0.4_20210317.bin)
4) CTT 테스트 오류 수정
  - 0x221F, 0x321F, 0x421F 
    -> Object Info오류 : SubIndex name이 정의되어 있지 않아 추가함(ETcia402drive.c), Inter locking -> Interlocking, Persistent Vibration -> Persistent Oscillation
    -> Read 오류 : OBJCODE_VAR -> OBJCODE_REC
  - 0x2426, 0x3426, 0x4426
    -> Read 오류 : OBJCODE_VAR -> OBJCODE_REC
  - 0x103F ESI와 name이 불일치하여 수정 
  - 0x2006 ESI와 name이 불일치하여 수정
5) CTT warning 수정
  - 0x241C, 0x341C, 0x441C : ESI에 맞게 aName 수정 
  - 0x2426, 0x3426, 0x4426 : ESI에 맞게 aName 수정 
  - 0x2300, 0x3300, 0x4300 : Reserved4 -> Gear Ratio Change
  - 0x2515, 0x3515, 0x4515 : Reserved4 -> BiSS Commutation Angle Search Enable
  - 0x2006, 0x3006, 0x4006 : Reserved1 -> Shunt Resistor Connection, Reserved2 -> Absolute Homing Completed
  - 0x60C2 : 'Interpolation Period ' -> 'Interpolation Period'
  - 0x60F2 : Positioning Option Code -> Position Option Code
  
6) CTT 테스트 시 예외 처리 
  - CTT를 돌리는 경우 기존에는 펌웨어 수정을 통해서 dwDebugLevel에 DBG_ECAT_CTT_PASSING를 OR하여 예외처리되도록 했으나
    별도 펌웨어를 만들어야 하는 번거로움이 있었다.
  - 개선후에는 OBJ 0x2008를 888로 설정만 하면 CTT 시 Auto tuning과 같은 RUN COMMAND가 실행되지 않는다.
  - Ft-0.8 (또는 0x2008)이 888 일 경우 WR_RUNCOMMAND()에서 명령을 처리하지 않고 바로 ABORTIDX_NOERROR를 리턴하도록 수정함.

20210305, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) 00.00.09.07 -> 00.00.09.08 MDM99: 20210305
1) PP 모드에서 역방향 동작 중 HALT 명령 시 (0x605D Halt Option code가 2 일때) 0x6085 로 감속되지 않는 문제 수정 (천병훈 수석 요청사항, MCprofile.c)
  - HALT 동작 : Controlword(0x6040)의 HALT bit(bit 8) ON
  - 0x605D Halt Option code : 1 - 0x6084 값으로 Decel, 2 - 0x6085 값으로 Decel
  - MCproGenStartTvelProfile()에서 가속도 결정 알고리즘을 절대값 기준으로 판정하도록 수정함
    -> 기존 알고리즘 대비 cmdVelocity = 0, actualVelocity < 0 일 때 Accel이 적용되던 것은 Decel이 적용됨
2) OBJ 0x605D 가 TxPDO Mapping 가능하도록 속성 수정(천병훈 수석 요청사항, ETcia402appl.h)
  - XML 수정(RSA_RMD_Series_V0.0.5_20210305.xml)
  
20201210, 이현규 
///////////////////////////////////////////////////////////////////////////////
0) 00.00.09.00 -> 00.00.09.00 MDM99: 20201210
# CTT Test Error 발생 관련 수정
# ESI(XML) Builer Project 추가(ESI_Builder > src > Readme.txt 참조
1) 0x6085, 0x60F2 EntryDesc 수정
  - RXPDO MAPPING 속성 삭제 

2) 0xF000 수정
  - Modulardeviceprofile 수정
    -> Subindex 갯수 오류 수정(3개->2개)
  - ApplicationObjDic[] 수정
    -> Max subindex : MAX_AXES -> 2

3) 0xF010 수정
  - Name 수정
    -> Subindex name 추가
  - ApplicationObjDic[] 수정
    -> Max subindex : MAX_AXES -> 3
    -> DataType / Object Code (Object Type) /  수정 : U32/ARR -> REC/REC
        -----------------------------------------------------------------------------------------------------
        |   Object Code 및 Data Type 의 정의
        |   (1) 참고
        |     - ETG.1000.6 Section 5.6.7.2~3
        |     - ETG.6100.2 Section 3.1
        |   (2) Object Code : 이 파라미터는 일반 타입 정보를 규정한다. (VAR, ARRAY, RECORD)
        |     - VAR : Single Variable
        |     -  ARRAY : Array of variables of same Type
        |     - RECORD : Struct of variables of different Types
        |   (3) Data Type : 데이터의 타입
        -----------------------------------------------------------------------------------------------------
        
4) 0xF030 수정
  - EntryDesc/Name 수정
    -> Subindex name 추가
    -> WRITE(in PREOP) 속성 삭제
  - ApplicationObjDic[] 수정
    -> Max subindex : MAX_AXES -> 3
    -> DataType / Object Code (Object Type) /  수정 : U32/ARR -> REC/REC
 
5) 0xF050 수정  
  - Name 수정
    -> Subindex name 추가
  - ApplicationObjDic[] 수정
    -> Max subindex : MAX_AXES -> 3
    -> DataType / Object Code (Object Type) /  수정 : U32/ARR -> REC/REC

6) 0x607B, 0x607D 수정
  - DefCiA402AxisObjDic 수정
    -> DEFTYPE_INTEGER32 -> DEFTYPE_RECORD

7) 0x241C, 0x341C, 0x441C 수정
  - Name 수정
    -> Reserved3 -> Current Command Feedforward
  - EntryDesc 수정
    -> Sbuindex3 : ACCESS_READ -> ACCESS_READWRITE

8) 0x2516, 0x3516, 0x4516 수정
  - EntryDesc : DEFTYPE_INTEGER16 -> DEFTYPE_UNSIGNED16
  - VendorSpecificObjDic
    -> Max subindex : 4 -> 0
    -> DataType / Object Code (Object Type) /  수정 : DEFTYPE_RECORD/OBJCODE_REC -> DEFTYPE_UNSIGNED16/OBJCODE_VAR
    -> asEntryDesc -> &sEntryDesc

9) 0x2C01, 0x2C02, 0x2C07, 0x2C09, 0x2C0A, 0x2C0C, 0x2C10, 0x2F00~0x2F09 수정
    0x3C01, 0x3C02, 0x3C07, 0x3C09, 0x3C0A, 0x3C0C, 0x3C10, 0x3F00~0x3F09 수정
    0x4C01, 0x4C02, 0x4C07, 0x4C09, 0x4C0A, 0x4C0C, 0x4C10, 0x4F00~0x4F09 수정
  - EntryDesc : ACCESS_READWRITE -> ACCESS_WRITE

10) CTT 동작시 오토튜닝 동작 문제 해결방안 제시
  - PASSWORD(0x2008) 값이 맞아야 처리하는 것으로 설계 변경 제안 
  - ETcia402drive.c 4765 line 주석처리된 내용 참고

11) 0x1010, 0x1011 수정
  - GenObjDic[]
    -> DEFTYPE_UNSIGNED32 -> DEFTYPE_RECORD

12) Diagonostic Service 관련 기능 중지
  - Diag_CreateNewMessage() 에서 예외 처리 조건문 수정
    -> CSD7(V2.11.00.00)과 동일하게 if(1)로 수정함
  - OBJ_GetDesc에서 Subindex 갯수를 6개로 처리해 주는 구문 주석처리
  - XML 에서 아래와 같이 수정하면 DiagHistory가 활성화 됨
    -> <CoE SdoInfo="1" PdoAssign="1" PdoConfig="1" CompleteAccess="0" DiagHistory="1">
  - 아직은 기능이 완전하지 않아 추후 개선 필요

13 ) INT8 타입 Object 쓰기 오류 수정 (OBJ_Write)
  - char 타입의 데이터 비교시 정상동작하지 않음에 따라 음수일 경우 음수변환 코드를 추가하여 처리하도록 함
  - U8 타입 처리에서 분리하였음.

  