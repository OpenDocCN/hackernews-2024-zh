<!--yml

分类：未分类

日期：2024-05-27 13:14:44

-->

# AI互联网的API接口

> 来源：[https://cloudcruise.com/](https://cloudcruise.com/)

```
curl -X POST https://api.cloudcruise.com/run \
     -H "cc-key: <apiKey>" \
     -H "Content-Type: application/json" \
     -d '{
 "workflow_id": "gusto_add_employee",
 "run_input_variables": {
 "first_name": "Galileo",
 "last_name": "Galilei",
 "email": "galileo@cloudcruise.com"
 }
 }' 
```
