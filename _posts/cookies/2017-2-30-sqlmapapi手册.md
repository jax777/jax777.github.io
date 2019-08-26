---
layout: post
title: sqlmapapi手册
categories: sql注入
tag: tips
---
The full list of methods available are:
```
@get("/task/new")
@get("/task/<taskid>/delete")
@get("/admin/<taskid>/list")
@get("/admin/<taskid>/flush")
@get("/option/<taskid>/list")
@post("/option/<taskid>/get")
@post("/option/<taskid>/set")
@post("/scan/<taskid>/start")
@get("/scan/<taskid>/stop")
@get("/scan/<taskid>/kill")
@get("/scan/<taskid>/status")
@get("/scan/<taskid>/data")
@get("/scan/<taskid>/log/<start>/<end>")
@get("/scan/<taskid>/log")
@get("/download/<taskid>/<target>/<filename:path>")


These are the methods I have been using
GET /task/new
Response:
{
    "taskid": "1d47d7f046df1504"
}



GET /task/<task_id>/delete
Response:
{
    "success": true
}



GET /option/<task_id>/list Response:
{
    "options": {
        "crawlDepth": null,
        "osShell": false,
        "getUsers": false,
        "getPasswordHashes": false,
        "excludeSysDbs": false,
        "uChar": null,
        "regData": null,
        "cpuThrottle": 5,
        "prefix": null,
        "code": null,
        "googlePage": 1,
        "query": null,
        "randomAgent": false,
        "delay": 0,
        "isDba": false,
        "requestFile": null,
        "predictOutput": false,
        "wizard": false,
        "stopFail": false,
        "forms": false,
        "taskid": "73674cc5eace4ac7",
        "skip": null,
        "dropSetCookie": false,
        "smart": false,
        "risk": 1,
        "sqlFile": null,
        "rParam": null,
        "getCurrentUser": false,
        "notString": null,
        "getRoles": false,
        "getPrivileges": false,
        "testParameter": null,
        "tbl": null,
        "charset": null,
        "trafficFile": null,
        "osSmb": false,
        "level": 1,
        "secondOrder": null,
        "pCred": null,
        "timeout": 30,
        "firstChar": null,
        "updateAll": false,
        "binaryFields": false,
        "checkTor": false,
        "aType": null,
        "direct": null,
        "saFreq": 0,
        "tmpPath": null,
        "titles": false,
        "getSchema": false,
        "identifyWaf": false,
        "checkWaf": false,
        "regKey": null,
        "limitStart": null,
        "loadCookies": null,
        "dnsName": null,
        "csvDel": ",",
        "oDir": null,
        "osBof": false,
        "invalidLogical": false,
        "getCurrentDb": false,
        "hexConvert": false,
        "answers": null,
        "host": null,
        "dependencies": false,
        "cookie": null,
        "proxy": null,
        "regType": null,
        "optimize": false,
        "limitStop": null,
        "mnemonics": null,
        "uFrom": null,
        "noCast": false,
        "testFilter": null,
        "eta": false,
        "threads": 1,
        "logFile": null,
        "os": null,
        "col": null,
        "rFile": null,
        "verbose": 1,
        "aCert": null,
        "torPort": null,
        "privEsc": false,
        "forceDns": false,
        "getAll": false,
        "api": true,
        "url": null,
        "invalidBignum": false,
        "regexp": null,
        "getDbs": false,
        "freshQueries": false,
        "uCols": null,
        "smokeTest": false,
        "pDel": null,
        "wFile": null,
        "udfInject": false,
        "tor": false,
        "forceSSL": false,
        "beep": false,
        "saveCmdline": false,
        "configFile": null,
        "scope": null,
        "dumpAll": false,
        "torType": "HTTP",
        "regVal": null,
        "dummy": false,
        "commonTables": false,
        "search": false,
        "skipUrlEncode": false,
        "referer": null,
        "liveTest": false,
        "purgeOutput": false,
        "retries": 3,
        "extensiveFp": false,
        "dumpTable": false,
        "database": "/tmp/sqlmapipc-EmjjlQ",
        "batch": true,
        "headers": null,
        "flushSession": false,
        "osCmd": null,
        "suffix": null,
        "dbmsCred": null,
        "regDel": false,
        "shLib": null,
        "nullConnection": false,
        "timeSec": 5,
        "msfPath": null,
        "noEscape": false,
        "getHostname": false,
        "sessionFile": null,
        "disableColoring": true,
        "getTables": false,
        "agent": null,
        "lastChar": null,
        "string": null,
        "dbms": null,
        "tamper": null,
        "hpp": false,
        "runCase": null,
        "osPwn": false,
        "evalCode": null,
        "cleanup": false,
        "getBanner": false,
        "profile": false,
        "regRead": false,
        "bulkFile": null,
        "safUrl": null,
        "db": null,
        "dumpFormat": "CSV",
        "alert": null,
        "user": null,
        "parseErrors": false,
        "aCred": null,
        "getCount": false,
        "dFile": null,
        "data": null,
        "regAdd": false,
        "ignoreProxy": false,
        "getColumns": false,
        "mobile": false,
        "googleDork": null,
        "sqlShell": false,
        "pageRank": false,
        "tech": "BEUSTQ",
        "textOnly": false,
        "commonColumns": false,
        "keepAlive": false
    }
}



POST /option/<task_id>/set -- Content-Type:application/json
Request:
{ "msfPath" : "/path/to/metasploit/framework" }
Response:
{
    "success": true
}



POST /scan/<task_id>/start -- Content-Type:application/json
Request (optional):
{ "url" : "192.168.1.250/index.php?wut=injectable" }
Response:
{
    "engineid": 16784,
    "success": true
}



GET /scan/<task_id>/log
Response:
{
    "log": [
        {
            "message": "testing connection to the target URL",
            "level": "INFO",
            "time": "14:11:23"
        },
        {
            "message": "testing if the target URL is stable. This can take a couple of seconds",
            "level": "INFO",
            "time": "14:11:24"
        },
        {
            "message": "target URL is stable",
            "level": "INFO",
            "time": "14:11:26"
        },
        {
            "message": "no parameter(s) found for testing in the provided data (e.g. GET parameter 'id' in 'www.site.com/index.php?id=1')",
            "level": "CRITICAL",
            "time": "14:11:26"
        },
        {
            "message": "testing connection to the target URL",
            "level": "INFO",
            "time": "14:17:30"
        },
        {
            "message": "testing if the target URL is stable. This can take a couple of seconds",
            "level": "INFO",
            "time": "14:17:31"
        },
        {
            "message": "target URL is not stable. sqlmap will base the page comparison on a sequence matcher. If no dynamic nor injectable parameters are detected, or in case of junk results, refer to user's manual paragraph 'Page comparison' and provide a string or regular expression to match on",
            "level": "WARNING",
            "time": "14:17:33"
        },
        {
            "message": "testing if GET parameter 'PAGE' is dynamic",
            "level": "INFO",
            "time": "14:17:33"
        },
        {
            "message": "confirming that GET parameter 'PAGE' is dynamic",
            "level": "INFO",
            "time": "14:17:33"
        },
        {
            "message": "GET parameter 'PAGE' does not appear dynamic",
            "level": "WARNING",
            "time": "14:17:33"
        },
        {
            "message": "reflective value(s) found and filtering out",
            "level": "WARNING",
            "time": "14:17:33"
        },
        {
            "message": "heuristic (basic) test shows that GET parameter 'PAGE' might not be injectable",
            "level": "WARNING",
            "time": "14:17:33"
        },
        {
            "message": "testing for SQL injection on GET parameter 'PAGE'",
            "level": "INFO",
            "time": "14:17:34"
        },
        {
            "message": "testing 'AND boolean-based blind - WHERE or HAVING clause'",
            "level": "INFO",
            "time": "14:17:34"
        },
        {
            "message": "testing 'MySQL >= 5.0 AND error-based - WHERE or HAVING clause'",
            "level": "INFO",
            "time": "14:17:34"
        },
        {
            "message": "testing 'PostgreSQL AND error-based - WHERE or HAVING clause'",
            "level": "INFO",
            "time": "14:17:34"
        },
        {
            "message": "testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause'",
            "level": "INFO",
            "time": "14:17:34"
        },
        {
            "message": "testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'MySQL inline queries'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'PostgreSQL inline queries'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'Microsoft SQL Server/Sybase inline queries'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'Oracle inline queries'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'SQLite inline queries'",
            "level": "INFO",
            "time": "14:17:35"
        },
        {
            "message": "testing 'MySQL > 5.0.11 stacked queries'",
            "level": "INFO",
            "time": "14:17:36"
        },
        {
            "message": "testing 'PostgreSQL > 8.1 stacked queries'",
            "level": "INFO",
            "time": "14:17:36"
        },
        {
            "message": "testing 'Microsoft SQL Server/Sybase stacked queries'",
            "level": "INFO",
            "time": "14:17:36"
        },
        {
            "message": "testing 'MySQL > 5.0.11 AND time-based blind'",
            "level": "INFO",
            "time": "14:17:36"
        },
        {
            "message": "testing 'PostgreSQL > 8.1 AND time-based blind'",
            "level": "INFO",
            "time": "14:17:37"
        },
        {
            "message": "testing 'Microsoft SQL Server/Sybase time-based blind'",
            "level": "INFO",
            "time": "14:17:37"
        },
        {
            "message": "testing 'Oracle AND time-based blind'",
            "level": "INFO",
            "time": "14:17:37"
        },
        {
            "message": "testing 'MySQL UNION query (NULL) - 1 to 10 columns'",
            "level": "INFO",
            "time": "14:17:37"
        },
        {
            "message": "testing 'Generic UNION query (NULL) - 1 to 10 columns'",
            "level": "INFO",
            "time": "14:17:38"
        },
        {
            "message": "using unescaped version of the test because of zero knowledge of the back-end DBMS. You can try to explicitly set it using option '--dbms'",
            "level": "WARNING",
            "time": "14:17:38"
        },
        {
            "message": "GET parameter 'PAGE' is not injectable",
            "level": "WARNING",
            "time": "14:17:39"
        },
        {
            "message": "all tested parameters appear to be not injectable. Try to increase '--level'/'--risk' values to perform more tests. Also, you can try to rerun by providing either a valid value for option '--string' (or '--regexp')",
            "level": "CRITICAL",
            "time": "14:17:40"
        },
        {
            "message": "HTTP error codes detected during run:\n404 (Not Found) - 183 times",
            "level": "WARNING",
            "time": "14:17:40"
        }
    ]
}

GET /scan/<task_id>/status
Response:
{
    "status": "terminated",
    "returncode": 0
}
```
