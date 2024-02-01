---
layout: single
title:  "Writing Ajax Script Includes to work with ATF"
date:   2020-07-21 10:18:02 +0100
categories: ajax javascript atf
excerpt: "Building Ajax script includes that be tested outside of GlideAjax calls."
---

## AJAX Script Include
As AJAX methods retrieve parameters using *getParameter* its best to split the actually functionality into another method which receives the parameters in the standard way. This is the approach that ServiceNow take in most of their AJAX libraries.

Below is a simple example where a sys_id of a user is passed in and a JSON object returned containing email address and phone number.

```javascript
var AjaxATFDemo = Class.create();
AjaxATFDemo.prototype = Object.extendsObject(AbstractAjaxProcessor, {

	getUserDetails_Ajax: function() {
		var sys_id = this.getParameter('sysparm_usersysid');
		return this._getUserDetails(sys_id);
	},

	/**SNDOC
	@name getUserDetails
	@description Return basic user details for provided sys_id
	@param {string} [sys_id] - description
	@returns {object} JSON object containing user details
	*/
	_getUserDetails: function(sys_id){
		var user_obj = {
			'email_address' : '',
			'phone_number' : '',		
		};
		
		var grUser = new GlideRecord('sys_user');
		if (grUser.get(sys_id)) {
			user_obj['email_address'] = grUser.email.toString();
			user_obj['phone_number'] = grUser.phone.toString();
		} 
		
		return JSON.stringify(user_obj);
	},

	type: 'AjaxATFDemo'
});
```

## ATF Server Side Script
The server side script test step can now call the non-AJAX method with a sys_id without needing to simulate a GlideAjax call in the client. Make sure there are some steps included to create the test user and set a phone and email address. This code is from the New York release so the test step returns a 'user' value, prior to New York the create record step will return a record_id. Use 'record_id' for instances before New York.

```javascript
(function(outputs, steps, stepResult, assertEqual) {

	var user_sysid = steps('sys_id_of_step_creating_user').user;  // use record_id instead of user for pre-New York instances
	var grUser = new GlideRecord('sys_user');
	if (grUser.get(user_sysid)) {

		var userJSON = new AjaxATFDemo().getUserDetails(grUser.getUniqueValue());
		userJSON = JSON.parse(userJSON);
		gs.log("Phone (sys_user) " + grUser.phone + " (json) " + userJSON['phone_number'], "ATF-LOG");
		gs.log("Email (sys_user) " + grUser.email + " (json) " + userJSON['email_address'], "ATF-LOG");

		if (userJSON['phone_number'] != grUser.phone || 
			userJSON['email_address'] != grUser.email) {
			stepResult.setOutputMessage("Details return from method do not match sys_user record");
			return false;	
		} else {
			stepResult.setOutputMessage("User details match");
			return true;	
		}

	} else {
		stepResult.setOutputMessage("Failed to find user record");
		return false;
	}

})(outputs, steps, stepResult, assertEqual);
```