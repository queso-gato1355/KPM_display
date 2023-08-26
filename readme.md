# 국내 기상청 Weather Display 만들기

## 개요

2017년부터 Weather Display와 관련한 다양한 프로젝트를 진행해온 나는 그동안 개발자들이 자주 사용해온 [WeatherUnderground](https://www.wunderground.com/) API대신 국내 기상청(Korea Meteorological Administration)에서 제공하는 기본 API 서비스를 사용해보기로 했다. 거기에 WeatherUnderground의 정책 변화로 [더이상 해당 API를 제공하지 않게 되었고,](https://medium.com/@imdipto/best-free-alternatives-to-the-wunderground-weather-api-21acb22450e6) 나는 이를 대체할 방법인 기상청 API에 집중하기로 했다.

## 고민거리

[WeatherUnderground](https://www.wunderground.com/)의 서비스가 마음에 든 가장 첫 번째 이유는 달의 위상도 같이 제공한다는 점이었으며, 파싱이 쉽도록 Json을 구조화했다는 점이었다. 이런 간단한 구조 덕분에 아두이노 커뮤니티에서는 WeatherUnderground를 위한 [전용 라이브러리도 개발하기에 이르렀다.](https://github.com/storca/ArdUnground) 그러나 국내 기상청 API의 구조에는 문제점이 많아보인다.

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
