---
layout: post
title:  "Supplemental Episode – Getting just group counts for calls to SharePoint 2013 REST API"
date:   2017-05-16 10:00:00 -0400
categories: search search_refiners sharepoint sp.requestexecutor sp2013 search_api
---
{% include youtubePlayer.html id="OAZptDTu3cM" %}
Every so often, your cloud talk guys get caught up in a small mystery that may be related to our cloud topics and in this case because SharePoint happens to be on the cloud using the same Search API via REST that this solution handles.  This post will include the notes that will help make it useful for those trying to solve a similar problem and perhaps give us something to look back on when the related cloud tech matures (or stop’s crying, at least …)
On that note fighting with the formatting issues of word press we have the following that assumes and you, the reader, are actually using this.
You will need to have developer tools working properly on the browser you choose.  Note that we used Chrome and Edge for this and they reported the same issues as we went along.
Here are the key points:
- Search Rest API URL to call considerations:
  - rowlimit=0 – to make sure you do not also bring back results, set this to zero
  - refiners=’group1, group2, groupN’ – after setting up your refiners in SharePoint AHEAD of time (https://blogs.technet.microsoft.com/tothesharepoint/2013/11/10/how-to-add-refiners-to-your-search-results-page-in-sharepoint-2013/) put then in a comma separated list correctly
- How do you know it works?
  - Just using the URL  you should have a node called:
  - that has content instead of !!
  - If you have refiners set up properly, you will have a breakdown in child node specifically in and then
    - RefinementCount (this is the number!!)
    - RefinementName
    - RefinmentToken
    - RefinmentValue (may be the same as the name)
  - There will also be no node called RelevantResults anywhere!!
- To do code to get it, you need SP.RequestExecutor to work on SharePoint 2013 – https://msdn.microsoft.com/en-us/library/office/dn735924.aspx
- To use code to get it from inside of SP2013 (either script link or a script editor web part) the following major considerations
  - Need to bring in JQuery (see code snippet below)
  - Need to manually bring in SP.RequestExecutor
§ Troubleshooting the need to use a script tag was based on using the Dev Tools on the browser and seeing that $getScript fails as per the link above and also because it never would find SP.RequestExecutor on its own (object not found for both!)
  - Make sure the value for your URL used for the SP.RequestExecutor instantiation is just the root of the site (http:// and does NOT include the query!!
- Notes on the Code:
  - Key code piece that is still on the snippet below is the if statement checking if refinements.length == 0 … if the root node to that is null they this will be object not found anyway and WILL fail if your refinement setting in the URL is not good or valid.  There may be a legitimate “zero returned” condition but this would only happen if all things are correct.  This should probably be more bullet proof with a check to precede it. Where allowed by WordPress, It is highlighted below.
The following code snippet works within the script editor.  Replace all items marked [insertValueHere] with those relevant to your search. Important items in code to watch out for (you may get errors because you have malformed these items such as putting in too many “single quotes”) are highlighted also where Word Press allowed it.  Please clean up code you copy paste from here due to the wordpress code display issues for example comments are just plain text coming after the pseudo tag “COMMENT”.
COMMENT: a place to see messages aka response.write! And a place to click.  Please note that you if you use a button, it will fail because there is already a default behavior assigned to the button even which “resets” or refreshes the page.  You could override this but what is the point of all that code when a clickable DIV will do, right?  No?  Ok what do you want 🙂
[div tag goes here with id called messages and onclick to reveal() and style to show the pointer]Press here For Script Activity[end tag]
COMMENT: results here is copy paste from the blog posts

[div tag here with id related-content-results which is an empty div]

COMMENT JQUERY reference (roll your own if you wish) .. Make sure it is HTTPS!
[script tag
Src -> code.jquery.com/jquery-3.2.1.slim.min.js”] [close script tag]
COMMENT: This is the key reference to the request executor JS file because $.getScript did not work!
[script tag src -> “http://[root site name]/[site root]/_l a y o u t s / 1 5 / SP.RequestExecutor.js”>[ close script tag]
COMMENT: work starts here …
[main code script tag here]
 
 Javascript |   copy code |?

1	
function reveal(){<br />$("#messages").text("Script activity Started"); //Response.write because, why not
searchPOC1();

}
function searchPOC1(){
var queryText = “‘ContentType:[YOUR CONTENT TYPE]’&rowlimit=0&trimduplicates=false&selectproperties=’Title,[put all your other properties you are selecting here]’&refiners='[your refiners comma separated]'”;
var appWebURL = _spPageContextInfo.webAbsoluteUrl ;

var executor = new SP.RequestExecutor(appWebURL);
$(“#messages”).text(“executor ready!”);
executor.executeAsync({
url: appWebURL + “/_api/search/query?querytext=” + queryText,
method: “GET”,
headers: { “Accept”: “application/json; odata=verbose” },
success: successHandler,
error: errorHandler
});
}
function successHandler(data) {
var jsonObj = JSON.parse(data.body);
var refinements = jsonObj.d.query.PrimaryQueryResult.RefinementResults.Refiners.results[0].Entries.results;
if (refinements.length == 0) {
$(‘#related-content-results’).text(‘No related documents were found’); //assumes refinement setting in url is valid!! See notes.
}
else
{
var searchRefinementHtml = ”;
$.each(refinements, function (index, refinement) {
searchRefinementHtml += ” ” + refinement.RefinementValue + ” (” + refinement.RefinementCount + “)” + “”;
});
$(‘#related-content-results’).html(searchRefinementHtml);
}
$(“#messages”).text(“executor succeeded!”);
}
function errorHandler() {
$(“#messages”).text(“executor failed!!!!”);
}

[end main code script tag here]

Please note that this is for 2013 and for the SP Framework, you would use standard REST call methods since you no longer need all the extra pieces provided by SP.RequestExecutor because you are “IN CONTEXT when running that way as opposed to in an IFRAME.

Other supporting links:
- The original overview: https://dev.office.com/sharepoint/docs/general-development/sharepoint-search-rest-api-overview
- The original call to use SP.RequestExecutor: https://dev.office.com/sharepoint/docs/sp-add-ins/access-sharepoint-data-from-add-ins-using-the-cross-domain-library
- The Chris O. Snippet that DOES NOT use $.getScript but presumably works … or assumes pre-loading the JS file: http://www.sharepointnutsandbolts.com/2013/01/calling-sharepoint-search-using-rest-eg.html
- The Suresh C blog post enhancing Chris O. post with the refiner look (assumes refiner set up!) – http://sharepoint.sureshc.com/2014/12/how-to-get-search-refiners-using-rest-api.html
- A SP.RequestExecutor post that underscores that you need the full path for the JS file online (but appears to also be the case locally!)       https://social.technet.microsoft.com/Forums/exchange/en-US/245696bc-1f75-4367-b3cb-08325524299c/sprequestexecutorjs-not-found-in-sharepoint-online?forum=sharepointdevelopment