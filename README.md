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

## Send an attachment from widget

### HTML template

```html
<div class="m-b-xs m-t-xs">
  <button class="btn btn-primary btn-block" ng-click="attachFile();"> ${Submit}</button>
</div>
<input type="file" id="widget-multi-file-input" onchange="angular.element(this).scope().sendAttachments();" multiple
       style="display:none" />
```

### Server Script

```javascript
(function() {
    data.sys_id = input.sys_id || options.record_id || $sp.getParameter("sys_id");
    data.table = input.table || options.record_table || $sp.getParameter("table");
    data.maxAttachmentSize = parseInt(gs.getProperty("com.glide.attachment.max_size", 1024));
    if (isNaN(data.maxAttachmentSize))
        data.maxAttachmentSize = 24;
    data.largeAttachmentMsg = gs.getMessage("Attached files must be smaller than {0} - please try again", "" + data.maxAttachmentSize + "MB");
    data.attachmentSuccessMsg = gs.getMessage("Attachment successfully uploaded");
    data.noDragDropMsg = gs.getMessage("Attachment drap-and-drop not enabled - use attachment button");
    data.noAttachmentsMsg = gs.getMessage("There are no attachments");
})();
```

### Client Controller

```javascript
api.controller = function($scope, $rootScope, $timeout, nowAttachmentHandler, spAttachmentUpload, spAriaUtil, spUtil, $uibModal, $log) {
    var c = this;

    $scope.errorMessages = [];

    var files_input = $('#widget-multi-file-input');
    $scope.attachmentHandler = new nowAttachmentHandler(setAttachments, appendError);

    $scope.$evalAsync(function() {
        $scope.attachmentHandler.setParams($scope.data.table, $scope.data.sys_id, 1024 * 1024 * $scope.data.maxAttachmentSize);
    });

    function appendError(error) {
        $scope.errorMessages.push(error);
        spUtil.addErrorMessage(error.msg + error.fileName);
    }

    function setAttachments(attachments, action) {
        if ($scope.submitting == true)
            return;

        $scope.attachments = attachments;
        if (!action)
            return;

        if (action === "added") {
            spAriaUtil.sendLiveMessage($scope.data.attachmentSuccessMsg);
            spUtil.addInfoMessage("Attachmend added");
        }

        $scope.data.action = action;
        spUtil.update($scope);
    }

    $scope.attachFile = function() {
        files_input.click();
    }

    $scope.sendAttachments = function() {
        spAttachmentUpload.uploadAttachments($scope.attachmentHandler, files_input[0].file);
        spUtil.update($scope);
    }

    $scope.$on('dialog.upload_too_large.show', function(e) {
        $log.error($scope.data.largeAttachmentMsg);
        spUtil.addErrorMessage($scope.data.largeAttachmentMsg);
    });

    $scope.$on('$destroy', function() {
        files_input[0].files = void(0);
    });
};
```

## Execute event

```javascript
 gs.eventQueue(/* event name */ 'sample.event.name',
  /* parameter 1 */ current, 
  /* parameter 2 */ gs.getUserID(), 
  /* parameter 3 */ gs.getUserName());
```

## Execute job

```javascript
var job = new GlideRecord('sysauto_script');
if (job.get('name', /* job name */ 'Sample Job'))
  SncTriggerSynchronizer.executeNow(job);
```

## Convert GB to Bytes

B - Byte  
KB - Kilobyte  
MB - Megabyte  
GB - Gigabyte  
TB - TB  
PB - Petabyte

```javascript
(new StorageDataSize('2', 'GB')).getBytes()
```

## Convert Bytes to GB

```javascript
(new StorageDataSize(2147483648)).toString()
```

## Validate the string is sys_id

```javascript
if(GlideStringUtil.isEligibleSysID('4c9fc5b897a701506adcf5efe153aff3'))
  gs.print('Valid sys_id');
```

## Using g_form at UI Script

Example of using g_form in UI Script. The UI Script is used as JS Include in Theme.

```javascript
$(document).ready(function () {
  var $body = angular.element(document.body);
  var $rootScope = $body.scope().$root;

  $rootScope.$on('spModel.gForm.rendered', function (a, g_form) {
    var url = top.location.href;
    var searchParams = new URLSearchParams(url);
    searchParams.forEach(function(value, key) {
      if(key.startsWith('set.')) {
        key = key.replace('set.', '');
        g_form.setValue(key, value);
      }
    });
  });
});
```

## Modal window as a result of UI Action button

### UI Action

Options to be set: Active, Client, Form button
Onclick: loadApproveDialog()

```javascript
function loadApproveDialog() {
  var objSysId = typeof rowSysId == 'undefined' || rowSysId == null ? g_form.getUniqueValue() : rowSysId;
  var dialogClass = GlideModal ? GlideModal : GlideDialogWindow;
  var dialog = new dialogClass('ui_page_example');
  dialog.setTitle(new GwtMessage().getMessage('Acceptation'));dialog.setPreference('sysparm_obj_id', objSysId);
  dialog.setPreference('sysparm_table_name', g_form.getTableName());
  dialog.setPreference('sysparm_action', 'close');
  dialog.setPreference('focusTrap', true);
  dialog.render();
}
```

### UI Page

Name: ui_page_exmaple

HTML:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly
  trim="false"
  xmlns:j="jelly:core"
  xmlns:g="glide"
  xmlns:j2="null"
  xmlns:g2="null"
