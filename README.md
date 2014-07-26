# SPPromise

SPPromise is a wrapper library over the [SharePoint 2010 JSOM](http://msdn.microsoft.com/fr-fr/library/hh372944(v=office.14).aspx) APIs, packaged as a jQuery plugin.
The name comes from the jQuery [Promise](http://api.jquery.com/promise/) objects, which are extensively used to streamline the usage of the underlying aynchronous APIs.

### Why

If you've ever used the SharePoint javascript client object model, you know just how much verbose the code you have to write is, mostly due to the asynchronous nature of the API (the infamous [executeQueryAsync](http://msdn.microsoft.com/en-us/library/office/ff411085(v=office.14).aspx) method being the major culprit).

Let's see how the code you need to write to retrieve all items from a list looks like by using the original APIs vs SPPromise.

**With the original JSOM:**
```js
function retrieveListItems() {

    var clientContext = new SP.ClientContext();
    var oList = clientContext.get_web().get_lists().getByTitle('Announcements');
            
    this.collListItem = oList.getItems();        
    clientContext.load(collListItem);    
    
    clientContext.executeQueryAsync(
		Function.createDelegate(this, this.onQuerySucceeded), 
		Function.createDelegate(this, this.onQueryFailed)
	);              
}

function onQuerySucceeded(sender, args) {    
    var listItemEnumerator = collListItem.getEnumerator();
        
    while (listItemEnumerator.moveNext()) {
        var oListItem = listItemEnumerator.get_current();
        console.log('ID: ' + oListItem.get_id() );
        console.log('Title: ' + oListItem.get_item('Title') );
        console.log('Body: ' + oListItem.get_item('Body') );        
    }
}

function onQueryFailed(sender, args) {
    console.log('Request failed. ' + args.get_message() + '\n' + args.get_stackTrace());
}

retrieveListItems();
```

**With SPPromise:**
```js
var spp = $().SPPromise; //store a reference just as a shortcut
spp.ListItems.getItems('Announcements')
    .then(writeToConsole)
    .fail(spp.printError);

function writeToConsole(announcements){
	for (var i = 0; i < announcements.length; i++){
		console.log('ID: ' + announcements[i].ID);
		console.log('Title: ' + announcements[i].Title);
		console.log('Body: ' + announcements[i].Body);
	}	
}
```

### Some key features:
* Nice and clean syntax, thanks to jQuery Deferred/Promise api
* Returned objects are simple javascript objects and arrays
* Access item properties with the standard dot notation and loop through collections with simple for loops  (no need to call get_item() and getEnumerator all around)
* Lists are retrieved by internal name by default (configurable. See the [API documentation](#api-documentation)).

### Get started
#### Installation
**Requirements:** SPService requires the [jQuery library](jquery.com).

In order to use the library, you must add a reference to it into a single page, a page layout or a master page, as you see fit.

The recommened way to include jQuery or other libraries used throughout a SharePoint site/site collection is to store them either in the _layouts directory or in the site collection Site Assets library.

##### Example: Adding SPPromise to a masterpage using a ScriptLink

```xml
<SharePoint:ScriptLink id="jquery" runat="server" Name="~sitecollection/SiteAssets/js/jquery.min.js" Language="javascript"/>
<SharePoint:ScriptLink id="sppromise" runat="server" Name="~sitecollection/SiteAssets/js/jquery.SPpromise.js" Language="javascript"/>
```

#### Using SPPromise
The library depends on the **SP.js** file, which it waits to be loaded before initializing itself. Once initalized, SPPromise notifies being ready by calling _notifyScriptLoadedAndExecuteWaitingJobs('SPPromise')_.

Code that depends on SPPromise must therefore wait for it to be ready. You can do this by calling 

```javascript
ExecuteOrDelayUntilScriptLoaded(sppReady, 'SPPromise');
function sppReady() { 
  // you can now use SPPromise
}
```


### API documentation
The unminified version of the library comes with full XML comments documentation, to support Visual Studio intellisense.

Here is a list of all currently available functions and usage examples.

### Supported SharePoint versions
The library was written and tested on SharePoint 2010.

Extending it to fully support the 2013 version shouldn't be a hard task and I'm considering doing it in a near future (code contributions are welcome!).
