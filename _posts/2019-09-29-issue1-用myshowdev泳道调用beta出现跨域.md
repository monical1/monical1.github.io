---
title: 用myshowdev泳道调用beta出现跨域
tags: issues
---



用postman 或curl 直连OK

```nginx
curl 'http://venue.test.maoyan.com/myshow/api/report/show/distributorTicketGather?token=cikcWdzgHPODIeEKGYqNr89CIxtEdkp0BLGpDm4WWgpu24jp9iF9mFV3w-pRBYDjhta4W9w2YXPqMDhRtveTyw' -H 'Accept: application/json, text/plain, */*' -H 'Referer: http://localhost:8080/datas/report/2153439' -H 'swimlane: myshowdev' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3922.0 Safari/537.36' -H 'Content-Type: application/json;charset=UTF-8' --data-binary '{"startTime":"2017-09-30 00:00:00","endTime":"2019-09-29 23:59:59","projectShowIdList":[379162],"projectId":"2153439","timeType":0}' --compressed
```



在local 域名下访问

```js
fetch('http://venue.test.maoyan.com/myshow/api/report/show/distributorTicketGather?projectId=2153460&token=VRAS7QiihMWZcTgPGwmz3VxzsPqEk8Ac28kvk7fLzbwIGs36BFQxG6RFr4JHCdjPDi2hvbihldcUdfOxG1KnTA', {method: 'POST', credentials: 'include', cache: 'no-cache', redirect: 'manual', mode: 'cors', referrerPolicy: 'no-referrer', body: JSON.stringify({
	"projectId":2153460,
	"projectShowIdList": [379223]
}), headers: {'swimlane':'myshowdev'} }).then(rep=>{ console.log(JSON.stringify(rep)); alert(" ") }) .catch(reason => { console.error(reason); });
```

Option会出现404 和跨域CORS



暂时解决方案

用dev_conflict_20190911 分支merge 到light_merge 里