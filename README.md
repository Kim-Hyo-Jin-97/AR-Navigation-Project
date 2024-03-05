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

이하는 지도 출력에 사용했던 코드 전문이다.

<details>
<summary>코드</summary>
 
> public class MapRequestManager : MonoBehaviour
>{
>    [Header("네이버 API를 받기 위한 정보")]
>    [SerializeField]string mapBaseURL = "https://naveropenapi.apigw.ntruss.com/map-static/v2/raster"; 
>    [SerializeField]string clientID = "";
>    [SerializeField]string clientPW = "";
>
>    [Header("지도 표시할 캔버스 이미지")]
>    [SerializeField] RawImage MapImage;
>
>    [Header("지도정보")]
>    int width = 360;
>    int height = 800;
>    double latitude =0f;
>    double longitude =0f;
>    [SerializeField]int MapSizeLevel=17;
>
>
>    [Header("GPS를 받기 위한 정보")]
>     GPS gps;
>
>    [Header("마커를 생성하는 스크립트")]
>    MarkerInstantiate markerInstantiate;
>
>    void Awake()
>    {
>        gps=GetComponent<GPS>();
>        markerInstantiate = GetComponent<MarkerInstantiate>();
>        MapImage = MapImage.gameObject.GetComponent<RawImage>();
>
>        gps.Request();  //GPS 클래스의 request 메서드를 호출. 사용자에게서 GPS 권한을 받아오고 수락시 location service 실행
>
>        MapSizeLevel = Mathf.Clamp(MapSizeLevel, 1, 20);    //지도의 확대 레벨을 1~20 사이로 제한
>
>    }
>
>    private void Start()
>    {
>        StartCoroutine(MapAPIRequest());    //지도를 띄우고 마커생성을 요청하는 코루틴
>    }
>    IEnumerator MapAPIRequest()     //네이버 지도 API를 받아와 MapImage에 표시하는 메서드
>    {
>        yield return new WaitUntil(() => POI.datalist.Count > 0);   //POI 데이터를 받아올 때 까지 대기
>
>        if (!gps.GetMyLocation(ref latitude, ref longitude))     //GPS를 받아올 수 있다면 위도, 경도를 현재 위치로 설정
>        {
>            latitude = 37.466480f;
>            longitude = 126.657566f;     //그렇지 않다면 재물포역으로
>        }
>
>        POI.datalist.Add(new POIData(0, "-", "MyLocation", "-", latitude, longitude, "-", "-", "-"));   //마커 생성을 위해 현>재위치 데이터를 POI에 추가
>
>        for (int i = 0; i < POI.datalist.Count; i++)    //POI datalist 리스트를 불러와 마커 배치
>        {
>            markerInstantiate.MarkerMake(width, height, MapSizeLevel, latitude, longitude, POI.datalist[i]);
>        }
>
>        string APIrequestURL = mapBaseURL + $"?w={width}&h={height}&center={longitude},{latitude}&level={MapSizeLevel}"+
>            $"&scale=2&format=png";     //지도 API를 받아오기 위한 요청
>
>        UnityWebRequest req = UnityWebRequestTexture.GetTexture(APIrequestURL);     //요청한 API대로 지도 텍스처를 받아온다.
>        req.SetRequestHeader("X-NCP-APIGW-API-KEY-ID", clientID);       //발급받은 ID
>        req.SetRequestHeader("X-NCP-APIGW-API-KEY", clientPW);          //발급받은 PW
>
>        yield return req.SendWebRequest();  //API요청
>
>        MapImage.texture = DownloadHandlerTexture.GetContent(req);  //MapImage에 받아온 지도의 텍스처 입히기
>
>        switch (req.result)     //받아오는데 성공시 종료. 실패할 경우 디버그로그(임시)로 실패원인 출력
>        {
>            case UnityWebRequest.Result.Success: yield break;
>            case UnityWebRequest.Result.ConnectionError: Debug.Log("Connection Error"); yield break;
>            case UnityWebRequest.Result.ProtocolError: Debug.Log("Protocol Error"); yield break;
>            case UnityWebRequest.Result.DataProcessingError: Debug.Log("DataProcessing Error"); yield break;
>        }
>    }
>
>}
</details>
 
ARCore Extension Package 사용법
Google Cloud Platform API 사용법
Geospatial API 사용방법
-- 사용 설정 및 API Key 사용 설명
Geospatial Creator API 사용방법
-- Cesium Pacakge 설치
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
