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

## Execute Data Set, load Import Set, execute Transform Map and cleanup Import Set

```javascript
var loader = new DataSourceLoader();
loader.load(/* Data Source Name */ "Example Data Source Name", /* Transform Map Name */ "Transform Map Name", /* Delete Transform Map */ true);
```

Detailed function

```javascript
function execute(props) {
  var sourceGr = new GlideRecord("sys_data_source");
  sourceGr.addQuery("name", props.dataSourceName);
  sourceGr.query();
  if (!sourceGr.next()) {
    gs.print("Did not find Data Source " + props.dataSourceName);
    return;
  }

  var map = null;
  if (!gs.nil(props.transforMapName)) {
    var map = new GlideRecord("sys_transform_map");
    map.addQuery("name", props.transforMapName);
    map.query();
    if (!map.next()) {
      gs.print("Did not find Transform map " + props.transforMapName);
      return;
    }
  }

  var loader = new GlideImportSetLoader();
  var importSetGr = loader.getImportSetGr(sourceGr);
  var ranload = loader.loadImportSetTable(importSetGr, sourceGr);

  if (!ranload) {
    gs.print("Failed to load import set " + source);
    return;
  }

  if (!gs.nil(props.transforMapName) && map) {
    var t = new GlideImportSetTransformer();
    t.setMapID(map.sys_id);
    t.transformAllMaps(importSetGr);
  }

  if (props.clean) {
    var cleaner = new ImportSetCleaner(importSetGr.table_name);
    cleaner.setDataOnly(true);
    cleaner.setDeleteMaps(cleanMap);
    cleaner.clean();
  }
}

execute({ 
    dataSourceName: "Sample Source Name",
    transforMapName: "Sample Transform Map Name",
    clean: true
});
```
