---
layout: post
title: Realm Browser
image: /assets/images/posts/gradle_dependencies/realm_browser_sample.png
description: Recently realm released realm 0.86 that included Dynamic Realms. With Dynamic Realms we can have access to Realm Schema and we can also create objects and access fields dynamically with using strings (Model name and Field name respectively). These flexibility of Dynamic Realms gave me idea to create a Realm Browser for Android.
---

<img src="/assets/images/posts/realm_browser/realm_browser_sample.png" style="margin-left=auto;margin-right-auto;">

Recently realm released realm 0.86 that included Dynamic Realms. With Dynamic Realms we can have access to Realm Schema and we can also create objects and access fields dynamically with using strings (Model name and Field name respectively). These flexibility of Dynamic Realms gave me idea to create a Realm Browser for Android.

## Concept

Realm browser that can be used to see tables, structure and content. This browser can be accessed from localhost that hosted by the Android device. To achieve that I'm using NanoHTTPD library.

## NanoHTTP

NanoHTTPD is a light-weight HTTP server designed for embedding in other applications, released under a Modified BSD licence.

### NanoHTTPD sample

```
public class HelloServer extends NanoHTTPD {

    public static void main(String[] args) {
        ServerRunner.run(HelloServer.class);
    }

    public HelloServer() {
        super(8080);
    }

    @Override
    public Response serve(IHTTPSession session) {
        String msg = "<html><body><h1>Hello server</h1></body></html>";

        return newFixedLengthResponse(msg);
    }
}
```

This simple code will create a HTML page with Hello server text that can be accessed from localhost with port 8080.

## DynamicRealm

Like I said in the beginning, Dynamic Realms are flexible version of Conventional Realm. Dynamic Realms are opened using the same configuration you use for the conventional Realms, but they ignore any schema, migration, and schema version.

With Dynamic Realms you can access contents as long as you know model name and field name

```
DynamicRealmObject user = realm.createObject("User");
String name = user.getString("name");
int age = user.getString("age");
```

As you can expect with all Dynamic Realms flexibility, Dynamic Realms don't have type safety and same performance that Conventional Realms have.

## RealmBrowser

### Implementation

I will just explain the important point of the code, for the full source code you can check [my github repository](https://github.com/NikoYuwono/ToolbarPanel)

First one I'm checking if there is any parameter passed by previous session

```
Map<String, String> params = session.getParms();
String className = params.get("class_name");
String selectedView = params.get("selected_view");
```

And then check if that class name is exist and subclass of RealmObject or not.

```
Class clazz = Class.forName(className);
if (RealmObject.class.isAssignableFrom(clazz)) {
	// Show structure or content
} else {
	// Show empty view
}
```

From that classname we can do standard realm query to find all results

```
RealmResults<DynamicRealmObject> realmResults = dynamicRealm.where(simpleTableName).findAll();
```

We can get field names from RealmSchema

```
RealmObjectSchema realmObjectSchema = dynamicRealm.getSchema().get(simpleTableName);
Set<String> fieldNames = realmObjectSchema.getFieldNames();
```

Currently there is no API to get field types but I already filled an [issue](https://github.com/realm/realm-java/issues/1883) and the fix will be released in next version (0.87).   
So currently I'm using a workaround to get field types with manually checking Object types

```
private RealmFieldType checkRealmFieldType(Object obj) {
    int nativeValue = -1;
    if (obj instanceof Long || obj instanceof Integer || obj instanceof Short || obj instanceof Byte) {
        nativeValue = 0;
    } else if (obj instanceof Boolean) {
        nativeValue = 1;
    } else if (obj instanceof String) {
        nativeValue = 2;
    } else if (obj instanceof byte[] || obj instanceof ByteBuffer) {
        nativeValue = 4;
    } else if (obj == null || obj instanceof Object[][]) {
        nativeValue = 5;
    } else if (obj instanceof Mixed) {
        nativeValue = 6;
    } else if (obj instanceof Date) {
        nativeValue = 7;
    } else if (obj instanceof Float) {
        nativeValue = 9;
    } else if (obj instanceof Double) {
        nativeValue = 10;
    }
    return RealmFieldType.fromNativeValue(nativeValue);
}
```

And then get the first Object from the results using get() function which return generic Object that can be type checked

```
int index = 0;
for (String fieldName : fieldNameArray) {
    Object object = realmResults.get(0).get(fieldName);
    realmFieldTypes[index] = checkRealmFieldType(object);
    index++;
}
```

And then the last one is to get the contents we need to iterate inside the table and call getter function based on field types

```
for (int i = 0; i < tableSize; i++) {
    DynamicRealmObject dynamicRealmObject = realmResults.get(i);
    for (int j = 0; j < columnCount; j++) {
        String columnName = fieldNameArray[j];
        String value = "";
        switch (realmFieldTypes[j]) {
            case INTEGER:
                value = String.valueOf(dynamicRealmObject.getLong(columnName));
                break;
            case BOOLEAN:
                value = String.valueOf(dynamicRealmObject.getBoolean(columnName));
                break;
            case FLOAT:
                value = String.valueOf(dynamicRealmObject.getFloat(columnName));
                break;
            case DOUBLE:
                value = String.valueOf(dynamicRealmObject.getDouble(columnName));
                break;
            case DATE:
                value = dynamicRealmObject.getDate(columnName).toString();
                break;
            case STRING:
                value = dynamicRealmObject.getString(columnName);
                break;
        }
    }
}
```

### Usage

To use this library you can create an instance of RealmBrowser and then call `RealmBrowser.start();` you can also set the port if you want. Don't forget to call `RealmBrowser.stop();` after you finished.

To access RealmBrowser from your browser your Android device and PC/Mac need to be on the same network. And then you can call `realmBrowser.showServerAddress(this);` to show the address in logcat.

### Download

You can get the full source code in my [github](https://github.com/NikoYuwono/realm-browser-android) or you can get it via gradle 

```
compile 'com.nikoyuwono:realm-browser:0.1'
```

### Ending Words

This library still need a lot of improvement, I'm planning to adding search function this week and will also thinking about better approach to create the webpage. Currently the full code is available in [github](https://github.com/NikoYuwono/realm-browser-android), any feedbacks, issues or pull requests would be greatly appreciated.
