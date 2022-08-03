# python-flask-request-sample
python flask request sample

```
from flask import Flask, jsonify, request, make_response
from flask_restx import Api, Resource
from urllib import parse
import requests
import xml.etree.ElementTree as ET
from bs4 import BeautifulSoup
import json
import xmltodict

app = Flask(__name__)
api = Api(app)

@api.route('/', methods=['POST'])
class Request(Resource):
    def post(self):
        userRequest = parse.quote(request.get_json().get('query'))

        pnu = RequestKakaoRegionCodeAPI(userRequest)
        region = RequestLandCharacterAPI(pnu)
        RequestLandUseAPI(pnu)
        return region

def RequestKakaoRegionCodeAPI(userRequest):
    apiKey = parse.quote('key')
    uri = 'https://dapi.kakao.com/v2/local/search/address.json?query=' + userRequest
    headers = {'Authorization': 'KakaoAK ' + apiKey}
    res = requests.get(uri, headers=headers)
    res.raise_for_status() # 200 OK 코드가 아닌 경우 에러 발동

    json_data = res.json()

    b_code = json_data['documents'][0]['address']['b_code']
    main_address_temp = json_data['documents'][0]['address']['main_address_no']
    sub_address_tmp = json_data['documents'][0]['address']['sub_address_no']
    
    main_address = main_address_temp.rjust(4, '0') 
    sub_address = sub_address_tmp.rjust(4, '0')
    pnu = b_code + main_address + sub_address
    return pnu

def RequestLandCharacterAPI(pnu):
    apiKey = parse.quote('key')
    encodePnuCode = parse.quote(pnu)
    uri =  'http://openapi.nsdi.go.kr/nsdi/LandCharacteristicsService/wfs/getLandCharacteristicsWFS?authkey=' + apiKey + '&pnu=' + encodePnuCode
    res = requests.get(uri)
    
    dictionary = xmltodict.parse(res.text)
    json_object = json.dumps(dictionary) 
    
    return json_object



if __name__ == "__main__":
    app.run(debug=True, host='IP', port=3000)        

```
