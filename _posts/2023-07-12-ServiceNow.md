---
title: ServiceNow
tags: study
---

# ServiceNow

## Basic Authentication

1. 登录官网

https://developer.servicenow.com/

2. 查看我们的instance信息

![image-20230709152933434](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843021.png)

3. 使用postman发送get request

![image-20230709153357696](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843899.png)

4. 通过linux命令行方法调用

![image-20230709153611530](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843620.png)

```shell
curl --location --request GET 'https://dev146324.service-now.com/api/now/table/incident' \
--header 'Authorization: Basic YWRtaW46NnBnWkhzXl5LTTln' \
--header 'Cookie: BIGipServerpool_dev146324=747525898.33598.0000; JSESSIONID=F3755DF0D9FA347A4E4DCF700C4F6924; glide_session_store=102CEE2397332110CB93FCDFE153AF9A; glide_user_activity=U0N2M18xOkFQTnB5dGhRcmtFZ3EvcTgrNmRRM1Q0YUVWbGFVUHV6VDhiUFJ6N2k3RUk9OlljbUNGZTFTM2lycVpjbVNTS1Fzb01yOE5DdGdaWG1pR00yYWFiWVE4MVU9; glide_user_route=glide.9ff2beb519ba4d5940da5f08b92e6d6f' \
-o output.txt

```

![image-20230709154159151](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843484.png)

## OAuth 2.0

1. 登录我们的instance

* 例如我的：https://dev146324.service-now.com/

2. 找到Application Registry

![image-20230709154633361](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843892.png)

3. 点击右上角的new

4. 选择   **Create an OAuth API endpoint for external clients**

![image-20230709154905271](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100845313.png)

5. 输入一个名字就可以提交**Client Secret**会自动生成

* 例如：我的

![image-20230709155109324](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843517.png)

6. 用postman发送post request

* https://dev146324.service-now.com/oauth_token.do
* Body 如下填写

![image-20230709155634770](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843065.png)

7. 复制刚刚获取的**access_token**，并且发送一个get request进行测试

```shell
Z1sIdGk9iozDtUdrYO7RV94vHq4TS8PfyzzQLgMX8amIUkRP9nMXFAy-QavS4Gz6gT7fjNNju6hR-q-5EcP7jQ
```

