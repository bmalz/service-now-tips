# ServiceNow Tips

ServiceNow Tips for daily task

## Query data from exact record

```javascript
var g = new GlideRecord('sc_req_item')
if(g.get(/* sys_id */ 'dd6b061a87f8c55083d9edb83cbb3583'))
{
    gs.print(g.getDisplayValue('number'))
}
```

## Query data from table

```javascript
var g = new GlideRecord('sc_req_item')
g.addQuery('state', '1');
g.setLimit(100);
g.query();
while(g.next()) {
    gs.print(g.getDisplayValue('number'));
}
```

## Set log

```javascript
var gl = new GSLog('pl.bmalz.log', 'sampleIntegration');
gl.setLevel('debug');
gl.logInfo("Sample info");
gl.logErr("Sample error");
```
