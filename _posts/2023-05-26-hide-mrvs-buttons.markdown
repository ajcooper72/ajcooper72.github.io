---
layout: post
title:  "Hide Multi-row Variable Set Buttons"
date:   2023-05-26 10:18:02 +0100
categories: service_catalog css
excerpt: "How to hide MRVS buttons on platform and portal"
---

Sometimes you need to control the buttons available on a multi-row variable set for catalog items.  Setting the variable set as readonly will do it, but these stylesheet changes will let you be a bit more granular.

In order to use the styles, create a new **Rich Text Label** and paste the stylesheet code into it.  Obviously remove the buttons you don't want to hide.

{% highlight css %}
<style>
	/* hide MRVS buttons on portal */
	sp-sc-multi-row-element .btn.btn-primary.m-r,   /* hide the add button */
	sp-sc-multi-row-element .btn.btn-default,		/* hide the add button */
	sp-sc-multi-row-element .fa-pencil,				/* hide the edit buttons */
	sp-sc-multi-row-element .fa-close,				/* hide the delete row buttons */
	sp-sc-multi-row-element .table > tbody > tr > td:first-child,	/* hide the first column cell */
	sp-sc-multi-row-element .table > caption + thead > tr:first-child > th:first-child	/* hide the first column header */
	{
  		display:none;
	}

  /* hide MRVS buttons on platform */
	.sc-table-variable-buttons .btn.btn-primary, 	/* hide the add button */
  	.sc-table-variable-buttons .btn.btn-default, 	/* hide the add button */
  	.sc-multi-row-actions .icon-edit,				/* hide the edit buttons */
  	.sc-multi-row-actions .icon-cross,				/* hide the delete row buttons */
  	.sc-table-variable-header:first-child,			/* hide the first column header */
  	.sc-multi-row-actions							/* hide the first column cell */
  	{
	    display: none;
	}

</style>
{% endhighlight %}