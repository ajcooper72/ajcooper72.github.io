---
layout: single
title:  "Watch for changes to Multi-Row Variable Sets"
date:   2023-11-28 15:00:02 +0000
categories: service_catalog javascript
excerpt: "An onChange equivalent for Multi-Row Variable Sets"
---
## Introduction
I've created a couple of UI Scripts that can be used on the desktop and portal service catalogs to enable something similar to the onChange event available for other variables on the catalog item.

The code has gone through a few iterations including decoding mutation records, parsing the HTML of the MRVS table to finally just using the g_form.getValue() method and checking for changes to the rows.  It's been a useful exercise as it's ended up far simpler than when it started.  Ultimately, this will be a single library for both platforms rather than two.

Essentially, it's using the MutationObserver API to watch for changes to the DOM (specifically the MRVS table).  It then queries the current MRVS and compares against the previous version, resulting in a JSON object which has either added, removed, updated rows.

i.e.

```json
{
   "added": {
      "vm_number": "1",
      "name": "Test Entry",
      "owner": "<sys_id>"
   }
}
```

I've posted the code to ServiceNow Share as well as the Github repository;

- [ServiceNow Share](https://developer.servicenow.com/connect.do#!/share/contents/4185574_multirow_variable_set_watcher?v=2.0&t=PRODUCT_DETAILS)
- [Github Repository](https://github.com/ajcooper72/MRVSUtil)

## Example

```javascript
function onLoad() {
	
    setTimeout(function() {
        var mrvsName = ''name of multi-row-variable-set'';
        var mrvs = (g_form.getControl(mrvsName).id) ?
            new MRVSWatcherUtil(mrvsName) : new MRVSWatcherUtilSP(mrvsName, this.window, g_form);

        var observer = new MutationObserver(function(mutationList, observer) {
            var modifiedData = mrvs.getRowUpdates(false);
            if (modifiedData) {
                 // do something with the data
                 console.log(JSON.stringify(modifiedData, '', 3));
            }
        });

        // create the observer looking for changes to the contents of the MRVS
        observer.observe(mrvs.getTableID(), mrvs.OBSERVER_CONFIG);

    }, 1000);

}
```

