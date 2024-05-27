<!--yml
category: 未分类
date: 2024-05-27 13:14:44
-->

# APIs for the AI Internet

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