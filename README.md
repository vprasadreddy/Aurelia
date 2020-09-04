# test
```
function getUrlParameter(name) {
    name = name.replace(/[\[]/, '\\[').replace(/[\]]/, '\\]');
    var regex = new RegExp('[\\?&]' + name + '=([^&#]*)');
    var results = regex.exec(location.search);
    return results === null ? '' : decodeURIComponent(results[1].replace(/\+/g, ' '));
}
```

```
 <!--Calendar Events webpart to display all upcoming events from SharePoint Calendar using Rest API. -->
<html> 
 <head> 
 <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script> 
 <style> 

 #left{ 
 	margin: 0px; 
	padding:0px;
 	text-align:center !important; 
 } 
	
.currentmonthicon
{
width:50px; 
border-radius: 5px; 
background-color:#dfe4ea;
font-weight:bold;
color:#6ab04c;
}

.futuremonthicon
{
width:50px; 
border-radius: 5px; 
background-color:#dfe4ea;
font-weight:bold;
color: #1e90ff;
}

.details
{

 width:200px; 
 border-radius:5px; 
 background-color:#dfe4ea;
 }


#resultDiv table
{
border-spacing: 10px;
border-collapse: separate;
border-radius:10px;
margin-left: 20px;
} 

#currenteventmonth
{ 
 	background-color: #6ab04c; 
 	color: #fff; 
 	text-align:center !important; 
}

 #futureeventmonth{ 
 	background-color: #1e90ff; 
 	color: #fff; 
 	text-align:center !important; 
 }  

#resultDiv tr td p
 {
  	margin: 0px; 
	padding:0px;
 }
 
 

<!-- #6ab04c color for current event --> 	
 </style> 
</head> 
 <body> 
 <script type="text/javascript"> 
 $(document).ready(function(){ 
  getCalendarEvents(); 
 }); 
 
 var listName = "Calendar"; // Enter List display name here
 
 function getCalendarEvents() 
 	{ 
	
	var webAbsoluteUrl = _spPageContextInfo.webAbsoluteUrl;
	var webServerRelativeUrl = _spPageContextInfo.webServerRelativeUrl;
	var res = webAbsoluteUrl.replace(webServerRelativeUrl, "");
	var allEventsURL = webAbsoluteUrl + "/Lists/" + listName + "/AllItems.aspx"; // URL to Events Calendar AllItems.aspx view
	console.log(res);
	
 		//query the calendar list using REST API 
	 
 	 var todaysDate = new Date(); 
     var n = todaysDate.toISOString(); //yyyy-mm-ddT14:05:09.341Z  --> 2018-04-10T14:05:09.341Z
	 var todaysShortDate = new Date(todaysDate.getFullYear(), todaysDate.getMonth(), todaysDate.getDate());  //Year, Month, Date
	 console.log(n); 
	 console.log(todaysShortDate); // Day Month Date 00:00:00 EDT Year -->Tue Apr 10 00:00:00 EDT 2018 
	 var webUrl=_spPageContextInfo.webAbsoluteUrl + "/_api/web/lists/GetByTitle('"+ listName+ "')/GetItems";
	 
	 //the below query string fetches 4 calendar events where Event Date(Start Date) is greater than or equal to today's date.
	 var queryString = { "query" :
						{"__metadata": 
							{ "type": "SP.CamlQuery" 
							},
							"ViewXml": "<View><Query><Where><Geq><FieldRef Name='EventDate' /><Value Type='DateTime'><Today /></Value></Geq></Where><OrderBy><FieldRef Name='EventDate' /></OrderBy></Query><RowLimit>4</RowLimit></View>"
						}
					   };
	 console.log(webUrl);
 	 $.ajax({ 
          //order by date and set the filter to get only events greater then or equal then todays date(future events) and top limits the no of items to display. 
        url: webUrl, 
 		type: "POST",  
		data: JSON.stringify(queryString),
        headers: 
		{ 
		"X-RequestDigest": $("#__REQUESTDIGEST").val(),
		"Accept": "application/json; odata=verbose",
		"Content-Type": "application/json;odata=verbose"
	    }, 
        success: function (data) { 
			if (data.d.results.length>0)
			{
				console.log(data);
				for(i=0;i<data.d.results.length;i++) 
				{ 
			//  var fileRef = data.d.results[i].FileRef;
			//	fileRef = fileRef.replace(/[^/]*$/gi, "");  //regular expression to remove text after last '/' (forward slash) from fileRef URL
			//	var allEventsURL = res + fileRef + "AllItems.aspx"; // URL to Events Calendar AllItems.aspx view
			//	console.log(fileRef);
			//	console.log(allEventsURL);
				var calendarIcon = "";
 				var eventLoc=data.d.results[i].Location!=null ? data.d.results[i].Location: "" ; 
				
				//remove <div></div> & <p></p> tags from the Description column(Multiple line text field).
 				var eventDes=data.d.results[i].Description!=null ? data.d.results[i].Description.replace(/<div>|<\/div>|<p>|<\/p>/gi,"").slice(0, 35): "";
				var eventTitle=data.d.results[i].Title!=null ? data.d.results[i].Title: "";
				var itemId=data.d.results[i].Id;
				var eventURL = encodeURIComponent(webAbsoluteUrl + "/Lists/" + listName+ "/DispForm.aspx?ID=" + itemId);
				console.log(eventURL);
				var eventDate=new Date(data.d.results[i].EventDate);//dd-mm-yyyyT00:00:00Z format 
				var eventShortDate = new Date(eventDate.getFullYear(), eventDate.getMonth(), eventDate.getDate());  //Year, Month, Date
				console.log(eventDate);
				console.log(eventShortDate);
 				var eventDay=eventDate.toDateString().slice(0, 3); // ddd mmmm dt yyyy ex: "Tue Apr 10 2018"
 				var eventMonth=eventDate.toDateString().slice(4, 7); 
 				var eventDayNum=eventDate.toDateString().slice(8, 10); 
				var eventYear=eventDate.toDateString().slice(11, 15);
				var eventTime=eventDate.toLocaleTimeString();
				console.log(eventTime);
				
				//dynamic html tag  for each event. 

				if (eventShortDate > todaysShortDate){
				calendarIcon= calendarIcon + "<tr><td class='futuremonthicon'><p id='futureeventmonth'>"+eventMonth+"</p><p id='left'>"+eventDayNum+"</p><p id='left'>"+eventDay+"</p></td><td class='details'><p><b>Agenda:</b>"+eventTitle+"</p><p>" + eventDes + "..." + "</p><p><b>Location:</b>"+eventLoc+"</p><p><b>Event Time:</b>"+eventTime+"</p></td></tr>"; 
				calendarIcon= calendarIcon + "<tr>" + "<td></td>" + "<td>" + "<a href=javascript:ModalDailog('"+ eventURL + "')>View event details</a>" +"</td></tr>";
				calendarIcon = "<table>" + calendarIcon + "</table>";
				$("#resultDiv").append(calendarIcon);
				}
				else{
				calendarIcon= calendarIcon + "<tr><td class='currentmonthicon'><p id='currenteventmonth'>"+eventMonth+"</p><p id='left'>"+eventDayNum+"</p><p id='left'>"+eventDay+"</p></td><td class='details'><p><b>Agenda:</b>"+eventTitle+"</p><p>" + eventDes + "..." + "</p><p><b>Location:</b>"+eventLoc+"</p><p><b>Event Time:</b>"+eventTime+"</p></td></tr>"; 
				calendarIcon= calendarIcon + "<tr>" + "<td></td>" + "<td>" + "<a href=javascript:ModalDailog('"+ eventURL + "')>View event details</a>" +"</td></tr>";
				calendarIcon = "<table>" + calendarIcon + "</table>";
				$("#resultDiv").append(calendarIcon);
				}
			}
			$("#resultDiv").append("<table><tr><td><b><a href='"+ allEventsURL + "' target="+ '_blank' +">" + "View All Events" + "</a></b></td></tr></table>");
}
else
{
$("#resultDiv").append("<b><i>No Upcoming Events</i></b>" + "<br><br>" + "<a href='"+ allEventsURL + "' target="+ '_blank' +">" + "View All Previous Events" + "</a>");
}
            			
         }, 
         error: function (data) { 
             
         } 
     }); 
 	} 
	
	 //Function to open url in Modal Dailog
 function ModalDailog(urlvalue)
 {    
     var options = {
         url: urlvalue,            
         allowMaximize: true,
         showClose: true,     
         dialogReturnValueCallback: silentCallback
     };
     SP.UI.ModalDialog.showModalDialog(options);
 }
 function silentCallback(dialogResult, returnValue)
 {
 }	
 	 
</script> 
 <div id ="resultDiv"> 
 <!-- dynamic tr and td tag for each event. -->
 </div> 
 </body> 
 </html> 
```

