# ServiceNow Tips

ServiceNow Tips for daily task

## Query data from exact record

```javascript
var g = new GlideRecord("sc_req_item");
if (g.get(/* sys_id */ "dd6b061a87f8c55083d9edb83cbb3583")) {
  gs.print(g.getDisplayValue("number"));
}
```

## Query data from table

```javascript
var g = new GlideRecord("sc_req_item");
g.addQuery("state", "1");
g.setLimit(100);
g.query();
while (g.next()) {
  gs.print(g.getDisplayValue("number"));
}
```

## Query data with time and list of values

```javascript
var user_sys_id = gs.getUserID();
var sysapproval_approvers = [];

var approved_approvalGr = new GlideRecord("sysapproval_approver");
approved_approvalGr.addQuery("approver", user_sys_id);
approved_approvalGr.addQuery(
  "state",
  "IN",
  "approved,rejected,not_required,not requested"
);
approved_approvalGr.addQuery("sys_updated_on", gs.endOfLast15Minutes());
approved_approvalGr.addActiveQuery();
approved_approvalGr.query();

while (approved_approvalGr.next())
  sysapproval_approvers.push(approved_approvalGr.sysapproval.sys_id);
```

## Query with encoded query

```javascript
var sysapproval_approvers = [];

var approved_approvalGr = new GlideRecord("sysapproval_approver");
approved_approvalGr.addEncodedQuery(
  "approver=" +
    user_sys_id +
    "^sys_updated_on>javascript:gs.beginningOfLast15Minutes()^stateINapproved,rejected"
);
approved_approvalGr.query();

while (approved_approvalGr.next())
  sysapproval_approvers.push(approved_approvalGr.sysapproval.sys_id);
```

## Create RITM with variables

```javascript
var cartId = GlideGuid.generate(null);
var cart = new Cart(cartId);
var item = cart.addItem(
  /* Cat item sys_id */ "6e0ce006975545506adcf5efe153axxx",
  1
);

// sc_cat_item fields
cart.setVariable(
  item,
  "assignment_group",
  /* Group sys_id */ "667533d487b538d07545edb83cbb3xxx"
);

// variables fields
cart.setVariable(item, "u_other", "value");

var rc = cart.placeOrder();
```

## Aggregate

```javascript
var g = new GlideAggregate(table);
g.addEncodedQuery(couter_query);
g.addAggregate("COUNT");
g.query();

if (g.next()) data.counter = parseInt(g.getAggregate("COUNT"), 10);
```

## Set log

```javascript
var gl = new GSLog("pl.bmalz.log", "sampleIntegration");
gl.setLevel("debug");
gl.logInfo("Sample info");
gl.logErr("Sample error");
```

## Execute Data Set, load Import Set, execute Transform Map and cleanup Import Set

```javascript
var loader = new DataSourceLoader();
loader.load(
  /* Data Source Name */ "Example Data Source Name",
  /* Transform Map Name */ "Transform Map Name",
  /* Delete Transform Map */ true
);
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
  clean: true,
});
```

## Fetch data from widget Server script

### Server script

```javascript
if (input.op == "SAMPLEOP") {
  try {
    var r = new sn_ws.RESTMessageV2("SampleRest", "Sample POST");
    r.setRequestBody(JSON.stringify(input.payload));

    var response = r.execute();
    var httpStatus = response.getStatusCode();

    if (httpStatus == 200) data.response = response.getBody();
  } catch (ex) {
    var message = ex.message;
  }
}
```

### Client controller

```javascript
c.server
  .get({
    op: "SAMPLEOP",
    payload: payload,
  })
  .then(function (response) {
    console.log(response.data.response);
  });
```

## Communicate between widgets

### Sender Client script

```javascript
setTimeout(function () {
  $rootScope.$broadcast("custom_event_name", true);
});
```

### Receiver Client script

```javascript
$rootScope.$on("custom_event_name", function (event, data) {
  console.log(data);
});
```

## Focus on tab (Client Script)

```javascript
getMessage("Information", function(msg) {
		$j('#tabs2_section .tab_caption_text').each(function(i, e) { 
			if(e.innerText == msg)
				g_tabs2Sections.setActive(i);
		});
	});
```
