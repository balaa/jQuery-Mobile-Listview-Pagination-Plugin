# test edit  jQuery Mobile Listview Pagination Plugin (aptly named  Listomatic)

## Requirements

Works on jQuery Mobile 1.3.0 (grab the latest copy at http://jquerymobile.com), though might work with older versions - never tested with old versions

## How To Use

First download the Listomatic plugin (jquery.mobile.listomatic.js)

Include the Listomatic plugin after the jquery mobile library: 

`<script type="text/javascript" src="<Path To This File>/jquery.mobile.listomatic.js"></script>`

Add the "data-listomatic" attribute to your ul list with a value of "true". You can also add the search box by adding the "data-filter" attribute with a value of "true".

`<ul id="listview" data-role="listview" data-filter="true" data-listomatic="true"></ul>`

The Listomatic plugin will take care of the rest, including setting sensible defaults (ie, 10 records per page, remembering state (ie, list offsets) and caching where appropriate for a great user experience.

# Register an Ajax Function

Register an Ajax function that will be called every time the "Show More" button is clicked or tapped on or when a "Search" is performed.

<pre>
$(document).on("pageinit", function(){
	var serviceURL = "http://www.your-domain.com/returns-json-numbers.php";	
	var getNumber = function() {
		return $.ajax({
			type: "post",
			beforeSend: function() { $.mobile.loading( 'show' ) }, //Show spinner
			complete: function() { $.mobile.loading( 'hide' ) }, //Hide spinner
			async: "true",
			dataType: 'json',
			url: serviceURL,
			data: { listomatic: $.mobile.listomatic.prototype.getResults() },
			success: function(data) {
				if (data && data.numbers) {
					var list = '';
					$.each(data.numbers, function(index, value) {
						list += '&lt;li data-icon="false" data-filtertext="' + value.date + '">' +
								'&lt;div class="latestNumber">' +
								'&lt;span class="circle">' + value.number[1] + '&lt;/span>' +
								'&lt;span class="circle">' + value.number[2] + '&lt;/span>' +
								'&lt;span class="circle">' + value.number[3] + '&lt;/span>' +
								'&lt;span class="circle">' + value.number[4] + '&lt;/span>' +
								'&lt;span class="circle">' + value.number[5] + '&lt;/span>' +
								'&lt;span class="circle powerball">' + value.number[6] + '&lt;/span>' +
								'&lt;/div>' +
								'&lt;p style="text-align:center;" class="latestNumberDate">' + value.date + '&lt;/p>' +
								'&lt;/li>';
					}); // end each	
					$('#listview').append(list).listview("refresh");
				} // end if
			} // end success
		}); // end ajax
	}
	$.extend($.mobile.listomatic.prototype.options, {perPage: 2, btnLabel: 'Show Me More', refreshContent: 'daily'});
	$.mobile.listomatic.prototype.registerAjaxCall(getNumber);
});
</pre>



#Server Side Configuration To Enable Pagination

For each Ajax call there will be several paramaters that will need to be hooked on to the sql query on the server side to allow for pagination by only getting a subset of records, one at a time. The following example is written in PHP using MySQL:

<pre>
error_reporting(0);
header('Access-Control-Allow-Origin: *');
$con=mysqli_connect("domain","username","password","database");
if (mysqli_connect_errno()) {
	echo "Failed to connect to MySQL: " . mysqli_connect_error();
}
$perPage    = $_REQUEST['listomatic']['perPage'];
$listOffset = $_REQUEST['listomatic']['listOffset'];
$searchTerm = $_REQUEST['listomatic']['searchTerm'];

if ($searchTerm) {
	$sql = "SELECT SQL_CALC_FOUND_ROWS *
			FROM numbers
			WHERE date LIKE '%$searchTerm%'
			ORDER BY date DESC
			LIMIT $listOffset, $perPage";
} else {
	$sql = "SELECT SQL_CALC_FOUND_ROWS *
			FROM numbers
			ORDER BY date DESC
			LIMIT $listOffset, $perPage";
}
$result = mysqli_query($con, $sql);

// If you are using MySQL use SQL_CALC_FOUND_ROWS in your main queries (above)
// Now to get the total records available use the FOUND_ROWS() function (below)
$resultNumRows = mysqli_query($con, 'SELECT FOUND_ROWS() as foundRows');
$rowFoundRows = mysqli_fetch_array($resultNumRows);
$iFoundRows = $rowFoundRows['foundRows'];

while($row = mysqli_fetch_array($result)) {
	$sDate = date('m/d/Y', strtotime($row['date']));
	// Listomatic requires the "total" field to show/hide the "Show More" button
	$aData['total'] = $iFoundRows; 
	// The following is sample data (in this case Powerball numbers) that you want to display
	$aData['numbers'][] = array('date' =>  $sDate,
	    						'number' => array(1 => $row['num1'],
													2 => $row['num2'],
													3 => $row['num3'],
													4 => $row['num4'],
													5 => $row['num5'],
													6 => $row['num6']));
}
echo json_encode($aData);
exit;
</pre>

#Response Data in JSON Format
Listomatic requires a data response in JSON format. The "total" field is required by the plugin to show/hide the "Show More" button. The "Show More" button will be showned if the current records displayed does not exceed the total records available from the server. Otherwise the "Show More" button will be hidden when no additional records are available to be displayed.

The numbers field as shown is the actual data to be put into the list. Pass what ever data you need to be displayed and modify your Ajax function to accept this data.
<pre>
{
   "total":"1599",
   "numbers":[
      {
         "date":"02\/16\/2013",
         "number":{
            "1":"58",
            "2":"16",
            "3":"15",
            "4":"46",
            "5":"50",
            "6":"29"
         }
      },
      {
         "date":"02\/13\/2013",
         "number":{
            "1":"12",
            "2":"43",
            "3":"27",
            "4":"25",
            "5":"23",
            "6":"29"
         }
      }
   ]
}
</pre>

#Configuration

The Listomatic plugin will set some default settings that can be overwritten, such as number of records to return per call (perPage), label text for the "Show More" button (btnLabel) and an option to refresh content (refreshContent), if set to 'false' the page will not refresh, if set to 'true' the page will refresh at midnight. 

<pre>
$.extend($.mobile.listomatic.prototype.options, {perPage: 2, 
													btnLabel: 'Show Me More', 
													refreshContent: 'daily',
													noResultsFound: 'No Results Found'
													});
</pre>

# License

Author & copyright (c) 2013: [Stakbit](http://www.stakbit.com)

Released under the MIT license.
