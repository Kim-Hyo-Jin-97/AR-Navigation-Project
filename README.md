# AR-Navigation-Project(Station J)

경기 인력개발원 유니티 부트캠프에서AR 네비게이션 제작 팀 프로젝트 [Statin J](https://github.com/Unity-Team-Unreal/AR-nav.git) <br>
개발 기간 2023.02.13 ~ 2023.02.29

---

# 목차

- [1. 프로젝트 개요](#1-프로젝트-개요)
- [2. 인원, 기간 및 담당 파트](#2-인원,-기간-및-담당-파트)
- [3. 사용 기술](#3-사용-기술)
- [4. 요구 기능](#4-요구-기능)
- [5. 사전 기획](#5-사전-기획)
- [6. 핵심 코드](#6-핵심-코드)
- [7. 핵심 구현 과정](#7-핵심-구현-과정)
- [8. 트러블 슈팅](#8-트러블-슈팅)
- [9. 좋았던 점](#9-좋았던-점)
- [10. 아쉬운 점 또는 개선점](#10-아쉬운-점-또는-개선점)
- [11. 마무리](#11-마무리)

# 1. 프로젝트 개요

SI기업 '사람과 숲'과 협력하여 유니티를 이용해 인천시 재물포의 주요 장소(이하 POI)들을 소개하고 장소까지 가는 길을 알려주는 네비게이션 앱.

# 2. 인원, 기간 및 담당 파트

총 4명이 함께 팀을 이루어 약 2주간 제작하였으며, 저는 길찾기 화면에 진입하면 원하는 지역의 지도와 사용자의 위치, POI의 데이터를 받아온 뒤 지도 화면을 불러오고 화면에 사용자와 POI의 위치에 맞게 마커를 띄워준 다음, POI의 마커를 띄우면 이와 관련된 정보를 보여주는 파트를 담당하였습니다.

# 3. 사용 기술

- C#
- Unity


# 4. 요구 기능

- 2D 지도를 얻고 화면 상에 출력.
- 현재 위치 정보를 얻기 위한 GPS 정보 획득.
- POI 데이터를 불러와 저장, 활용.
- 마커를 원하는 위치에 출력하고 터치 시 상호작용 기능

# 5. 사전 기획

저는 맡았던 기능을 크게 세 가지로 나누었습니다.

**1. POI데이터를 받아와 데이터를 저장한다.**
- POI데이터를 받아와 저장하는데 있어 가장 중점적으로 고려했던 것은<br>1) 데이터는 App상의 어디서나 접근할 수 있어야 하며<br>2) POI DB의 원하는 데이터만 사용할 수 있도록 각 데이터가 분리되어있어야 하고<br>3) POI가 추가되거나 제거되더라도 수정할 필요가 없도록(또는 최소한의 수정만 필요하도록) 유연성을 가지고 있어야 한다는 점입니다.
   
**2. 지도를 받아와 화면에 띄운다.**
- 해당 기능의 구현은 네이버, 카카오 등 지도 서비스를 제공하는 회사의 API를 활용할 계획입니다.
   
**3. POI의 위치에 맞게 지도상의 위치에 마커를 띄운다.**
- 에서 사용한 API의 마커 기능을 활용할 계획입니다.

# 6. 핵심 코드


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
<summary>MapRequest</summary>
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


<details>
<summary>MarkerInstantiate</summary>
<br>

```
    /// <summary>
    /// 마커가 배치될 위치를 정하고 마커를 생성하는 스크립트
    /// </summary>

    Vector2 point = new();  //마커가 생성될 위치

    [SerializeField] GameObject Marker;     // 마커 프리팹

    BubbleState MarkersbubbleState;     //마커 프리팹의 스크립트 컴포넌트

    public void MarkerMake(int width, int height, float Level, double latitude, double longitude, POIData poidata)    //    마커를 생성하는 메서드.
    {
            float Lv1size = 156_543;    //줌 레벨 1의 픽셀당 미터

            float perPixel = Lv1size / Mathf.Pow(2, Level + 1);     //1단계마다 절반씩 줄어들며, openstreetmap 기준이므로 네이버맵에 맞게 1단계 더 올린다.

            float defaulte = 256f;      //픽셀당 미터 값의 기준 해상도는 256x256.

            float widthPer = defaulte / width;
            float heightPer = defaulte / height;        //기준해상도 대비 지도의 해상도 비율


            float inUnityPerPixel = widthPer * heightPer * perPixel;    //지도 해상도의 픽셀당 미터값


            float distance = (float)distanceInKilometerByHaversine(latitude, longitude, poidata.Latitude(), poidata.Longitude());     //기준점과 마커와의 실제 거리 재기

            float bearing = bearingP1toP2(latitude, longitude, poidata.Latitude(), poidata.Longitude());    //기준점과 마커와의 방위각 재기


            distance = 1000 * distance / inUnityPerPixel;  // 실제 거리를 유니티 지도상에서 몇 픽셀만큼 그려야 하는지 계산


            Vector2 direction = new Vector2(Mathf.Sin(bearing * Mathf.Deg2Rad), Mathf.Cos(bearing * Mathf.Deg2Rad));    //방위각을 이용해 마커가 놓일 방향을 정하기


            point = new Vector2(0, 0) + direction.normalized *  distance;     //노멀라이즈로 방향만 정한 뒤, distance만큼 떨어진 거리에 마커를 띄운다.


            if(GameObject.Find("Marker_" + poidata.Number()) == null)       //이 POI 마커가 아직 없다면
            {
                Marker = Instantiate(Marker, point, quaternion.identity);   //point 위치에 마커를 생성한다.
                MarkersbubbleState = Marker.GetComponent<BubbleState>();
                MarkersbubbleState.thisData = poidata;                      //POI 데이터를 마커에 넣는다.
                Marker.name = "Marker_" + poidata.Number();                 //마커의 이름을 설정한다.
                Marker.transform.SetParent(GameObject.Find("StaticMapImage").transform, false);     //지도의 위치 변화에 따라갈 수 있도록 자식으로 넣는다.
                MarkersbubbleState.MarkerStart();   //마커의 설정 시작
            }
            else GameObject.Find("Marker_" + poidata.Number()).transform.localPosition = point;     //이미 마커가 있다면 위치만 바꾼다.
    }
```
</details>


# 7. 핵심 구현 과정

## POI 획득과 저장

POI POI는 구글 스프레드 시트로 관리할 것이고 각 POI의 정보는 오름차순으로 정렬되어 있으며 인덱스를 통해 바로바로 접근할 수 있도록 컨테이너는 리스트를 사용해 구현하였습니다.<br>

POI값은 어디서나 불러올 수 있어야 하기 때문에 이 리스트는 static이며, 또한 값을 한번 저장하면 수정되어선 안되기 때문에 각 POI의 필드값은 private 한정자를 사용하되<br>
값을 읽어들이는 것은 가능해야하므로 getter를 제공, 마지막으로 생성자를 만들어 매개변수를 받아와 POI의 필드값을 초기화 할 수 있도록 설계하였습니다.<br>

WebReqeust 클래스를 사용해서 데이터를 받아오고, 이를 POI 구조체의 매개변수에 맞게 분리, 취합하여 적절히 편집하고 구조체를 생성하여 리스트에 추가하는 방법으로 POI 컨테이너를 구현하였습니다.

## 지도 출력

우선 지도를 받아올 플랫폼으로 Naver Cloud Platform을 이용하였습니다. Naver Cloud Platform에 계정을 등록한 다음 API 양식대로 제공받은 ID, PW와 함께 유니티의 WebRequest 클래스를 사용하여 지도 출력은 간단하게 해낼 수 있었습니다.<br>
그러나 곧바로 문제가 발생하였는데 바로 유니티에서는 네이버에서 제공하는 Dynamic map을 사용할 수 없었다는 것입니다.<br>
따라서 Dynamic map이 제공하는 다양한 기능들, 특히 마커와 관련된 기능을 사용할 수 없게 되었고, 저는 기존 기획을 수정하여 유니티에서 사용할 수 있는 Static map을 텍스처로 불러와 Raw Image에 입히고 마커는 직접 버튼을 생성하여 구현하기로 하였습니다.

## 마커 생성

"위도, 경도 데이터가 있으니, 그것을 지도의 축척, 해상도 크기와 대조해서 적절하게 유니티의 좌표로 변환하면 되지 않을까? 하는 아이디어로 출발했습니다.<br>

네이버 static map에서 사용하는 지도의 축척과 확대 단계에 따른 축척값의 변화를 찾아보았고, 확대 레벨 1일때의 축척을 x라고 할 때 x*(2^-n+1)이라고 계산할 수 있었습니다.<br>
다음은 상기한 축척에서의 기준 해상도를 알아내어 우리가 사용할 지도의 크기에 맞게 수정할 비율을 구한 다음, 이 값을 아까 얻어낸 축척에 곱하여 1픽셀 당 거리값을 구합니다.<br>

1픽셀 당 실제 거리값을 구했으니 이제 마커가 기준점과 어디로, 얼마나 떨어져 있는지 구할 수 있었습니다.(기준점은 지도 중앙이며 사용자의 현 위치로 가정)<br>
이제 픽셀당 거리 * distance * 1000(distance 공식의 결과값 단위는 km고 픽셀당 거리는 m이기 때문입니다) 여기에 bearing으로 방향을 구한 뒤 곱해주어 POI가 가진 위도, 경도를 유니티 월드 좌표에 맞게 변환, 생성 할 수 있었습니다.


# 8. 트러블 슈팅

## 8.1 지도의 확대와 움직임

- 제작하고자 했던 앱은 네비게이션이니만큼 당연히 지도를 받아오고 나서 자유롭게 화면을 드래그하거나 확대/축소할 수 있어야 한다고 생각했습니다.
- 그러나 유니티에서 Dynamic Map을 사용할 수 없었기에 저는 간단하게 사용자의 입력값이 주어지면 그만큼 지도를 입힌 RawImage의 Transform을 변경하는 방식으로 구현했습니다.

<details>
<summary>최초 코드</summary>
<br>
   
```
/// <summary>
/// 지도의 확대, 축소, 움직임을 담당하는 스크립트
/// </summary>
public class MapTransformManager : MonoBehaviour
{
    [Header("지도의 최소, 최대 확대 비율")]
    [SerializeField] float minSize;
    [SerializeField] float maxSize;
    [Header("줌인,아웃, 스크롤링 속도")]
    [SerializeField] float ZoomSpeed;
    [SerializeField] float MoveSpeed;

    [Header("지도")]
    RawImage MapImage;



    [Header("터치 계산용 벡터")]
    Vector2 nowPos, prePos;
    Vector3 movePos;


    void Awake()
    {
        MapImage = GetComponent<RawImage>();
    }


    void Update()
    {
        TouchZoom();
        TouchMove();
    }

    void TouchZoom()    //확대, 축소를 담당하는 메서드
    {
        if (Input.touchCount == 2) //손가락 2개가 눌렸을 때
        {
            Touch touchZero = Input.GetTouch(0); //첫번째 손가락 터치를 저장
            Touch touchOne = Input.GetTouch(1); //두번째 손가락 터치를 저장

            //터치에 대한 이전 위치값을 각각 저장
            //처음 터치한 위치(touchZero.position)에서 이전 프레임에서의 터치 위치와 이번 프레임에서 터치 위치의 차이를 뺌
            Vector2 touchZeroPrevPos = touchZero.position - touchZero.deltaPosition;
            Vector2 touchOnePrevPos = touchOne.position - touchOne.deltaPosition;

            // 각 프레임에서 터치 사이의 벡터 거리 구함
            float prevTouchDeltaMag = (touchZeroPrevPos - touchOnePrevPos).magnitude;
            float touchDeltaMag = (touchZero.position - touchOne.position).magnitude;

            // 거리 차이 구함(거리가 이전보다 크면(마이너스가 나오면)손가락을 벌린 상태_줌인 상태)
            float deltaMagnitudeDiff = prevTouchDeltaMag - touchDeltaMag;

            Vector3 mapScale = MapImage.transform.localScale;   //지도의 현재 스케일을 저장

            mapScale.x += -deltaMagnitudeDiff * ZoomSpeed * Time.deltaTime;
            mapScale.y += -deltaMagnitudeDiff * ZoomSpeed * Time.deltaTime;
            //지도의 스케일을 얼마나 바꿀 것인지 계산

            float MapScaleX = Mathf.Clamp(mapScale.x, minSize, maxSize);
            float MapScaleY = Mathf.Clamp(mapScale.x, minSize, maxSize);
            //지도의 확대 수준을 제한

            MapImage.transform.localScale = new Vector2(MapScaleX, MapScaleY);
            //지도 크기 변경 적용
        }
    }

    void TouchMove()    //지도의 움직임을 담당하는 메서드
    {
        if (Input.touchCount == 1)  //손가락 하나만 눌렀을 때
        {
            Touch touch = Input.GetTouch(0);    //터치 저장

            if (touch.phase == TouchPhase.Began)    //터치를 막 시작했을 떄
            {
                prePos = touch.position - touch.deltaPosition;  //터치의 이전 위치값 저장
            }
            else if (touch.phase == TouchPhase.Moved)   //터치가 움직일 때
            {
                nowPos = touch.position - touch.deltaPosition;  //터치의 현재 위치값을 저장
                movePos = (Vector3)(prePos - nowPos) * Time.deltaTime * MoveSpeed * MapImage.transform.localScale.x;
                //얼마나 움직였는지를 이전위치-현재위치로 계산, 이후 움직임 속도와 지도의 스케일만큼 보정

                MapImage.transform.Translate(movePos);  //지도를 움직인다.

                prePos = touch.position - touch.deltaPosition;  //터치의 이전 위치값 저장
            }


        }

    }
}
```
</details>


- 그러나 이 코드대로 동작시켜보았을 때 지도가 화면 밖으로 빠져나오거나, 지도를 드래그 한 상태에서 scale을 축소시 그만큼 Canvas의 빈 영역이 생기는 문제가 발생하였습니다.
- 그래서 저는 이 문제를 해결하기 위해 RawImage의 앵커를 Canvas의 각 꼭짓점으로 설정하고, Screen.width(height) *  rawImage의 scale.x(y)의 값을 받아 Mathf.clamp함수를 사용하여 rawImage의 anchoredPosition을 이 범위 내로 한정시키는 것으로 해결할 수 있었습니다.


<details>
<summary>개선 코드</summary>
<br>
   
```
/// <summary>
/// 지도의 확대, 축소, 움직임을 담당하는 스크립트
/// </summary>
public class MapTransformManager : MonoBehaviour
{
    [Header("지도의 최소, 최대 확대 비율")]
    [SerializeField] float minSize;
    [SerializeField] float maxSize;
    [Header("줌인,아웃, 스크롤링 속도")]
    [SerializeField] float ZoomSpeed;
    [SerializeField] float MoveSpeed;

    [Header("지도")]
    RawImage MapImage;



    [Header("터치 계산용 벡터")]
    Vector2 nowPos, prePos;
    Vector3 movePos;


    void Awake()
    {
        MapImage = GetComponent<RawImage>();
    }


    void Update()
    {
        TouchZoom();
        TouchMove();
    }

    void TouchZoom()    //확대, 축소를 담당하는 메서드
    {
        if (Input.touchCount == 2) //손가락 2개가 눌렸을 때
        {
            Touch touchZero = Input.GetTouch(0); //첫번째 손가락 터치를 저장
            Touch touchOne = Input.GetTouch(1); //두번째 손가락 터치를 저장

            //터치에 대한 이전 위치값을 각각 저장
            //처음 터치한 위치(touchZero.position)에서 이전 프레임에서의 터치 위치와 이번 프레임에서 터치 위치의 차이를 뺌
            Vector2 touchZeroPrevPos = touchZero.position - touchZero.deltaPosition;
            Vector2 touchOnePrevPos = touchOne.position - touchOne.deltaPosition;

            // 각 프레임에서 터치 사이의 벡터 거리 구함
            float prevTouchDeltaMag = (touchZeroPrevPos - touchOnePrevPos).magnitude;
            float touchDeltaMag = (touchZero.position - touchOne.position).magnitude;

            // 거리 차이 구함(거리가 이전보다 크면(마이너스가 나오면)손가락을 벌린 상태_줌인 상태)
            float deltaMagnitudeDiff = prevTouchDeltaMag - touchDeltaMag;

            Vector3 mapScale = MapImage.transform.localScale;   //지도의 현재 스케일을 저장
            
            mapScale.x += -deltaMagnitudeDiff * ZoomSpeed * Time.deltaTime;
            mapScale.y += -deltaMagnitudeDiff * ZoomSpeed * Time.deltaTime;
            //지도의 스케일을 얼마나 바꿀 것인지 계산

            float MapScaleX = Mathf.Clamp(mapScale.x, minSize, maxSize);    
            float MapScaleY = Mathf.Clamp(mapScale.x, minSize, maxSize); 
            //지도의 확대 수준을 제한

            MapImage.transform.localScale = new Vector2(MapScaleX, MapScaleY);
            //지도 크기 변경 적용

            DontOutMAP();   //지도 크기가 바뀌면서 캔버스를 벗어나지 않도록 조절
        }
    }

    void TouchMove()    //지도의 움직임을 담당하는 메서드
    {
        if (Input.touchCount == 1)  //손가락 하나만 눌렀을 때
        {
            Touch touch = Input.GetTouch(0);    //터치 저장

            if (touch.phase == TouchPhase.Began)    //터치를 막 시작했을 떄
            {
                prePos = touch.position - touch.deltaPosition;  //터치의 이전 위치값 저장
            }
            else if (touch.phase == TouchPhase.Moved)   //터치가 움직일 때
            {
                nowPos = touch.position - touch.deltaPosition;  //터치의 현재 위치값을 저장
                movePos = (Vector3)(prePos - nowPos) * Time.deltaTime * MoveSpeed * MapImage.transform.localScale.x;
                //얼마나 움직였는지를 이전위치-현재위치로 계산, 이후 움직임 속도와 지도의 스케일만큼 보정
                MapImage.transform.Translate(movePos);  //지도를 움직인다.

                prePos = touch.position - touch.deltaPosition;  //터치의 이전 위치값 저장
            }
            DontOutMAP();   //지도 크기가 바뀌면서 캔버스를 벗어나지 않도록 조절
        }


    }

    void DontOutMAP()   //지도가 캔버스 밖을 벗어나지 않도록 제한하는 메서드
    {
        float x = (Screen.width / 2) * (transform.localScale.x - 1);
        float y = (Screen.height / 2) * (transform.localScale.y - 1);
        // 화면의 가로, 세로값의 절반에 지도의 스케일 값-1만큼 곱
        // 지도가 담긴 캔버스의 앵커가 각 꼭지점에 있고, 지도의 피벗은 정중앙이다.

        float MapPositionX = Mathf.Clamp(MapImage.rectTransform.anchoredPosition.x, -x, x);    //지도의 이동 제한

        float MapPositionY = Mathf.Clamp(MapImage.rectTransform.anchoredPosition.y, -y, y);    //지도의 이동 제한

        MapImage.rectTransform.anchoredPosition = new Vector2(MapPositionX, MapPositionY);  //지도의 앵커드 포지션을 설정

    }
}
```
</details>


# 9. 좋았던 점

## 9.1 팀 프로젝트 경험

여태껏 개인적으로 공부하거나 간단한 프로젝트를 시도해본 적은 있었으나, 팀을 이뤄 프로젝트를 진행해본 경험은 많은 도움이 되었습니다.<br>

필요하면 코드를 즉석에서 수정할 수 있던 개인일 때와 다르게 팀 프로젝트에서 사전에 코드의 유연성, 결합도에 대해 고민하거나 함수, 변수명 또는 주석을 직관적으로 표기하는 등 스크립트를 짤 때 고려했던 요소들은 저에게 개선해야 할 점이 무엇인지, 더 배워야 할 것이 무엇인지 깨닫게 해주는 지양분이 되어주었습니다.<br>

## 9.2 일정 관리 경험

자유롭게 남는 시간을 활용해 작업했던 지금까지와 달리 정해진 마감 기한과 작업 시간을 관리하며 진행했던 이번 프로젝트에서 시간 관리의 중요성을 깨달을 수 있었습니다<br>

특히 처음 계획했던 것과는 다르게 작업 과정에서 예상치 못한 이슈나 잘못된 로직으로 인한 리팩터링, 개발 도중 기획 변경 등 개발과정 상의 오버헤드가 빈번히 발생한다는 것을 이해하고 이를 고려하여 작업 계획을 짜야 함을 배웠습니다.<br>


# 10. 아쉬운 점 또는 개선점

## 10.1 디자인 패턴 사용의 부족

디자인 패턴의 활용이 미숙했습니다. 하나의 예시로, 네비게이션 앱 UI를 디자인하면서 마커 범례를 구분하는 방법으로 enum과 기준점이 될 static enum 객체를 만들고, 각 마커가 기준점 enum객체와 자신의 enum값을 비교하는 함수를 update에 넣는 방법을 사용하였습니다.<br>

그러나 이 방식으로는 마커의 갯수만큼 Update 단계에서 호출되는 횟수도 늘어나는 문제를 가지고 있습니다.<br>

다시 비슷한 작업을 진행할 시에는 Observer 디자인 패턴을 사용하여 Update에 호출되는 횟수를 줄이고 범례를 변경할 때에만 비교하는 함수가 호출될 수 있게 디자인할 것입니다.<br>


## 10.2 협업간 분담 소통 미숙

팀 프로젝트는 처음 진행해보았기 때문에 개인 프로젝트와 달리 각 팀원들이 유기적으로 소통하고, 업무 분담을 확실히 해야한다는 점이 부족하다고 생각했습니다.<br>

예를 들어 개발 초기, POI데이터를 구현하는 과정에서 저와 별개로 다른 팀원 두명도 같은 작업을 하고 있었고 이를 뒤늦게 깨달아 생산성을 저하시켰던 것이 못내 아쉬운 점으로 남아있습니다.<br>


# 11. 마무리

프로그래밍을 배우기 시작한지 약 반년도 채 되지 않아 제대로 프로젝트를 진행해본 적이 없었습니다. 이번 프로젝트는 저에게 팀원과 협력하며 진행해본 첫 경험이었고 그렇기 때문에 많은 실수와 실패, 그리고 그로부터 많은 교훈을 얻을 수 있었습니다.<br>

특히 이론으로 짠 스크립트과 실제 구현과정은 하늘과 땅만큼 다르다는 것을 직접 체감했고, 여러 우여곡절 속에서 직접 작동하는 코드를 스스로 완성하고 조합해보는 경험을 통해<br> 프로그래머란 무엇인가를 만드는 것 뿐만이 아니라, 오류를 찾아내고 고치는 것 역시 중요하다는 것을 배울 수 있었습니다.<br>

앞으로 여러 프로젝트를 거치며 더 많은 오류와 실패를 겪게 될지도 모릅니다. 하지만 이번 프로젝트에서 얻은 경험과 자신감이 있다면 어떤 난관이라도 헤쳐나갈 수 있다고 자신합니다.