![image-20230709160042449](https://wuyiqi.obs.cn-north-1.myhuaweicloud.com/tuchuang/202307100843456.png)



## 利用Mockaroo生成并导入数据

> 待补充

* https://mockaroo.com

1. 首先在ServiceNow上面创建一个table（studio）
2. 然后再Mockaroo船舰一个table让他们的字段对应上

3. 生成api

4. 返回instance 搜索并且点击**Connection & Credential Aliases**

5. 返回instance 搜索并且点击**Data Sources**

6. Transform Map

7. Scheduled Imports



## rest api explorer

> 待补充

### get

一、直接使用serviceNow table api

```shell
 https://dev146324.service-now.com/api/now/table/x_1069741_wuyq54_0_vehicle?sysparm_query=make%3DAudi&sysparm_limit=10
```

部分参数解释

sysparm_query：用于指定在查询数据时应用的过滤条件。

make后边的%3是“=”的意思

sysparm_limit=10：查询限制限制10条

二、在Studio创建api

1. (get 获取单个汽车)

```shell
/*
API:            Vehicles
Name:           Get Vehicle
HTTP Method:    GET
Relative path:  /vehicle/{vin}
Description:    Returns data for selected vehicle.
*/

(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Declare empty array for answer
    var answer = [];

    // Get the VIN from the path parameter. Generate 404 if not provided.
    var vin = request.pathParams.vin;
    if (!vin) {
        response.setError(new sn_ws_err.NotFoundError('VIN not provided.'));
        response.setStatus(404);
        return;
    }

    // Query the Vehicle table for this VIN.
    var vehicleTable = 'x_1069741_wuyq54_0_vehicle';
    var vehicleGr = new GlideRecord(vehicleTable);
    vehicleGr.addQuery('vin', vin);
    vehicleGr.query();

    // Generate a 404 if no vehicle was found.
    if (!vehicleGr.hasNext()) {
        response.setError(new sn_ws_err.NotFoundError('No vehicle found.'));
        response.setStatus(404);
        return;
    }

    // For the vehicle found, generate a JSON object and save the vehicle data in it.
    while (vehicleGr.next()) {

        var vehicleObj = {
            "make": vehicleGr.getValue('make'),
            "model": vehicleGr.getValue('model'),
            "year": vehicleGr.getValue('year'),
            // Or pass year as integer
            // "year": parseInt(vehicleGr.getValue('year'), 10),
            "city": vehicleGr.getValue('city'),
            "country": vehicleGr.getValue('country')
        };
        answer.push(vehicleObj);
    }

    // Generate the response body.
    response.setBody(answer);

})(request, response);
```

2. 获取多个汽车（get）(需要添加一些参数)

```javascript
/*
API:            Vehicles
Name:           Get Vehicles
HTTP Method:    GET
Relative path:  /vehicles
Description:    Returns data for selected vehicles.
*/

(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Declare empty array for answer
    var answer = [];

    // Get values from the respective query parameters.
    var make = request.queryParams.make;
    var model = request.queryParams.model;
    var year = request.queryParams.year;
    var country = request.queryParams.country;
    var city = request.queryParams.city;

    // Generate 404 if no query parameters provided.
    if (!make && !model && !year && !country && !city) {
        response.setError(new sn_ws_err.NotFoundError('No query parameters defined.'));
        response.setStatus(404);
        return;
    }

    // Query the Vehicle table using these parameters.
    var vehicleTable = 'x_1069741_wuyq54_0_vehicle';
    var vehicleGr = new GlideRecord(vehicleTable);
    if (make) { vehicleGr.addQuery('make', make); }
    if (model) { vehicleGr.addQuery('model', model); }
    if (year) { vehicleGr.addQuery('year', year); }
    if (country) { vehicleGr.addQuery('country', country); }
    if (city) { vehicleGr.addQuery('city', city); }
    vehicleGr.query();


    // Generate a 404 if no vehicles were found.
    if (!vehicleGr.hasNext()) {
        response.setError(new sn_ws_err.NotFoundError('No records found.'));
        response.setStatus(404);
        return;
    }

    // For the vehicles found, generate a JSON object and save the vehicle data in it.
    while (vehicleGr.next()) {

        var vehicleObj = {
            "make": vehicleGr.getValue('make'),
            "model": vehicleGr.getValue('model'),
            "vin": vehicleGr.getValue('vin'),
            "year": parseInt(vehicleGr.getValue('year'), 10),
            "country": vehicleGr.getValue('country'),
            "city": vehicleGr.getValue('city'),
            "start_date": vehicleGr.getValue('start_date'),
            "end_date": vehicleGr.getValue('end_date'),
        };
        answer.push(vehicleObj);
    }

    // Generate the response body.
    response.setBody(answer);

})(request, response);
```



* postman 请求

```shell
https://dev146324.service-now.com/api/1069741/vehicles/vehicles?make=Audi
```

### post

```javascript
/*
API:            Vehicles
Name:           Create Vehicles
HTTP Method:    POST
Relative path:  /vehicles
Description:    Create vehicle(s) from request body data.
*/

(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Declare empty array for answer
    var answer = [];

    // Declare variables for the possible names in the request body.
    var make;
    var model;
    var vin;
    var year;
    var country;
    var city;

    // Get the request body, a JSON object.
    var requestBody = request.body;

    // Convert JSON object to a Javascript string using the RESTAPIRequestBody.datastring() method.
    var requestString = requestBody.dataString;

    // Convert Javascript string to an array.
    var result = JSON.parse(requestString);

    // For every vehicle in the array, create a record and push the sys_id and VIN of the record to an object.
    try {
        for (i in result.vehicles) {
            make = result.vehicles[i].make;
            model = result.vehicles[i].model;
            vin = result.vehicles[i].vin;
            year = result.vehicles[i].year;
            country = result.vehicles[i].country;
            city = result.vehicles[i].city;

            var vehicleTable = 'x_1069741_wuyq54_0_vehicle';
            var vehicleGr = new GlideRecord(vehicleTable);
            vehicleGr.initialize();
            vehicleGr.make = make;
            vehicleGr.model = model;
            vehicleGr.vin = vin;
            vehicleGr.year = year;
            vehicleGr.country = country;
            vehicleGr.city = city;
            vehicleGr.insert();

            var vehicleObj = {
                "vin": vehicleGr.vin,
                "sys_id": vehicleGr.sys_id
            };
            answer.push(vehicleObj);

        }
        response.setStatus(201);

    } catch (error) {
        var serviceError = new sn_ws_err.ServiceError();
        serviceError.setStatus(500);
        serviceError.setMessage('Service error');
        serviceError.setDetail('The server could not process the request.');
        response.setError(serviceError);
    }


    // Generate the response body. Contains all vehicle records created.
    response.setBody(answer);


})(request, response);
```



```json
{
    "vehicles": [
        {
            "make": "Mahindra",
            "model": "XUV700",
            "vin": "WDCGG0EB6DG730474",
            "year": "2020",
            "country": "India",
            "city": "Bangalore"

        },
        {
            "make": "Ford",
            "model": "Mustang",
            "vin": "1FTEX1CM4BF996681",
            "year": "2022",
            "country": "United States",
            "city": "Las Vegas"
        }
    ]
}
```



### put and patch

*  Update Vehicle.js

```shell
/*
API:            Vehicles
Name:           Update Vehicle
HTTP Method:    PATCH
Relative path:  /vehicle/{vin}
Description:    Update vehicle with provided data.
*/

(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Declare empty array for answer
    var answer = [];

    // Get the VIN from the path parameter.
    var vin = request.pathParams.vin.toString();

    // Declare variables for the possible names in the request body.
    var requestBody = request.body.data;
    var make = requestBody.make;
    var model = requestBody.model;
    var year = requestBody.year;
    var country = requestBody.country;
    var city = requestBody.city;

    //Generate 404 if VIN not provided.
    if (!vin) {
        response.setError(new sn_ws_err.NotFoundError('No VIN defined.'));
        response.setStatus(404);
        return;
    }

    // Generate 404 if request body contains no or invalid data.
    if (!make && !model && !year && !country && !city) {
        response.setError(new sn_ws_err.NotFoundError('Object contains no or invalid properties.'));
        response.setStatus(404);
        return;
    }

    // Query the Vehicle table for this VIN.
    var vehicleTable = 'x_1069741_wuyq54_0_vehicle';
    var vehicleGr = new GlideRecord(vehicleTable);
    vehicleGr.addQuery('vin', vin);
    vehicleGr.query();

    // Generate a 404 if no vehicle was found.
    if (!vehicleGr.hasNext()) {
        response.setError(new sn_ws_err.NotFoundError('No vehicle found.'));
        response.setStatus(404);
        return;
    }

    // For the vehicle found, update the record with the details in the request body.
    if (vehicleGr.next()) {
		if (make) {vehicleGr.make = make;}
		if (model) {vehicleGr.model = model;}
		if (year) {vehicleGr.year = year;}
		if (country) {vehicleGr.country = country;}
		if (city) {vehicleGr.city = city;}
        vehicleGr.update();
    }

    // Set a 200 status for success.
    response.setStatus(200);

    // Create a JSON object with details of the updated record.
    var vehicleObj = {
        "make": vehicleGr.make,
        "model": vehicleGr.model,
        "vin": vehicleGr.vin,
        "year": vehicleGr.year,
        "country": vehicleGr.country,
        "city": vehicleGr.city,
        "sys_id": vehicleGr.sys_id
    };
    answer.push(vehicleObj);

    // Generate the response body.
    response.setBody(answer);


})(request, response);
```



* Update Vehicles – Request Body Examples.json

```json
{
    "_comment": "Request Body Example 1: All fields that can be updated.",
    "make": "Volvo",
    "model": "XC40",
    "year": "2021",
    "country": "Sweden",
    "city": "Helsingborg"
}

{
    "_comment": "Request Body Example 2: Subset of fields that can be updated.",
    "country": "Sweden",
    "city": "Helsingborg"
}
```



### delete

* Delete Vehicle.js

```javascript
/*
API:            Vehicles
Name:           Delete Vehicle
HTTP Method:    DELETE
Relative path:  /vehicle/{vin}
Description:    Delete selected vehicle.
*/

(function process( /*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {

    // Declare empty array for answer
    var answer = [];

    // Get the VIN from the path parameter. Generate 404 if not provided.
    var vin = request.pathParams.vin;
    if (!vin) {
        response.setError(new sn_ws_err.NotFoundError('VIN not provided.'));
        response.setStatus(404);
        return;
    }

    // Query the Vehicle table for this VIN.
    var vehicleTable = 'x_1069741_wuyq54_0_vehicle';
    var vehicleGr = new GlideRecord(vehicleTable);
    vehicleGr.addQuery('vin', vin);
    vehicleGr.query();

    // Generate a 404 if no vehicle was found.
    if (!vehicleGr.hasNext()) {
        response.setError(new sn_ws_err.NotFoundError('No vehicle found.'));
        response.setStatus(404);
        return;
    }

    // For the vehicle found, generate a JSON object and save the vehicle data in it.
    while (vehicleGr.next()) {

        vehicleGr.deleteRecord();

        var vehicleObj = {
            "vin": vehicleGr.getValue('vin'),
            "make": vehicleGr.getValue('make'),
            "model": vehicleGr.getValue('model'),
            "year": parseInt(vehicleGr.getValue('year'), 10),
            "city": vehicleGr.getValue('city'),
            "country": vehicleGr.getValue('country'),
            "message": 'Vehicle deleted.'
        };

        answer.push(vehicleObj);

    }

    // Generate the response body.
    response.setBody(answer);

})(request, response);
```





# Scripted REST Service

> 未完待补充

1. studio

2. scripted rest apis
3. ui pages

creat ui page

* Name: vehicles_api_documentation