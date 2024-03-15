# AR-Navigation-Project(Station J)

경기 인력개발원 유니티 부트캠프에서AR 네비게이션 제작 팀 프로젝트


# 프로젝트 개요

SI기업 '사람과 숲'과 협력하여 유니티를 이용해 인천시 재물포의 주요 장소(이하 POI)들을 소개하고 장소까지 가는 길을 알려주는 네비게이션 앱.

# 인원, 기간 및 담당 파트

총 4명이 함께 팀을 이루어 약 2주간 제작하였으며, 저는 길찾기 화면에 진입하면 원하는 지역의 지도와 사용자의 위치, POI의 데이터를 받아온 뒤 지도 화면을 불러오고 화면에 사용자와 POI의 위치에 맞게 마커를 띄워준 다음, POI의 마커를 띄우면 이와 관련된 정보를 보여주는 파트를 담당하였습니다.

# 사용 기술
- C#
- Unity


# 요구 기능

- 2D 지도를 얻고 화면 상에 출력.
- 현재 위치 정보를 얻기 위한 GPS 정보 획득.
- POI 데이터를 불러와 저장, 활용.
- 마커를 원하는 위치에 출력하고 터치 시 상호작용 기능

# 사전 기획

저는 맡았던 기능을 크게 세 가지로 나누었습니다.

1. POI데이터를 받아와 데이터를 저장한다.
- POI데이터를 받아와 저장하는데 있어 가장 중점적으로 고려했던 것은 1)데이터는 App상의 어디서나 접근할 수 있어야 하며 2)POI DB의 원하는 데이터만 사용할 수 있도록 각 데이터가 분리되어있어야 하고 3) POI가 추가되거나 제거되더라도 수정할 필요가 없도록(또는 최소한의 수정만 필요하도록) 유연성을 가지고 있어야 한다는 점입니다.
   
 2. 지도를 받아와 화면에 띄운다.
- 해당 기능의 구현은 네이버, 카카오 등 지도 서비스를 제공하는 회사의 API를 활용할 계획입니다.
   
 3. POI의 위치에 맞게 지도상의 위치에 마커를 띄운다.
- 에서 사용한 API의 마커 기능을 활용할 계획입니다.

# 코드
<details>
<summary>POI Info</summary>
<br>
   
