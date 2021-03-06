﻿========================================================================
    콘솔 응용 프로그램 : BGIocpServer 프로젝트 개요
========================================================================


/////////////////////////////////////////////////////////////////////////////
특징:
 1. 장점
  1) WorkerThread에 if문 1개도 없음 (CPU가 분기처리 되지 않도록)
  2) 타이머 스레드에서 busywating하지 않도록 설계 (window event활용)
  3) IOCP에서 send할 경우, 버퍼를 항상 만들어 줘야 하는데, 멀티스레드에서 잘 동작하도록 버퍼 재활용
  4) WorkerThread 종료시에, 다른 작업이 실행중에 종료되지 않도록 설계
  5) 타이머 종료시에, 기존에 등록된 타이머작업 안전하게 해제후 종료하도록 설계
  
  
  6) c++11 최대한 활용
   std::thread, 람다를 활용하여 스레드 생성함수 spawn 구현
   typedef 대신 using사용할수 있는부분 사용
   일관된 초기화 방법 사용 {}, nullptr으로 사용
   std::thread 강제 종료 기능 고민하다가, native_handle과 SuspendThread 사용 
   
  7) 재사용 가능하도록 객체 정의
  8) 자세한 주석
  
/////////////////////////////////////////////////////////////////////////////




/////////////////////////////////////////////////////////////////////////////
버전 설명:
[1.00 버전]
 -BGProto프로젝트에서 만든 공통적으로 쓰이는 객체 적용
  BGSingleton,  BGConfigManager,  BGLogManager

[1.01 버전]
 -멀티스레드에서 사용 가능한 기본적인 객체 구현
  BGIOObject, BGIOCompletionHandler, BGIOThread, BGIOAcceptThread, BGIOServer

[1.02 버전]
 -타이머 객체 만들어, BGIOObject를 상속하면 기능 사용 가능하도록 구현
 특징:
  1. 하나의 스레드가 담당해서 동작
  2. 불필요한 while문을 돌지 않는다.
  (BG Proto 프로젝트에서는 항상 while을 돌면서 타이머 동작이 있는지 여부를 검사했다.)

 수정되어야 할 사항:
  1. 하나의 스레드에서 실제 객체의 동작까지 돌아간다.
  -> BG Proto 프로젝트에서는 타이머 스레드와 워커 스레드가 연동되어, 실제 객체의 동작은 워커 스레드에서 담당하도록 처리하였다.
  
 [1.03 버전]
 -멀티스레드에서 사용 가능한 각종 IOLock, IOBuffer 구현
  BGIOBuffer 객체
   특징 : 한번 할당 받은 버퍼는 지우지 않고 재활용 한다.
   -> TCP 전송을 하기 위한 버퍼로 사용된다.

  BGIOLock 객체
  1) BGLockInfo 객체
   스레드 로컬 변수로 선언되며, 스레드마다 객체가 생성된다.
   스레드 소멸시에 객체도 소멸된다.
   스레드마다 pending 된 락의 갯수, lock 된 시각 정보를 가지고 있다.

  2) BGILock 객체
   Lock객체 중 가장 상위 객체
   static 객체로 LockInfo를 map으로 관리하고,
   해당 map을 동기화 시키기 위한 BGRWLock을 가지고 있다.
   
  [1.04 버전]
  -BGIOSocket 구현
   특징 : write시에 항상 바로 보내지 않고, 보내는 중이라면 이후에 패킷을 모아서 한번에 전송하도록 구현
   
  
  [1.05 버전]
  -BGWorkerThread 구현
  
  
  [1.06 버전]
  -스레드를 종료할 수 있는 BGIOTerminate를 구현
  -config 사용법을 개선하기 위해 BGMainConfig 생성
  -BGIOServer를 사용하는데 필수적인 IOCP와 WorkerThread를 관리해주는 BGLayer 객체를 구현
  
  [1.07 버전]
  -이제까지 구현한 것들을 테스트 하기 위해 환경을 구축하고, TestSample 객체들을 생성한다.
  테스트 환경
	0) 필요한 설정값 관리 (포트, 스레드 갯수, 허용 유저 수)
	1) Layer를 통해 IOCPHandler, WorkerThread 생성
	2) BGTestServer를 통해 서버 생성 후 , Accept스레드 가동
	3) accept처리 후 TestSocket 생성
  
  [1.08 버전]
  -StressTest Project 생성
  +서버 부하테스트를 위한 프로젝트
  +완성되기 전에는 간단한 테스트 용으로 사용
  
  -서버와 클라이언트가 공유할 패킷 관련 프로토콜을 정의한 헤더 생성
  +각 구조체를 packeting
  +패킷 갯수와 크기가 바뀔것을 고려하여 설계 (현재는 unsigned char를 사용하여 패킷 종류 최대 256가지)
 
  [1.09]
  -앞으로 실제 사용할 BGTestSocket을 구현에 집중
  +패킷 조립, 처리 집중 구현
  +타이머스레드와 워커스레드 연동 작업 중심
  
  [2.00]
  -본격적인 서버 구조, 로직 개발 시작
  
  [2.01]
  -GameObject 관리 구조 만들기
  
  [2.02]
  -Socket 관리 구조 개선
  +기존에는 std::map 자료구조로 관리하여, 가장 많이 사용되는 find를 할때에도 lock이 필요하였다.
  +메모리는 조금 손해보더라도, 배열로 미리 객체를 만들어 두고, 인덱스로 lock을 필요없이 바로 사용할 수 있도록 개선
  +추가적으로 삽입, 삭제도 일어나지 않기 때문에다. (그대신 종료할때 초기화 해주는 기능이 필요하다.)

 
  [2.0x]
  -GameWorld 관리 구조 만들기
  
