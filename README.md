# AR-Navigation-Project

- 경기 인력개발원 유니티 부트캠프에서 약 3주간 진행했던 AR 네비게이션 제작 팀 프로젝트

# 계기?

- '사람과 숲'과 협력하여 유니티를 이용한 인천시 재물포의 주요 장소(이하 POI)들을 소개하고 장소까지 가는 길을 알려주는 네비게이션 프로그램을 제작하였다.
- 총 4명의 팀을 이루어 제작하였으며 나는 길찾기 화면에 진입하면 원하는 지역의 지도와 사용자의 위치, POI의 데이터를 받아온 뒤 지도 화면을 불러오고 화면에 사용자와 POI의 위치에 맞게 마커를 띄워준 다음, POI의 마커를 띄우면 이와 관련된 정보를 보여주는 파트를 담당하였다.

# 필요했던 기능

- 2D 지도를 얻고 화면 상에 출력.
- 현재 위치 정보를 얻기 위한 GPS 정보 획득.
- POI 데이터를 불러와 저장, 활용.

# 구현 과정

- **지도 출력**
 기능은 Naver Cloud Platform을 이용하였다. 사실 게임 개발자를 지망했었고, 또 코딩을 배운지 겨우 반년도 채 되지 않았었기에 네비게이션 앱 프로젝트 자체가 모르는 것 투성이었다. 다행히 네이버에서 제공하는 map API를 이용하면 화면에 출력하는 것은 간단했다. Naver Cloud Platform에 계정을 등록한 다음 API 양식대로 제공받은 ID, PW와 함께 유니티의 WebRequest 클래스를 사용하면 지도 출력 자체는 어렵지 않았다.

Naver API 사용 설명
-- Directions5 API
-- Static map API
-- API Key
해당 프로젝트를 사용하여 개발
ROI 변경 방법
Directions5 Json 구조
Static map API 링크
License
License.md
