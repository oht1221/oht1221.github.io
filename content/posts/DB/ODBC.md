---
title: "ODBC(Open Database Connectivity)란?"

date: "2020-09-14"

tags:

- ODBC
- 오디비씨
- Microsoft


---

 이 글은 Microsoft 사에서 제공하는 ODBC 관련 공식 문서([https://docs.microsoft.com/en-us/sql/odbc/reference/what-is-odbc?view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/odbc/reference/what-is-odbc?view=sql-server-ver15))를 읽고 번역한 글이며, 영어 실력이 미천하여 오역이 다수 포함될 수 있습니다. 문제가 될 시 삭제하겠습니다.



## ODBC란?

 컴퓨팅 세상에는 ODBC에 대한 많은 오해가 있다. End user 에게는 윈도우즈 제어판의 아이콘일 뿐이고, 어플리케이션 개발자에게는 데이터 접근 routines을 포함한 라이브러리이며, 그 외 많은 사람들에게는 수 많은 DB 접근 문제들에 대한 해답이다.

 그 중에서 ODBC를 가장 잘 설명해주는 말은 'DB API에 대한 스펙'이라고 할 수 있다. 이 API는 어떤 DBMS나 운영체제에도 독립적이다( 이 메뉴얼에선 C를 사용하지만, ODBC API는 프로그래밍 언어에 독립적이다). ODBC API는 Open Group, ISO/IEC의 CLI 스펙에 기반하고 있다. ODBC 3.x는 이 두 스펙을 완벽하게 구현하고 있고 - 이전의 ODBC 버전은 이 스펙들의 이전 버전에 기반하고 있었지만 완벽하게 그 스펙들을 구현하지는 않았다. - 추가적으로 스크롤 가능한 커서처럼 화면 기반의 DB 어플리케이션 개발자들이 자주 쓰는 기능들을 포함하고 있다.

 ODBC API 함수(functions)들은 각 DBMS 드라이버 개발자들에 의해 구현된다. 그러면 에플리케이션들이 DBMS 독립적인 방법으로 데이터에 접근하기 위해 이 드라이버 내의 함수(functions)들을 호출한다. 드라이버 매니저는 에플리케이션과 드라이버 사이의 커뮤니케이션을 관리한다.

마이크로소프트에서 윈도우 95 이상을 사용하는 컴퓨터에 드라이버 매니저를 제공하고 몇 가지의 ODBC 드라이버 를 만들었으며, 마이크로소프트 응용 프로그램에서 ODBC 함수들을 호출하기는 하지만, 사실 ODBC 드라이버는 누구나 작성할 수 있다. 실제로 오늘날 대부분의 ODBC 응용 프로그램과 드라이버는 마이크로소프트에서 만든 것이 아니다. 게다가, ODBC 드라이버와 응용프로그램은 매킨토시에도 있고 UNIX 기반 플랫폼에도 있다.

 응용프로그램 및 드라이버 개발자들을 위해서, 마이크로소프트는 윈도우즈 95 이상을 사용하는 컴퓨터에 대해서 ODBC SDK를 제공하는데 여기에는 드라이버 매니저, 설치 DLL, 테스트 툴, 예제 프로그램들이 포함돼있다. 마이크로소프트는 Visigenic Software와 협업하여 이 SDK들을 매킨토시와 다른 UNIX  기반 플랫폼에 포팅하였다.

 ODBC는 데이터베이스의 가용성을 개방하기(expose)위해 만든 것이지, 강화(supplement)하기 위한 것이 아니다. 그렇기 때문에 애플리케이션 개발자는 ODBC를 사용하면 마치 간단한 데이터베이스가 모든 기능을 갖춘 관계형 데이터베이스로 뚝딱 변신할 거라고 기대하면 안된다. 또한 드라이버 개발자들이 기반이 되는 데이터베이스에 있지도 않은 기능까지 구현하기를 기대해서도 안된다. 한 가지 예외가 있다면 파일 데이터(xBase 파일에 있는 데이터 같은)에 직접 접근하는 드라이버를 개발하는 개발자들은 최소한 SQL기본 기능들을 구현하는 것은 필요하다. 또 다른 예외는, 이전에는 Microsoft Data Access Components(MDAC) SDK에 포함됐던 Windows SDK의 ODBC 컴포넌트는 일정 수준의 기능들을 구현한 드라이버를 위한 커서 라이브러리를 제공한 것인데 라이브러리는 스크롤할 수 있는 커서를 나타내기 위한(simulate) 것이다.

 ODBC를 사용하는 애플리케이션은 어떤 데이터베이스간에도 기능할 수 있도록 할 수 있어야 한다. 예를 들어, ODBC는 heterogeneous join engine 이나 distributed transaction processor는 아니지만 DBMS 독립적이기 때문에 그러한 cross-database tools들을 구현하기 위해 사용될 수도 있다.

 