/////////////////////////////////////////////////////////////////////////////



/////////////////////////////////////////////////////////////////////////////
고려했던 사항들&이슈들&고민사항 정리:

(1.06버전부터 기록 시작)
- 워커스레드 안전하게 종료하는 방법, std::thread 강제 종료하는 법
- 기본적인 BGServer가 동작하는데 필수적인 객체들 관리해야 할 것 같음 (BGLayer 생성)
- 변수명, 주석 등등 통일성
- Config.ini에 정의된 값들 BGConfig만들어, BGConfigManager 로드 이후에 매번 읽지 않고, 한번 저장후 해당값만 사용하도록 개선


* WSAOVERLAP 구조체를 확장하여 쓸지에 대한 이슈
 - 추가로 IO OPERATION 값을 멤버로 가지고 있어서,
 - IOCP에 신호가 왔을 때, 해당 값으로 send, recv인지 추가로 timer 작업인지 구분하려고 시도
 장점음 타이머 스레드의 작업이 워커 스레드로 연동됨
 하지만 현재 wrtite시에 버퍼를 항상 만들지 않고, 풀에서 재활용 하는 방식으로 구현되어 있고,
 매번 write하지 않고, 네이글알고리즘 기능이 코드단에서 구현이 되어 있음

/////////////////////////////////////////////////////////////////////////////


/////////////////////////////////////////////////////////////////////////////
개선이 필요한 부분:
 1) 타이머 스레드 개선사항
 Timer에서 동작이 Timer Thread에서만 처리된다.
 해당 Thread에 부하가 걸릴수 있어, Timer 에서 처리되는 작업을
 IOCP와 연결된 WorkerThread에 넘겨 스레드 사용이 균등하도록 작업
 -> Timer Thread는 타이머 작업을 확인하여 WorkerThread에 넘기는 일만 하도록 변경한다.

 2) 각종 lock 객체 구현
 사실 검증이 안되어서, std::14에 나온 lock을 사용하는게 좋을 것도 같다.
 
/////////////////////////////////////////////////////////////////////////////