```
/// <summary>
/// POI 데이터를 저장하는 컨테이너와 POI 데이터를 받아와서 컨테이너에 저장하는 스크립트
/// </summary>
public struct POIData   //저장할 POI의 정보가 담긴 구조체

{
    private int number;
    private string category;
    private string name;
    private double latitude;
    private double longitude;
    private string branch;
    private string address;
    private string description;
    private string eventinformation;


    //생성자로 POI 데이터 담기
    public POIData(int number, string category, string name, string branch, double latitude, double longtitude, string address, string description, string eventinformation)
    {
        this.number = number;
        this.category = category;
        this.name = name;
        this.branch = branch;
        this.description = description;
        this.address = address;
        this.latitude = latitude;
        this.longitude = longtitude;
        this.eventinformation = eventinformation;
    }

    //read only 프로퍼티로 POI 데이터 출력
    public int Number() => number;
    public string Category() => category;
    public string Name() => name;
    public string Branch() => branch;
    public double Latitude() => latitude;
    public double Longitude() => longitude;
    public string Address() => address;
    public string Description() =>description;
    public string Eventinformation() =>eventinformation;
}


public static class POI   // POI데이터는 공유할 것이므로 static, 리스트로 구현
{
    public static List<POIData> datalist = new List<POIData>();
}



public class POI_Info : MonoBehaviour
{
    [Header("POI데이터가 있는 웹 주소")]
    [SerializeField] string POIwebURL;
    [SerializeField] string POIsheetName;
    [SerializeField] string POIrange;

    private void Awake()
    {
        StartCoroutine(requestGetPOI()); // POI 요청 시작
    }
    IEnumerator requestGetPOI()  //인터넷에서 POI 데이터를 받아와 POI.datalist에 저장하는 메서드
    {
        UnityWebRequest WebData = UnityWebRequest.Get($"{POIwebURL}&{POIsheetName}&{POIrange}");

        

        yield return WebData.SendWebRequest();  //POI 정보가 있는 주소로부터 데이터 받아오기 요청

        string json = WebData.downloadHandler.text;  //받아온 데이터를 저장.

        string[] jsonRow = json.Split('\n');    //받아온 데이터를 POI별로 분리.


        string[] splited=new string[jsonRow[0].Length]; //POI의 각각의 정보를 분리할 배열

        for (int i = 0; i < jsonRow.Length; i++)
        {
            jsonRow[i] = jsonRow[i].Replace("\"", string.Empty);    //필요없는 문자 지우기

            if (i % 2 == 0)  //셀 하나에 줄이 두개인 데이터가 있어 두번째 줄을 처리하기 위한 구분. 첫째줄일 경우
            {
                splited = jsonRow[i].Split(',');   // ,를 기준으로 분리

                if (splited.Length > 1 && splited[1] == "") splited[1] = POI.datalist.Last().Category();
                //카테고리의 셀이 병합된 상태라 ""로 나오는 때가 있어 바로 직전의 카테고리 셀로 적용되게끔 처리

            }

            else
            {
                string[] desAndEvent = jsonRow[i].Split(',');   // ,를 기준으로 분리


                if (int.TryParse(splited[0], out int splited_1)
                    &&double.TryParse(splited[5], out double splited_2)
                    && double.TryParse(splited[4], out double splited_3))   //문자열로 받아온 POI의 번호와 위경도를 변환
                {
                    POI.datalist.Add(new POIData(splited_1, splited[1], splited[3], splited[6], splited_2, splited_3, splited[7], desAndEvent[0], desAndEvent[1]));
                    //분리한 POI데이터를 알맞게 분배하여 datalist에 POI데이터 생성

                }
            }
        }


        yield break;

    }

}
```
</details>


 
<details>
<summary>POI Info</summary>
<br>
```
   

/// <summary>
/// 네이버 지도 API를 사용하여 2D static 지도를 받아와 화면에 출력하는 스크립트
/// </summary>

public class MapRequestManager : MonoBehaviour
{
    [Header("네이버 API를 받기 위한 정보")]
    [SerializeField]string mapBaseURL = "https://naveropenapi.apigw.ntruss.com/map-static/v2/raster"; 
    [SerializeField]string clientID = "r2kal6fto4";
    [SerializeField]string clientPW = "hEidFWsoN8dBnqFCreezcSC5HEE1NuYMNTmboVNz";

    [Header("지도 표시할 캔버스 이미지")]
    [SerializeField] RawImage MapImage;

    [Header("지도정보")]
    int width = 360;
    int height = 800;
    double latitude =0f;
    double longitude =0f;
    [SerializeField]int MapSizeLevel=17;


    [Header("GPS를 받기 위한 정보")]
     GPS gps;

    [Header("마커를 생성하는 스크립트")]
    MarkerInstantiate markerInstantiate;

    void Awake()
    {
        gps=GetComponent<GPS>();
        markerInstantiate = GetComponent<MarkerInstantiate>();
        MapImage = MapImage.gameObject.GetComponent<RawImage>();

        gps.Request();  //GPS 클래스의 request 메서드를 호출. 사용자에게서 GPS 권한을 받아오고 수락시 location service 실행

        MapSizeLevel = Mathf.Clamp(MapSizeLevel, 1, 20);    //지도의 확대 레벨을 1~20 사이로 제한

    }


    private void Start()
    {
        StartCoroutine(MapAPIRequest());    //지도를 띄우고 마커생성을 요청하는 코루틴
    }
    IEnumerator MapAPIRequest()     //네이버 지도 API를 받아와 MapImage에 표시하는 메서드
    {
        yield return new WaitUntil(() => POI.datalist.Count > 0);   //POI 데이터를 받아올 때 까지 대기


        if (!gps.GetMyLocation(ref latitude, ref longitude))     //GPS를 받아올 수 있다면 위도, 경도를 현재 위치로 설정
        {
            latitude = 37.466480f;
            longitude = 126.657566f;     //그렇지 않다면 재물포역으로
        }


        POI.datalist.Add(new POIData(0, "-", "MyLocation", "-", latitude, longitude, "-", "-", "-"));   //마커 생성을 위해 현재위치 데이터를 POI에 추가

        for (int i = 0; i < POI.datalist.Count; i++)    //POI datalist 리스트를 불러와 마커 배치
        {
            markerInstantiate.MarkerMake(width, height, MapSizeLevel, latitude, longitude, POI.datalist[i]);
        }



        string APIrequestURL = mapBaseURL + $"?w={width}&h={height}&center={longitude},{latitude}&level={MapSizeLevel}"+
            $"&scale=2&format=png";     //지도 API를 받아오기 위한 요청


        UnityWebRequest req = UnityWebRequestTexture.GetTexture(APIrequestURL);     //요청한 API대로 지도 텍스처를 받아온다.
        req.SetRequestHeader("X-NCP-APIGW-API-KEY-ID", clientID);       //발급받은 ID
        req.SetRequestHeader("X-NCP-APIGW-API-KEY", clientPW);          //발급받은 PW


        yield return req.SendWebRequest();  //API요청


        MapImage.texture = DownloadHandlerTexture.GetContent(req);  //MapImage에 받아온 지도의 텍스처 입히기

        switch (req.result)     //받아오는데 성공시 종료. 실패할 경우 디버그로그(임시)로 실패원인 출력
        {
            case UnityWebRequest.Result.Success: yield break;
            case UnityWebRequest.Result.ConnectionError: Debug.Log("Connection Error"); yield break;
            case UnityWebRequest.Result.ProtocolError: Debug.Log("Protocol Error"); yield break;
            case UnityWebRequest.Result.DataProcessingError: Debug.Log("DataProcessing Error"); yield break;
        }

    }


}

```
</details>

