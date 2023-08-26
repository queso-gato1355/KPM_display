# 국내 기상청 Weather Display 만들기

## 개요

2017년부터 Weather Display와 관련한 다양한 프로젝트를 진행해온 나는 그동안 개발자들이 자주 사용해온 [WeatherUnderground](https://www.wunderground.com/) API와 [Open Weather Map](https://openweathermap.org/guide) API 대신 국내 기상청(Korea Meteorological Administration)에서 제공하는 기본 API 서비스를 사용해보기로 했다. 거기에 WeatherUnderground의 정책 변화로 [더이상 해당 API를 제공하지 않게 되었고,](https://medium.com/@imdipto/best-free-alternatives-to-the-wunderground-weather-api-21acb22450e6) 나는 이를 대체할 방법인 기상청 API에 집중하기로 했다.

## 고민거리

나는 WeatherUnderground와 Open Weather Map 서비스 중 Open Weather Map 서비스를 좋아한다. 첫 번째 이유는 달의 위상도 같이 제공한다는 점이었으며, 파싱이 쉽도록 Json을 구조화했다는 점이었다. 이런 간단한 구조 덕분에 아두이노 커뮤니티에서는 Open Weather Map을 위한 [전용 라이브러리도 개발하기에 이르렀다.](https://github.com/Bodmer/OpenWeather) 그러나 국내 기상청 API의 구조에는 문제점이 많아보인다.

### WeatherUnderground의 API구조

```json
{
    "current_observation": {
        "UV": "1",
        "dewpoint_c": 11,
        "dewpoint_f": 51,
        "dewpoint_string": "51 F (11 C)",
        "display_location": {
            "city": "San Francisco",
            "country": "US",
            "country_iso3166": "US",
            "elevation": "47.00000000",
            "full": "San Francisco, CA",
            "latitude": "37.77500916",
            "longitude": "-122.41825867",
            "magic": "1",
            "state": "CA",
            "state_name": "California",
            "wmo": "99999",
            "zip": "94101"
        },
        "estimated": {},
        "feelslike_c": "13.9",
        "feelslike_f": "57.0",
        "feelslike_string": "57.0 F (13.9 C)",
        "forecast_url": "http://www.wunderground.com/US/CA/San_Francisco.html",
        "heat_index_c": "NA",
        "heat_index_f": "NA",
        "heat_index_string": "NA",
        "history_url": "http://www.wunderground.com/weatherstation/WXDailyHistory.asp?ID=KCASANFR58",
        "icon": "partlycloudy",
        "icon_url": "http://icons.wxug.com/i/c/k/partlycloudy.gif",
        "image": {
            "link": "http://www.wunderground.com",
            "title": "Weather Underground",
            "url": "http://icons.wxug.com/graphics/wu2/logo_130x80.png"
        },
        "local_epoch": "1402931138",
        "local_time_rfc822": "Mon, 16 Jun 2014 08:05:38 -0700",
        "local_tz_long": "America/Los_Angeles",
        "local_tz_offset": "-0700",
        "local_tz_short": "PDT",
        "nowcast": "",
        "ob_url": "http://www.wunderground.com/cgi-bin/findweather/getForecast?query=37.773285,-122.417725",
        "observation_epoch": "1402931132",
        "observation_location": {
            "city": "SOMA - Near Van Ness, San Francisco",
            "country": "US",
            "country_iso3166": "US",
            "elevation": "49 ft",
            "full": "SOMA - Near Van Ness, San Francisco, California",
            "latitude": "37.773285",
            "longitude": "-122.417725",
            "state": "California"
        },
        "observation_time": "Last Updated on June 16, 8:05 AM PDT",
        "observation_time_rfc822": "Mon, 16 Jun 2014 08:05:32 -0700",
        "precip_1hr_in": "0.00",
        "precip_1hr_metric": " 0",
        "precip_1hr_string": "0.00 in ( 0 mm)",
        "precip_today_in": "0.00",
        "precip_today_metric": "0",
        "precip_today_string": "0.00 in (0 mm)",
        "pressure_in": "29.89",
        "pressure_mb": "1012",
        "pressure_trend": "+",
        "relative_humidity": "81%",
        "solarradiation": "--",
        "station_id": "KCASANFR58",
        "temp_c": 13.9,
        "temp_f": 57.0,
        "temperature_string": "57.0 F (13.9 C)",
        "visibility_km": "16.1",
        "visibility_mi": "10.0",
        "weather": "Scattered Clouds",
        "wind_degrees": 238,
        "wind_dir": "WSW",
        "wind_gust_kph": 0,
        "wind_gust_mph": 0,
        "wind_kph": 4.8,
        "wind_mph": 3.0,
        "wind_string": "From the WSW at 3.0 MPH",
        "windchill_c": "NA",
        "windchill_f": "NA",
        "windchill_string": "NA"
    }
}
```
<br>
딱 봐도 파이썬으로 활용할 때 딕셔너리 형으로의 전환이 매우 쉽게끔 각각의 값이 일대일 대응이 된다. 필요한 정보값은 모두 텍스트로 명문되어 초보 개발자로 하여금 이용하기 매우 쉬웠다. 지원되지 않는 게 아쉬울 따름이다.

### Open Weather Map API 구조

```json
{
   "lat":33.44,
   "lon":-94.04,
   "timezone":"America/Chicago",
   "timezone_offset":-18000,
   "current":{
      "dt":1684929490,
      "sunrise":1684926645,
      "sunset":1684977332,
      "temp":292.55,
      "feels_like":292.87,
      "pressure":1014,
      "humidity":89,
      "dew_point":290.69,
      "uvi":0.16,
      "clouds":53,
      "visibility":10000,
      "wind_speed":3.13,
      "wind_deg":93,
      "wind_gust":6.71,
      "weather":[
         {
            "id":803,
            "main":"Clouds",
            "description":"broken clouds",
            "icon":"04d"
         }
      ]
   },
   "minutely":[
      {
         "dt":1684929540,
         "precipitation":0
      },
      ...
   ],
   "hourly":[
      {
         "dt":1684926000,
         "temp":292.01,
         "feels_like":292.33,
         "pressure":1014,
         "humidity":91,
         "dew_point":290.51,
         "uvi":0,
         "clouds":54,
         "visibility":10000,
         "wind_speed":2.58,
         "wind_deg":86,
         "wind_gust":5.88,
         "weather":[
            {
               "id":803,
               "main":"Clouds",
               "description":"broken clouds",
               "icon":"04n"
            }
         ],
         "pop":0.15
      },
      ...
   ],
   "daily":[
      {
         "dt":1684951200,
         "sunrise":1684926645,
         "sunset":1684977332,
         "moonrise":1684941060,
         "moonset":1684905480,
         "moon_phase":0.16,
         "summary":"Expect a day of partly cloudy with rain",
         "temp":{
            "day":299.03,
            "min":290.69,
            "max":300.35,
            "night":291.45,
            "eve":297.51,
            "morn":292.55
         },
         "feels_like":{
            "day":299.21,
            "night":291.37,
            "eve":297.86,
            "morn":292.87
         },
         "pressure":1016,
         "humidity":59,
         "dew_point":290.48,
         "wind_speed":3.98,
         "wind_deg":76,
         "wind_gust":8.92,
         "weather":[
            {
               "id":500,
               "main":"Rain",
               "description":"light rain",
               "icon":"10d"
            }
         ],
         "clouds":92,
         "pop":0.47,
         "rain":0.15,
         "uvi":9.23
      },
      ...
   ],
    "alerts": [
    {
      "sender_name": "NWS Philadelphia - Mount Holly (New Jersey, Delaware, Southeastern Pennsylvania)",
      "event": "Small Craft Advisory",
      "start": 1684952747,
      "end": 1684988747,
      "description": "...SMALL CRAFT ADVISORY REMAINS IN EFFECT FROM 5 PM THIS\nAFTERNOON TO 3 AM EST FRIDAY...\n* WHAT...North winds 15 to 20 kt with gusts up to 25 kt and seas\n3 to 5 ft expected.\n* WHERE...Coastal waters from Little Egg Inlet to Great Egg\nInlet NJ out 20 nm, Coastal waters from Great Egg Inlet to\nCape May NJ out 20 nm and Coastal waters from Manasquan Inlet\nto Little Egg Inlet NJ out 20 nm.\n* WHEN...From 5 PM this afternoon to 3 AM EST Friday.\n* IMPACTS...Conditions will be hazardous to small craft.",
      "tags": [

      ]
    },
    ...
```
<br>
이 제공처도 원래는 이 서비스를 무료로 제공했지만, 현재는 1000번 호출은 무료이고 그 이후로는 100회당 요금을 부과하는 방식으로 바뀌었다. 하루 1000번 이상 호출하려면 1분에 1번씩 24시간 내내 호출하면 되는데, 솔직히 누가 그렇게 자주 호출하겠는가.

### 기상청의 API 구조

```
# YYMMDDHHMI STN  WD   WS GST  GST  GST     PA     PS PT    PR    TA    TD    HM    PV     RN     RN     RN     RN     SD     SD     SD WC WP WW                      CA  CA   CH CT        CT  CT  CT    VS   SS    SI ST    TS    TE    TE    TE    TE  ST   WH BF IR IX
#        KST  ID  16  m/s  WD   WS   TM    hPa    hPa  -   hPa     C     C     %   hPa     mm    DAY    JUN    INT    HR3    DAY    TOT -- -- ---------------------- TOT MID  MIN -------- TOP MID LOW                  GD     C     5    10    20    30 SEA    m --  -  -
201512110100 108  29  1.7  -9 -9.0   -9 1002.1 1012.7 -9  -9.0   5.2   3.0  86.0   7.6   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 19                      -9  -9   -9 -         -9  -9  -9   800 -9.0 -9.00 -9   4.0   4.6   5.7   6.5   6.5  -9 -9.0 -9  3 -9
201512110200 108  29  2.2  -9 -9.0   -9 1001.9 1012.5 -9  -9.0   4.6   2.4  86.0   7.3   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 19                      -9  -9   -9 -         -9  -9  -9   900 -9.0 -9.00 -9   3.7   4.2   5.4   6.5   6.5  -9 -9.0 -9  3 -9
201512110300 108  29  2.4  -9 -9.0   -9 1002.0 1012.6  5  -0.6   4.1   1.6  84.0   6.9   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 10 11 19                       3   2   30 AcCi       1   3   0   900 -9.0 -9.00  1   3.5   4.0   5.2   6.4   6.5  -9 -9.0 -9  3  1
201512110400 108  29  1.4  -9 -9.0   -9 1002.1 1012.7 -9  -9.0   3.6   1.4  86.0   6.8   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 10 21 19                       4   2   10 ScAc       0   3   5   900 -9.0 -9.00 -9   3.3   3.6   5.0   6.3   6.4  -9 -9.0 -9  3  1
201512110500 108  29  2.2  -9 -9.0   -9 1002.6 1013.2 -9  -9.0   3.4   1.1  85.0   6.6   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 10 10 19                       3   1   10 ScAc       0   3   5   900 -9.0 -9.00 -9   3.3   3.5   4.8   6.2   6.4  -9 -9.0 -9  3  1
201512110600 108  27  1.8  -9 -9.0   -9 1003.0 1013.6  2   1.0   3.4   0.9  84.0   6.5   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 19                       6   3   10 ScAc       0   3   5  1200 -9.0 -9.00 -9   3.0   3.3   4.7   6.1   6.4  -9 -9.0 -9  3  2
201512110700 108  27  1.8  -9 -9.0   -9 1003.6 1014.2 -9  -9.0   3.4   0.4  81.0   6.3   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 -                        6   6   10 Sc         0   0   5  1300 -9.0 -9.00 -9   2.8   3.1   4.5   6.0   6.3  -9 -9.0 -9  3  2
201512110800 108  29  1.5  -9 -9.0   -9 1004.7 1015.3 -9  -9.0   3.3   0.3  81.0   6.2   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 -                        6   6   10 Sc         0   0   5  1400  0.0  0.01 -9   2.4   2.9   4.4   5.9   6.3  -9 -9.0 -9  3  2
201512110900 108   0  0.3  -9 -9.0   -9 1005.8 1016.4  2   2.8   3.7   0.3  79.0   6.2   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 -                        6   6   10 Sc         0   0   5  1300  0.1  0.13  1   3.2   3.0   4.2   5.8   6.2  -9 -9.0 -9  3  2
201512111000 108   9  2.6  -9 -9.0   -9 1006.5 1017.0 -9  -9.0   6.4  -0.8  60.0   5.8   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 -                        1   1   10 Sc         0   0   5  1700  0.7  0.54 -9   5.4   4.2   4.3   5.7   6.2  -9 -9.0 -9  3  2
201512111100 108   7  3.9  -9 -9.0   -9 1007.1 1017.5 -9  -9.0   9.4  -0.7  49.0   5.8   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0   -9.0 -9 -9 -                        0   0   -9 -         -9  -9  -9  1900  1.0  1.20 -9   8.8   6.8   4.9   5.7   6.1  -9 -9.0 -9  3  2
```

원래 예전에는 XML이나 JSON과 유사한 구조였는데 갑자기 표 형태의 텍스트로 바뀌었다. 아마 더 활용하기 까다로워지지 않았나 하는 생각이 든다.
