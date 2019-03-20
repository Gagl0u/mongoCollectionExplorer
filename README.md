## MAP objects from MONGODB

1. First you need to add the script to your mongo instance.

Open a new shell in your mongo and copy paste code below.

------------ BEGINNING
```

db.system.js.save({"_id": "exploreJson",
value: function (collectionName, maxLimit) {
    // collectionName must be name of collection to map
    // maxLimit is number of objects to retrieve

    var id = null;

    var keyInObject = function (key, object) {
        return (key in object);
    }

    var createTargetKeyObject = function (keyName, targetObject, isArray) {
        if (!keyInObject(keyName, targetObject)) {
            var base = isArray ? [{}] : {};
            targetObject[keyName] = base;
        }
        return isArray ? targetObject[keyName][0] : targetObject[keyName];
    }

    function handleFinalKey(keyName, sourceKeyObject, targetObject) {
        if (!keyInObject(keyName, targetObject)) {
            targetObject[keyName] = sourceKeyObject;
        }
    }

    function handleArrayObject(keyName, sourceKeyObject, targetObject) {
        var targetKeyObject = createTargetKeyObject(keyName, targetObject, true);
        for (var z = 0; z < sourceKeyObject.length; z++) {
            var indexSourceObject = sourceKeyObject[z];
            if (indexSourceObject instanceof Object) {
                exploreKeys(indexSourceObject, targetKeyObject);
            } else {
                targetObject[keyName] = [indexSourceObject];
            }
        }
    }

    function handleObject(keyName, sourceKeyObject, targetObject) {
        if (sourceKeyObject instanceof NumberLong) {
            targetObject[keyName] = sourceKeyObject;
        } else {
            var targetKeyObject = createTargetKeyObject(keyName, targetObject);
            exploreKeys(sourceKeyObject, targetKeyObject);
        }
    }

    function exploreKeys(sourceObject, targetObject) {
        var keys = Object.keys(sourceObject);
        for (var i = 0; i < keys.length; i++) {
            var keyName = keys[i];
            var sourceKeyObject = sourceObject[keyName];
            if (sourceKeyObject != null) {
                if (sourceKeyObject instanceof Array) {
                    handleArrayObject(keyName, sourceKeyObject, targetObject);
                } else if (sourceKeyObject instanceof Object) {
                    handleObject(keyName, sourceKeyObject, targetObject)
                } else {
                    handleFinalKey(keyName, sourceKeyObject, targetObject)
                }
            }
        }
    }

    var finalObject = {};
    var collectionCursor = db.getCollection(collectionName).find({}).limit(maxLimit).snapshot();

    while (collectionCursor.hasNext()) {
        var order = collectionCursor.next();
        id = order._id;
        exploreKeys(order, finalObject);
    }
    printjson(finalObject);
}
});
```
------------ END

You should now find a new entry in your database. To look for it open `System` folder and view `system.js` collection.



2. Now you need to run the script

Open a new shell (or you can use an existing one) and copy paste the code below

------------ BEGINNING
```
db.loadServerScripts()
exploreJson("COLLECTION_NAME", "MAX_NUMBER_OF_OBJECTS_TO_RETRIEVE")
```
------------ END

3. Update the values used in the script

You need to change the "COLLECTION_NAME" with the actual collection name as a string
You need to change "MAX_NUMBER_OF_OBJECTS_TO_RETRIEVE" with an integer from 0 (0 meaning all the result). The use of this limit is for GUI uses ase timeout can cause errors thus preventing the result from being displayed.


4. Run the script and wait for the result to be printed.

The result will be printed as a Json string once the script finishes running.