>
  <style>
    .modal-footer {
      margin-top: 10px;
    }
    
    .list-padding {
      padding-left:20px;
      margin-top:10px;
      margin-bottom:10px;
    }
  </style>
  <g:ui_form>
    <input type="hidden" id="sysparm_obj_id" name="sysparm_obj_id" value="$[HTML,JS:sysparm_obj_id]"/>
    <input type="hidden" id="sysparm_table_name" name="sysparm_table_name" value="$[HTML,JS:sysparm_table_name]"/>
    <input type="hidden" id="sysparm_action" name="sysparm_action" value="$[HTML,JS:sysparm_action]"/>
    <div class="form-group">
      <g:form_label>
        ${gs.getMessage("Comment")}
      </g:form_label>
      
      <textarea class="form-control" id="dialog_comment" name="dialog_comment" />
    </div>
    <footer class="modal-footer">
      <g:dialog_buttons_ok_cancel
        ok_text="${gs.getMessage('Save')}"
        ok_title="${gs.getMessage('Save')}"
        ok="return true"
        ok_style_class="btn btn-primary"
        cancel_text="${gs.getMessage('Cancel')}"
        cancel_title="${gs.getMessage('Cancel')}"
        cancel="return onCancel();"
      />
    </footer>
  </g:ui_form>
</j:jelly>
```

Client script:

```javascript
function onCancel() {
    GlideDialogWindow.get().destroy();
    return false;
}
```

Processing script:

```javascript
if (sysparm_obj_id && sysparm_table_name) {
    var taskGr = new GlideRecord(sysparm_table_name);
    if (taskGr.get(sysparm_obj_id)) {
        if (taskGr.getValue('sys_id') && taskGr.canWrite()) {
            if (sysparm_action == 'close') {
                taskGr.setValue('state', '3');
            }

            taskGr.setValue('description', dialog_comment);
            taskGr.work_notes = dialog_comment;
            taskGr.update();
        }
    }

    response.sendRedirect(sysparm_table_name + '.do?sys_id=' + sysparm_obj_id);
}
```

## Get sys_ids from selected records on list

Create UI Action with options.

Options to be set: Active, Show update, Client, List context menu, List choice
Onclick: executeListChoice()

```javascript
var executeListChoice = function() {
  var selSysIds = gel( /* Current UI Action sys_id */ '4633d83c9798d1506adcf5efe153afd9').getAttribute('gsft_allow');
  if (selSysIds === null || selSysIds === '') {
    selSysIds = g_list.getChecked();

    if (selSysIds === null || selSysIds === '')
      alert('No elements found');
    else
      alert(selSysIds);

    return;
  }
};
```

## Strip HTML string

Get text only from HTML

```javascript
GlideSPScriptable().stripHTML(html)
```

## Using not allowed function in scope application

Create Script Include in Global scope.

Options to be set:

* Accessible from: All application scopes
* Client callable: false

```javascript
var StripHTMLWrapper = Class.create();
StripHTMLWrapper.prototype = {
  initialize: function() {
  },
  strip: function(html) {
    return GlideSPScriptable().stripHTML(html);
  },

  type: 'StripHTMLWrapper'
};
```

Using:

```javascript
var wrapper = new StripHTMLWrapper();
wrapper.strip(/* some html code */ html);
```

## Get variable as HTML element by name in Client Script

```javascript
var field_id = g_form.resolveNameMap('variable_name');
var field = g_form.getElement(field_id);
```

## Get all editable fields in Client Script

```javascript
var fields = g_form.getEditableFields();
```

## Get number of fullfilers in application

Returning number of active licences by subscription

```javascript
var subscription_id = '822918621bbbe4107fa82f81f54bcb3a';

var applications = [];
var roles = [];
var users = [];

var applicationGr = new GlideRecord('license_has_family');
applicationGr.addQuery('license', subscription_id);
applicationGr.query();

while(applicationGr.next()) {
    applications.push(applicationGr.license_family.family_id.toString());
}

var licenseRoleGr = new GlideRecord('license_role');
licenseRoleGr.addQuery('application', 'IN', applications);
licenseRoleGr.addQuery('license_role_type', 'fulfiller');
licenseRoleGr.query();

while(licenseRoleGr.next()) {
    var roleGr = new GlideRecord('sys_user_role');
    if(roleGr.get('name', licenseRoleGr.getValue('name'))) {
        var roleItem = { 
            name: roleGr.getValue('name'), 
            id: roleGr.getUniqueValue(),
            users: []
        };

        var userRoleGr = new GlideRecord('sys_user_has_role');
        userRoleGr.addQuery('role', roleGr.getUniqueValue());
        userRoleGr.addQuery('user.active', 'true');
        userRoleGr.addActiveQuery();
        userRoleGr.query();

        while(userRoleGr.next()) {
            var user = { 
                id: userRoleGr.user.sys_id.toString(),
                name: userRoleGr.user.getDisplayValue(), 
                email: userRoleGr.user.email.getDisplayValue()
            };

            users.push(userRoleGr.user.sys_id.toString());
            roleItem.users.push(user);
        }

        roles.push(roleItem);
    } else {
        gs.print('No role ' + licenseRoleGr.getValue('name'));
    }
}

var unique = users.filter(function (value, index, array) { 
  return array.indexOf(value) === index;
});

gs.print("Unique users in application: " + unique.length);

//gs.print(JSON.stringify(roles));
```

## Change repository for application

1. Go to sys_repo_config.list
2. Delete record
3. Set new repo

## Move application between different companies instances

1. Export application as Update Set
2. Open xml file in editor
3. Replace all values in file from `application_scope` element (x_xxx_ is a company tag + application name)
4. In background script execute script:

``` javascript
gs.print(gs.generateGUID());
```

5. Replace all values in file from `application` element to value generated from 4.
6. Go to System Update Sets > Retrieved Update Sets
7. From Related Links click on Import Update Set from XML
8. Preview and Commit
