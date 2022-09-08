# Incremental counter in Filenet

## Artifact needed:

- 1 Custom object class : Counter 
    4 properties : 
        CounterName : The Counter class name of your target object "DAC_Archive".   
        CounterProperty: The properties name where to store the value in the class.
        CounterPrefix: if the property is a string, we can add a static prefix. 
        Count: the current count.

- 1 event actions that will perform the incremental update.

- 1 folder:
    config folder is the folder where to store and retrieve all the counters object.
    
- X number of counter object per document or folder to increments. 


## Setup 

### Create properties template & custom class 

1. Create properties template in acce :

1.1 Browse to property templates folder and click new 

![](capture-d-e-cran-2022-09-08-a--13-28-41-l7syvuln.png)

1.2 Commplete the name for the properties 

![](capture-d-e-cran-2022-09-08-a--12-14-17-l7syxvn6.png)

1.3 Select the data type String 

![](capture-d-e-cran-2022-09-08-a--12-14-24-l7syyx4q.png)


1.4 Click next

![](capture-d-e-cran-2022-09-08-a--13-22-19-l7sysav8.png)

**** 
 
 
1.5 Only for CounterName select set other attributes: 

![](capture-d-e-cran-2022-09-08-a--12-14-33-l7sz1l80.png)

1.6 Click next 

![](capture-d-e-cran-2022-09-08-a--13-23-23-l7szanef.png)
****

1.7 Click Finish

![](capture-d-e-cran-2022-09-08-a--13-23-30-l7szazb5.png)


2 Follow the same procedure of the remaining propreties:

- CounterProperty is string property
- CounterPrefix is string property
- Count is Integer property
   ![](image-l7syrj9n.png)

3. Create Custom Class

3.1 Locate Custom Class, click actions and new class

![](capture-d-e-cran-2022-09-08-a--13-47-40-l7szm1i6.png)

3.2 Complete the name, click next on the remaining screen, finish and open

![](capture-d-e-cran-2022-09-08-a--13-50-44-l7szo0sw.png)

![](capture-d-e-cran-2022-09-08-a--13-50-54-l7szo3po.png)

![](capture-d-e-cran-2022-09-08-a--13-51-00-l7szobxq.png)

3.3 Link properties to the custom class and save it.

![](capture-d-e-cran-2022-09-08-a--13-52-54-l7sztd0n.png)

![](capture-d-e-cran-2022-09-08-a--13-53-02-l7sztgjc.png)

![](capture-d-e-cran-2022-09-08-a--13-53-20-l7sztnkp.png)

3.4 the result should be like this:

![](capture-d-e-cran-2022-09-08-a--13-53-36-l7sztyqt.png)


### Create event Action 


![](capture-d-e-cran-2022-09-08-a--13-56-47-l7szvq0r.png)

Enter Counter as name and click next

![](capture-d-e-cran-2022-09-08-a--13-57-38-l7t019vz.png)

Select Event action

![](capture-d-e-cran-2022-09-08-a--13-57-50-l7t021qh.png)

Select Javascript

![](capture-d-e-cran-2022-09-08-a--13-57-56-l7t02rax.png)

Insert the code below in the box.

![](capture-d-e-cran-2022-09-08-a--13-58-00-l7t033eq.png)


