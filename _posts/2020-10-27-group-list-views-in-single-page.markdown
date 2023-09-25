---
layout: single
title:  "Group list views in single page"
date:   2020-10-27 10:18:02 +0100
categories: javascript
excerpt: "When configuring forms and lists, there is the option to configure all records from a single view (Configure -> All). These scripts build on that functionality to allow for custom configurations to be grouped together (for example, script includes, rest messages for an integration). Reduces the number of modules in the application menu."
---

When configuring forms and lists, there is the option to configure all records from a single view (Configure -> All). These scripts build on that functionality to allow for custom configurations to be grouped together (for example, script includes, rest messages for an integration). Reduces the number of modules in the application menu.

![Grouped tabs](/assets/grouped_config.jpg)

### Overview
The grouped config is made up of the following elements;

- grouped_config.xml - UI Page which builds the tabbed list
- grouped_config_list.xml - UI Macro for displaying the tab
- System property defining each config group
- Module for launching the grouped config

### UI Page
This is modelled on the personalize_all.xml page provided out of the box.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
	<script>
		addLoadEvent( function() {
		var tabs = new GlideTabs2("tabs2_list", gel("rules_span"), 0, '');
		tabs.activate();
		show("rules_span");
		});
	</script>

	<g:evaluate jelly="true">

		var configJSON = gs.getProperty(jelly.sysparm_config_name);
		var json = JSON.parse(configJSON);
		var properties = json['properties'];

		var property = new GlideRecord('sys_properties');
		property.get('name', jelly.sysparm_config_name);
		var property_sysid = property.getUniqueValue();

	</g:evaluate>

	<g:inline template="tabs2.xml" />

	<TABLE id="testtable" width="100%" cellSpacing="0" cellPadding="0" border="0">
		<TR>
			<TD valign="top">
				<j:set var="jvar_list_type" value="cardboard" />

				<div style="font-size:1.3em;margin-bottom:5px;">
					${jvar_board_type} ${gs.getMessage("Configurations for")}
					<span style="font-weight:bold;">
						$[SP]${json.title}
					</span>
				</div>

				<ul style="font-size:1.3em">
					<j:if test="${properties != null}">
						<li><a href="system_properties_ui.do?sysparm_title=${properties.title}&amp;sysparm_category=${properties.category}">${json.title} properties</a></li>
					</j:if>
					<li><a href="/sys_properties.do?sys_id=${property_sysid}">Edit configuration page settings</a></li>
				</ul>

				<!-- tabstrip for related lists -->
				<div class="tabs2_strip" id="tabs2_list">
					<!-- hack for IE bug when tab strip causes a horizontal scroll bar to appear -->
					<img src="images/s.gifx" alt="" height="1px" width="1px" style="margin-right: 0px;" />
				</div>

				<span id="rules_span" style="display:none;border-top: 1px solid black; padding-top: 5px; margin-top: -7px;">
					<j:if test="${!RP.isPortal()}">
						<j:set var="jvar_use_name_for_list_id" value="true" />
					</j:if>

					<j:forEach var = "jvar_get" items = "${json.tabs}">
						<g:call function="grouped_config_list.xml" table="${jvar_get.table}" source="${jvar_get.query}" />
					</j:forEach>    

					<g2:client_script type="user" />
				</span>
			</TD>
		</TR>
	</TABLE>

</j:jelly>
{% endhighlight %}

### UI Macro - grouped_config_list.xml
{% highlight xml %}
<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">

	<g:evaluate var="jvar_table_exists" jelly="true">
		var grTable = new GlideRecord(jelly.jvar_table);
		grTable.isValid();
	</g:evaluate>

	<j:if test="${jvar_table_exists}">

		<g:list_default table="${jvar_table}" view="" />

		<!-- Set up ListProperties for the list -->
		<g:evaluate jelly="true">
			ListProperties.setHasTopNav(true);
			ListProperties.setHasTitle(true);
			ListProperties.setHasTitleContextMenu(true);
			ListProperties.setHasFilter(true);
			ListProperties.setHasBreadcrumbs(true);
			ListProperties.setHasSearch(true);
			ListProperties.setHasTopVCR(true);
			ListProperties.setHasHeader(true);
			ListProperties.setHasHeaderContextMenu(true);
			ListProperties.setHasListMechanic(true);
			ListProperties.setHasRowContextMenu(true);
			ListProperties.setShowLinks(true);
			ListProperties.setHasPopup(true);
			ListProperties.setShowEmpty(true);
			ListProperties.setHasBottomNav(true);
			ListProperties.setHasActions(true);
			ListProperties.setHasBottomVCR(true);
			ListProperties.setCanChangeView(true);
			ListProperties.setCanGroup(true);

		</g:evaluate>

		<j2:set var="sysparm_query" value="${jvar_source}" />
		<!-- register for list printing -->
		<j:set var="jvar_handle_list_printing" value="true" />

		<!-- and build the list output -->
		<g:inline template="list2_default.xml" />

	</j:if>

</j:jelly>
{% endhighlight %}

### Configuration
The config for the tabs is stored in a system property as a JSON object in the following format.


{% highlight json %}
{
  "title" : "title for the integration",
  "tabs": [
    { "table": "table_name", "query" : "active=true" },
    { "table": "table_name", "query" : "active=true" }
  ],
  "properties": {
    "title": "property page title",
    "category": "property page category"
  }
}

{% endhighlight %}

### Application Module
Finally, to actually open the page, a URL (with parameters) module needs to be created with the following link;

```grouped_config.do?sysparm_config_name=[sys property name]```