https://velog.io/@productuidev/%EC%A4%80%EB%B9%84%ED%95%98%EA%B8%B0-7-%ED%8F%AC%ED%8A%B8%ED%8F%B4%EB%A6%AC%EC%98%A4-GitHub

이걸 참고로 다시 정리하기

# 구현 과정

**POI 획득과 저장**

POI POI는 구글 스프레드 시트로 관리할 것이고 각 POI의 정보는 오름차순으로 정렬되어 있으며 인덱스를 통해 바로바로 접근할 수 있도록 컨테이너는 리스트를 사용해 구현하였습니다. POI값은 어디서나 불러올 수 있어야 하기 때문에 이 리스트는 static이며, 또한 값을 한번 저장하면 수정되어선 안되기 때문에 각 POI의 필드값은 private 한정자를 사용하되 값을 읽어들이는 것은 가능해야하므로 getter를 제공, 마지막으로 생성자를 만들어 매개변수를 받아와 POI의 필드값을 초기화 할 수 있도록 설계하였습니다.
다음으로 한 작업은 데이터가 저장된 시트를 불러와 적절히 편집한 다음 리스트 저장하는 작업입니다. WebReqeust 클래스를 사용해서 데이터를 받아오고, 이를 POI 구조체의 매개변수에 맞게 분리, 취합하여 적절히 편집하고 구조체를 생성하여 리스트에 추가하는 방법으로 POI 컨테이너를 만들었습니다.


**지도 출력**

우선 지도를 받아올 플랫폼으로 Naver Cloud Platform을 이용하였습니다. Naver Cloud Platform에 계정을 등록한 다음 API 양식대로 제공받은 ID, PW와 함께 유니티의 WebRequest 클래스를 사용하여 지도 출력은 간단하게 해낼 수 있었습니다.
그러나 곧바로 문제가 발생하였는데 바로 유니티에서는 네이버에서 제공하는 Dynamic map을 사용할 수 없었다는 것입니다. 따라서 Dynamic map이 제공하는 다양한 기능들, 특히 마커와 관련된 기능을 사용할 수 없게 되었고, 저는 기존 기획을 수정하여 유니티에서 사용할 수 있는 Static map을 텍스처로 불러와 Raw Image에 입히고 마커는 직접 버튼을 생성하여 구현하기로 하였습니다.


**마커 생성**

"위도, 경도 데이터가 있으니, 그것을 지도의 축척, 해상도 크기와 대조해서 적절하게 유니티의 좌표로 변환하면 되지 않을까? 하는 아이디어로 출발했습니다. 그러나 나이브하게 간단하다고 여겼던 아이디어 단계에서와 달리 위, 경도를 변환하는 과정은 생각보다 번거로운 과정이었습니다.
먼저 네이버 static map에서 사용하는 지도의 축척과 확대 단계에 따른 축척값의 변화가 필요합니다. 네이버에서 제공하는 static map의 축척값은 OpenStreetMap과 같았고, 확대 레벨에 따른 변화는 OpenStreetMap에서 1을 더한 것과 같습니다.

OpenStreetMap에서는 확대 레벨이 1 오를 때마다 지도의 확대 수준이 2배, 달리 말하면 확대 레벨이 오를 수록 (유니티 화면상에서)같은 크기가 차지하는 실제 면적이 절반씩 줄어든다는 뜻이고 따라서 확대 레벨 1일때의 축척을 x라고 할 때 x*(2^-n+1)이라고 계산합니다.
다음은 상기한 축척에서 기준으로 잡았던 지도의 크기(=해상도)를 알아내어 우리가 사용할 지도의 크기에 맞게 수정할 차례입니다. OpenStreetMap 기준 해상도는 256x256이며 네이버 static map도 동일하다는 전제를 가지고 이를 우리가 사용할 360x800크기에 맞게 비율을 구한 다음 이 값을 아까 얻어낸 축척에 곱하여 1픽셀 당 거리값을 구합니다.

1픽셀 당 실제 거리값을 구했으니 이제 마커가 기준점과 어디로, 얼마나 떨어져 있는지 구하면 됩니다. (기준점은 지도 중앙이며 사용자의 현 위치로 가정)
이제 픽셀당 거리 * distance * 1000(distance 공식의 결과값 단위는 km고 픽셀당 거리는 m이기 때문입니다) 여기에 bearing으로 방향을 구한 뒤 곱해주면 POI가 가진 위도, 경도를 유니티 월드 좌표에 맞게 변환, 생성 수 있게 됩니다.