Insert the following code in the section.

    importPackage(java.lang);
    importClass(Packages.com.filenet.api.engine.EventActionHandler);
    importClass(Packages.com.filenet.api.collection.GroupSet);
    importClass(Packages.com.filenet.api.util.Id);
    importPackage(Packages.com.filenet.api.events);
    importPackage(Packages.com.filenet.api.property);
    importPackage(Packages.com.filenet.api.security);
    importPackage(Packages.com.filenet.api.core);
    importPackage(Packages.com.filenet.api.constants);


    function onEvent(event, Subscription){
    for(var i =0;i<maxloop;i++) System.out.println("");
    // Paramters that my change  
    var pathConfiguration = "/config/";
    var varCounterAttribute = "CounterProperty";
    var varCounterPrefix = "CounterPrefix";
    var varCount = "Count";
    var maxloop = 50;
    var checkIfEmpty= true;
    //

    var os = event.getObjectStore();
    var id=event.get_SourceObjectId();
    //System.out.println("Document or Folder: "+ id);
    var docOrFolder = null;
    try {   
            docOrFolder = Factory.Document.fetchInstance(os, id, null);
            System.out.println("Counter: the document has been found with the following id: "+ id);
        }catch (e ) 
        {
            try {   
                System.out.println("Counter: The document with the following id "+id+" is not found, lets try if it's a folder.")
                docOrFolder = Factory.Folder.fetchInstance(os, id, null);
                System.out.println("Counter: the folder has been found with the following id: "+ id);
              }catch (e) 
                     {
                     System.out.println("Counter: The document or  with the following id "+id+" is not found, Let's stop here ");
                     System.out.println(e);
                }
        }
    if(docOrFolder != null ) {
    var className = docOrFolder.getClassName();
    var dispenser = null;
    var attributeToUpdate = null;
    var prefix = null;
    // Target property symbolic name where to store the count 
    var counterProperty1 =  new FilterElement(null,null,null, varCounterAttribute,null);
    // prefix before storing the id = like contract-2022-"Count"
    var counterProperty2 =  new FilterElement(null,null,null,varCounterPrefix,null);
    // the current counter is an integer
    var counterProperty3 =  new FilterElement(null,null,null,varCount,null);
    var propertyFilter =  new PropertyFilter();
    propertyFilter.addIncludeProperty(counterProperty1);
    propertyFilter.addIncludeProperty(counterProperty2);
    propertyFilter.addIncludeProperty(counterProperty3);
    var className = docOrFolder.getClassName();
    var count = -1;
    var docOrFolderProperties = docOrFolder.getProperties();
    var currentCount = null ;
    var proceed = false;
    dispenser = Factory.CustomObject.fetchInstance(os, pathConfiguration+className, propertyFilter);
    if(dispenser == null ) System.out.println("Counter: Dispenser Not found - in the following path : "+ pathConfiguration+className);
            else {
                var dispenserProperties = dispenser.getProperties();
                attributeToUpdate =  dispenserProperties.getStringValue(varCounterAttribute);
                prefix =dispenserProperties.getStringValue(varCounterPrefix);
                if(attributeToUpdate !=null  || attributeToUpdate.lentgh > 0 )  {    
                if(docOrFolderProperties.get(attributeToUpdate).toString().contains("PropertyInteger32Impl")) {
                     currentCount = docOrFolderProperties.getInteger32Value(attributeToUpdate);
                    if(!checkIfEmpty || currentCount == null || currentCount == 0 ) proceed =true;
                     }else if(docOrFolderProperties.get(attributeToUpdate).toString().contains("PropertyStringImpl")){
                        currentCount = docOrFolderProperties.getStringValue(attributeToUpdate);
                        if(!checkIfEmpty || currentCount == null || currentCount.trim().length == 0 ) proceed =true;
                     }    else {
                    System.out.println("Counter: Faild to update as the attribute to update is "+ attributeToUpdate+ "is not a supported format (Integer or String):"+ docOrFolderProperties.get(attributeToUpdate).toString());
                }     
                }            
                if(proceed) {
                for(var i =0;i<maxloop;i++) {
                    try {                   
                            count  = dispenserProperties.getInteger32Value(varCount);   
                            if(count == null) count = 0;                    
                            count++;
                            dispenserProperties.putValue(varCount, Integer.valueOf(count));
                            dispenser.save(RefreshMode.NO_REFRESH);
                            System.out.println("Counter: Dispenser Updated");
                            break;              

                    }catch (e ) {
                         System.out.println("Counter: Failed to save with the following issue: "+ e);
                          dispenser.fetchProperties(propertyFilter);
                          dispenserProperties = dispenser.getProperties();
                          Thread.sleep(5);
                        }
                }
                }

            }
            if(count>0 && proceed ){

                if(docOrFolderProperties.get(attributeToUpdate).toString().contains("PropertyInteger32Impl")) {
                    docOrFolderProperties.putValue(attributeToUpdate, Integer.valueOf(count));
                }else if(docOrFolderProperties.get(attributeToUpdate).toString().contains("PropertyStringImpl")){               
                      if(prefix!=null ) docOrFolderProperties.putValue(attributeToUpdate, prefix+count);
                      else  docOrFolderProperties.putValue(attributeToUpdate, count+"");
                }
                System.out.println("Counter: "+count + " is the current count");
                docOrFolder.save(RefreshMode.NO_REFRESH);
            }else{
                System.out.println("Counter: Failed with the following values: The current count is "+ count+ " and the attribute to update is "+ attributeToUpdate + " proceed is " + proceed);
            }
    }

    }
    

![](image-l7t051x1.png)


### Create Config Folder

Create folder config at the root  folder 

Click on the Root folder , Actions and new Folder

![](capture-d-e-cran-2022-09-08-a--14-04-58-l7t06yid.png)

Enter config as name

![](capture-d-e-cran-2022-09-08-a--14-05-37-l7t073y8.png)

Next

![](capture-d-e-cran-2022-09-08-a--14-05-41-l7t0777c.png)

Finish

![](capture-d-e-cran-2022-09-08-a--14-05-45-l7t079rs.png)

Tips Hide the folder config for lambda user : 

![](capture-d-e-cran-2022-09-08-a--14-08-05-l7t0azbc.png)


### Create custom object related to the target class folder or document class folder

Lets assume that I have already a document class DAC Email with one property 

![](image-l7t0dcvh.png)


1. Create an instance of the custom class counter to configure the counter for the DAC Email Class with the sympbolic name DACEmail

In the folder config created in the previous step, create a custom class instance of Counter.

![](capture-d-e-cran-2022-09-08-a--14-11-54-l7t0hk5t.png)

Enter the Containement name as the Sympbolic Name and select the class as Counter

![](capture-d-e-cran-2022-09-08-a--14-15-33-l7t0l1dz.png)

Complete the values: 

- CounterProperty is the property in teh Dac Email class where to store the value
- CounterPrefix is the on optional static prefix when storing the value
- CounterName is the Sympbolic Name of Dac Email
- Count is the current count should be at 0 by default.
 
![](capture-d-e-cran-2022-09-08-a--14-19-21-l7t0pcvy.png)

Click Next

![](capture-d-e-cran-2022-09-08-a--14-19-27-l7t0pkjs.png)

Click Finish

![](capture-d-e-cran-2022-09-08-a--14-19-38-l7t0pn5m.png)


### Create Subscription 

Create Subscription to trigger a counter for the DAC Email Class 

Select DAC Email Class and click create new subscription

![](image-l7t0wkxl.png)

Complete the name

![](capture-d-e-cran-2022-09-08-a--14-26-19-l7t0yhfn.png)

Select the scope

![](capture-d-e-cran-2022-09-08-a--14-26-25-l7t0yk8t.png)

Select on which event the counter is trigger, in my case the creation

![](capture-d-e-cran-2022-09-08-a--14-26-31-l7t0zed0.png)

Select the event action created previously

![](capture-d-e-cran-2022-09-08-a--14-26-44-l7t0zlak.png) 

Click next

![](capture-d-e-cran-2022-09-08-a--14-26-49-l7t0zlal.png)

Click Finish

![](capture-d-e-cran-2022-09-08-a--14-26-54-l7t0zlam.png)

### Test it
