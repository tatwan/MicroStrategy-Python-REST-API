



# MicroStrategy-Python-REST-API

<a id='top'></a>

### MicroStrategy Reference Material:
*  MicroStrategy RESTful API Interactive (your Local Demo): http://yourmstrEnv.com/MicroStrategyLibrary/api-docs/
*  [MicroStrategy RESTful API Interactive (external demo)](https://demo.microstrategy.com/MicroStrategyLibrary/api-docs/index.html#/)
*  [MicroStrategy REST API Online Documentation](https://lw.microstrategy.com/msdz/MSDL/GARelease_Current/docs/projects/RESTSDK/Content/topics/REST_API/REST_API.htm)

### Additional Resources/Inspirations:
* [MicroStrategy Sample API Python Example by Robert Prochowicz](https://community.microstrategy.com/s/article/REST-API-10-9-code-example-in-Python)
* [Machine Learning with Python On-Demand Video with Scott Rigney](https://www.microstrategy.com/us/resources/library/webcasts/machine-learning-with-python-train-models-on-trust)

### Python Library References:
* [Python Requests](http://docs.python-requests.org/en/master/)
* [Python JSON](https://docs.python.org/3/library/json.html)
* [Python Pandas](https://pandas.pydata.org/)


### List of Code Examples :
1. [login()](#auth) 

   ```python
   authToken, cookies = login(baseURL,username,password)
   ```

2. [sessionValidate()](#test)

   ```python
   sessionValiade(baseURL, authToken, cookies)
   ```

3. [userInfo()](#user)

   ```python
   user = userInfo(baseURL, authToken, cookies)
   ```

4. [listProjcets()](#projects)   

   ```python
   projectList = listProjects(baseURL, authToken, cookies)
   ```

   

5. [getLibrary()](#library)

   ```python
   libraryInfo = getLibrary(baseURL, authToken, cookies, 'FILTER_TOC')
   ```

6. [searchObjects()](#search)

   ```python
   mySearch = searchObjects(baseURL, authToken, '39')
   ```

7. [cubeObjects()](#cubeobjects)

   ```python
   cObjects = cubeObjects(baseURL, authToken, projectId, cookies, 'BD23848347017FC2C0B4509AED1AF7B4')
   ```

8. [Logout User()](#exit)  

  ```python
  logout(baseURL, authToken)
  ```



** Work In Progress **

1. [Cube Instance](#cubeinstance)
2. [Get Cube Data](#cubedata)
3. [Create Cube](#createcube)
4. [Write/publish a Cube](#writecube)
5. More .....



### Import Python Libraries 

```python
import requests
import json
from pandas.io.json import json_normalize
import pandas as pd
import numpy as np
```

## Set Parameters 

Create the necessary varibales such as `username`, `password`, `projectid` and `baseURL`

```python
### Parameters ###
username = 'Administrator'
password = ''
iserver = '10.254.113.99'
projectId = 'B19DEDCC11D4E0EFC000EB9495D0F44F'
projectName = 'MicroStrategy Tutorial'
baseURL = "http://yourmstrEnv.com/MicroStrategyLibrary/api/" #replace with your own URL for MicroStrategy Library API
```

<a id='auth'></a>

## Authentication: Returns authToken & SessionId
[Top](#top)

Implementation Notes (source: MicroStrategy Documentation):  
Authenticate a user and create an HTTP session on the web server where the user’s MicroStrategy sessions are stored. This request returns an authorization token (X-MSTR-AuthToken) which will be submitted with subsequent requests. The body of the request contains the information needed to create the session. The loginMode parameter in the body specifies the authentication mode to use. You can authenticate with one of the following authentication modes: Standard (1), Anonymous (8), or LDAP (16). Authentication modes can be enabled through the System Administration REST APIs, if they are supported by the deployment. If you are not able to authenticate using any of the authentication modes, please contact your administrator to determine current support or currently enabled authentication modes.

```python
def login(baseURL,username,password):
    """
    Authenticate a user and create an HTTP session on the web server.
    
    Parameters:
    -----------
    baseURL, username, password
    
    Returns:
    --------
    authToken and sessionId.
    
    Example:
    --------
    authToken, cookies = login(baseURL, username, password)
    """
    header = {'username': username,
                'password': password,
                'loginMode': 1}
    r = requests.post(baseURL + 'auth/login', data=header)
    if r.ok:
        authToken = r.headers['X-MSTR-AuthToken']
        cookies = dict(r.cookies)
        print("Token: " + authToken)
        print("Session ID: {}".format(cookies))
        return authToken, cookies
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []

```

```python
authToken, cookies = login(baseURL,username,password)
```

```bash
>> output
Token: 5q4mb2nlcpk434ol4ors52sb5h
Session ID: {'JSESSIONID': '3F23A282B4A7FAE9BB4E99C50EDA4321'}
```



<a id='test'></a>

## Test Session
[Top](#top)

Implementation Notes (source: MicroStrategy Documentation):  
Get information about a configuration session. You obtain the authorization token needed to execute the request using POST /auth/login; you pass the authorization token in the request header. Each time you call this endpoint, both the HTTP and Intelligence Server session timeouts are reset. This request returns information about the authenticated user, locale, timeout duration, maximum number of concurrent searches, and limit on number of instances kept in memory.



```python
def sessionValiade(baseURL, authToken, cookies):
    """
    Get information about a configuration session
    
    Parameters:
    ----------
    baseURL, authToken, cookies
    
    Returns:
    -------
    None
    
    Example:
    --------
    sessionValiade(baseURL, authToken, cookies)
    
    """
    print("Checking session...")
    header = {'X-MSTR-AuthToken': authToken,
                 'Accept': 'application/json'}
    r = requests.get(baseURL + "sessions", headers=header, cookies=cookies)
    
    if r.ok:
        print(r.text)
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
```

```python
sessionValiade(baseURL, authToken, cookies)
```

```bash
>> output
Checking session...
{"locale":1033,"maxSearch":3,"workingSet":10,"timeout":600,"id":"54F3D26011D2896560009A8E67019608","fullName":"Administrator","initials":"A"}
```



<a id='user'></a>

## Get UserInfo
[Top](#top)

```python
def userInfo(baseURL, authToken, cookies):
    """
    Returns:
    --------
    Pandas DataFrame
    id, fullName, initials
    
    Example:
    --------
    user = userInfo(baseURL, authToken, cookies)
    """
    header = {'X-MSTR-AuthToken': authToken,
                 'Accept': 'application/json'}
    r = requests.get(baseURL + "sessions/userInfo", headers=header, cookies=cookies)
    
    if r.ok:
        return json_normalize(json.loads(r.text))
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
    
```

```python
user = userInfo(baseURL, authToken, cookies)
```

\>> output

|      | fullName      | id                               | initials | metadataUser |
| ---- | ------------- | -------------------------------- | -------- | ------------ |
| 0    | Administrator | 54F3D26011D2896560009A8E67019608 | A        | True         |



<a id='library'></a>

## Get Library for user
[Top](#top)

Implementation Notes (source: MicroStrategy Documentation)  
Get the library for the authenticated user. You obtain the authorization token needed to execute the request using POST /auth/login; you pass the authorization token in the request header.

```python
    """
    Get library for authenticated user.
    
    Parameteres:
    ------------
    baseURL, authToken, cookies, flag.
    flag: 'DEFAULT' or'FILTER_TOC'
    
    Returns:
    --------
    Pandas DataFrame (pandas.core.frame.DataFrame)
    id, name, description, projectId, active, lastViewedTime
    
    Example:
    --------
    getLibrary(baseURL, authToken, cookies, 'DEFAULT')
    """
    
    header = {'X-MSTR-AuthToken': authToken,
                 'Accept': 'application/json'}
    r = requests.get(baseURL + "library?outputFlag="+ flag, headers=header, cookies=cookies)
    
    if r.ok:            
        a = pd.DataFrame(json.loads(r.text))[['id', 'name', 'projectId', 'active','lastViewedTime']]
        tmp = []
        if (flag == 'DEFAULT'):
            for i in json.loads(r.text):
                tmp.append(i['target']['id'])
            a['target'] = pd.DataFrame(tmp).astype(str)
        return a
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
```

```python
libraryInfo = getLibrary(baseURL, authToken, cookies, 'FILTER_TOC')
```

\>> output

|      | id                               | name                                 | projectId                        | active | lastViewedTime               |
| ---- | -------------------------------- | ------------------------------------ | -------------------------------- | ------ | ---------------------------- |
| 0    | 21A521BA4DB47ADAEBE19E9E9F7EC7D9 | Executive Business User Data Dossier | B19DEDCC11D4E0EFC000EB9495D0F44F | True   | 2018-08-08T16:57:48.000+0000 |
| 1    | 21A521BA4DB47ADAEBE19E9E9F7EC7D9 | Category Breakdown Dossier           | B19DEDCC11D4E0EFC000EB9495D0F44F | True   | 2018-08-08T16:59:08.000+0000 |

<a id='projects'></a>

## List of Projects
[Top](#top)

Implementation Notes (Source: MicroStrategy Documentation)  
Get a list of projects which the authenticated user has access to. This returns the name, ID, description, alias, and status of each project; the status corresponds to values from EnumDSSXMLProjectStatus. You obtain the authorization token needed to execute the request using POST /auth/login; you pass the authorization token in the request header.

```python
def listProjects(baseURL, authToken, cookies):   
    """
   Get a list of projects that can be accessed by the authenticated user
    
    Parameters:
    ----------
    baseURL, authToken, cookies
    
    Returns:
    -------
    Pandas DataFrame
    Project Id, Name, Description and Status code
    
    Example:
    --------
    sessionValiade(baseURL, authToken, cookies)
    
    """
    header = {'X-MSTR-AuthToken': authToken,
                 'Accept': 'application/json'}
    r = requests.get(baseURL + 'projects', headers=header, cookies=cookies)
    if r.ok:
        return pd.DataFrame(json.loads(r.text))[['id','name','description', 'status']]
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
```

```python
projectList = listProjects(baseURL, authToken, cookies)
```

\>> output

|      | id                               | name                            | description                                       | status |
| ---- | -------------------------------- | ------------------------------- | ------------------------------------------------- | ------ |
| 0    | B19DEDCC11D4E0EFC000EB9495D0F44F | MicroStrategy Tutorial          | MicroStrategy Tutorial project and application... | 0      |
| 1    | AF09B3E3458F78B4FBE4DEB68528BF7B | Human Resources Analysis Module | The Human Resources Analysis Module analyses w... | 0      |
| 2    | 4DD3B04B40D227471401609D630C76ED | Enterprise Manager              |                                                   | 0      |

<a id='search'></a>

## Search Objects
[Top](#top)

Implementation Notes (Source: MicroStrategy Documentation)  
Use the stored results of the Quick Search engine to return search results and display them as a list. The Quick Search engine periodically indexes the metadata and stores the results in memory, making Quick Search very fast but with results that may not be the most recent. You obtain the authorization token needed to execute the request using POST /auth/login. You identify the project by specifying the project ID in the request header; you obtain the project ID using GET /projects. You specify the search criteria using query parameters in the request; criteria can include the root folder ID, the search domain, the type of object, whether to return ancestors of the object, and a search pattern such as Begins With or Exactly. You use the offset and limit query parameters to control paging behavior. The offset parameter specifies where to start returning search results, and the limit parameter specifies how many results to return.



```python
def searchObjects(baseURL, authToken, stype):
    """
    Search for meteadata Objects using EnumDSSObjectType. 
    
    Parameters:
    -----------
    baseURL, authToken, stype
    stype is based on EnumDSSObjectType values for example Folder is 8, Search is 39, Metric is 4 and Attribute is 12
    for a lsit of EnumDSSObjectType values reference https://community.microstrategy.com/s/article/KB16048-List-of-all-object-types-and-object-descriptions-in
    
    Return:
    -------
    Pandas DataFrame which contains object ID, name, type, owner and additional details
    
    Example:
    --------
    searchObjects(baseURL, authToken, '8')
    
    """
    
    header = {'X-MSTR-AuthToken': authToken,
              'X-MSTR-ProjectID': projectId,
              'Accept': 'application/json'}
    
    r = requests.get(baseURL + 'searches/results?type='+ stype, headers=header, cookies=cookies)
    
    if r.ok:
        return pd.DataFrame(json.loads(r.text)['result'])
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
```

```python
mySearch = searchObjects(baseURL, authToken, '39')
```

\>> output

|      | acg  | dateCreated                  | dateModified                 | extType | id                               | name                                       | owner                                             | subtype | type | version                          |
| ---- | ---- | ---------------------------- | ---------------------------- | ------- | -------------------------------- | ------------------------------------------ | ------------------------------------------------- | ------- | ---- | -------------------------------- |
| 0    | 255  | 2005-06-27T21:33:41.000+0000 | 2010-09-13T10:40:53.000+0000 | 0       | 87F09D2EBB9B462CAC4581ABCAD97BBD | Search for all objects of type Grid        | {'name': 'Administrator', 'id': '54F3D26011D28... | 9984    | 39   | 08B3974B493CE1E84106EB825B71CB6A |
| 1    | 255  | 2005-06-27T21:33:42.000+0000 | 2005-06-27T21:33:42.000+0000 | 0       | 8A7CAF697BB64191BA3E15FA10DEDA61 | Search for all objects of type Text Prompt | {'name': 'Administrator', 'id': '54F3D26011D28... | 9984    | 39   | AC6316004E27925A85DDDF928D276A43 |
| 2    | 255  | 2010-04-12T11:13:59.000+0000 | 2010-04-12T11:13:59.000+0000 | 0       | 9F4A56074EDD734CBEFFC79A68BC36AF | MicroStrategy Web User Objects             | {'name': 'Administrator', 'id': '54F3D26011D28... | 9984    | 39   | 5726EAF84C05E5B3854423A0E8BA1106 |

<a id='cubeobjects'></a>

## List Cube Objects
[Top](#top)

(mplementation Notes (Source: MicroStrategy Documentation)  
Get the definition of a specific cube, including attributes and metrics. The cube can be either an Intelligent Cube or a Direct Data Access (DDA)/MDX cube. The in-memory cube definition provides information about all available objects without actually running any data query/report. The results can be used by other requests to help filter large datasets and retrieve values dynamically, helping with performance and scalability. You obtain the authorization token needed to execute the request using POST /auth/login; you pass the authorization token and the project ID in the request header. You specify the cube ID in the path of the request; this can be either an Intelligent cube ID or a DDA/MDX cube ID.

```python
def cubeObjects(baseURL, authToken, projectId, cookies, cubeId):
    """
    Get definition of a specific cube with cubeId 
    
    Parameters:
    -----------
    baseURL, authToken, projectId, cookies, cubeId
    
    Return:
    -------
    Pandas DataFrame which contains object ID, Object Name and Type (Attribute or Metrics)
    
    Example:
    --------
    cubeObjects(baseURL, authToken, projectId, cookies, 'BD23848347017FC2C0B4509AED1AF7B4')
    
    """
    header = {'X-MSTR-AuthToken': authToken,
                  'X-MSTR-ProjectID': projectId,
                 'Accept': 'application/json'}
    
    r = requests.get(baseURL + 'cubes/' + cubeId, headers=header, cookies=cookies)
    
    if r.ok:
        node = r.json()
        attr =  pd.DataFrame(node['result']['definition']['availableObjects']['attributes'])[['id', 'name', 'type']]
        mtrcs =  pd.DataFrame(node['result']['definition']['availableObjects']['metrics'])[['id', 'name', 'type']]
        return pd.concat([attr, mtrcs])
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
        return []
```

```python
cObjects = cubeObjects(baseURL, authToken, projectId, cookies, 'BD23848347017FC2C0B4509AED1AF7B4')
```

\>> output

|      | id                               | name          | type      |
| ---- | -------------------------------- | ------------- | --------- |
| 0    | 8D679D3811D3E4981000E787EC6DE8A4 | Country       | Attribute |
| 1    | 8D679D3611D3E4981000E787EC6DE8A4 | Catalog       | Attribute |
| 2    | 8D679D3711D3E4981000E787EC6DE8A4 | Gross Revenue | Metric    |



<a id='exit'></a>

##  Log Out and end session
[Top](#top)

```python
def logout(baseURL,authToken):
    
    header = {'X-MSTR-AuthToken': authToken,
                  'Accept': 'application/json'}

    r = requests.post(baseURL + 'auth/logout',headers=header, cookies=cookies)
    if r.ok:
        print("Logged Out")
       
    else:
        print("HTTP {} - {}, Message {}".format(r.status_code, r.reason, r.text))
```

```python
logout(baseURL, authToken)
```

```bash
>> output
Logged Out
```

