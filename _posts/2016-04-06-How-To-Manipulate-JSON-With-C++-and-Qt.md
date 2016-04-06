---
layout: post
title: How To Manipulate JSON With C++ and Qt
date: 16-04-06 04:03:17 -0800
categories: 
---
# How To Use JSON in C++ With Qt

## Load An Object From a JSON File and Print Its Members

First, we are going to load a simple JSON object and access the data saved in each of its fields.

I start with a simple JSON object file:

```
{
    "name": "Thor",
    "str": 34,
    "enemy": "Loki"
}
```

Here we have a nice, flat object, with some mixed data.

First step is to load the file and convert its contents into a `QbyteArray`.

```
QFile file_obj(file_path);
if(!file_obj.open(QIODevice::ReadOnly)){
    qDebug()<<"Failed to open "<<file_path;
    exit(1);
}

QTextStream file_text(&file_obj);
QString json_string;
json_string = file_text.readAll();
file_obj.close();
QByteArray json_bytes = json_string.toLocal8Bit();
```

Next step is to load it into a `QJsonDocument` then convert that to `QJsonObject`.

```
auto json_doc=QJsonDocument::fromJSON(json_bytes);

if(json_doc.isNull()){
    qDebug()<<"Failed to create JSON doc.";
    exit(2);
}
if(!json_doc.isObject()){
    qDebug()<<"JSON is not an object.";
    exit(3);
}

QJsonObject json_obj=json_doc.object();

if(json_obj.isEmpty()){
    qDebug()>>"JSON object is empty.";
    exit(4);
}
```

And finally, we access the data:

```
QVariantMap json_map = json_obj.toVariantMap();

qDebug()<< json_map["name"].toString();
qDebug()<< json_map["str"].toInt();
qDebug()<< json_map["enemy"].toString();
```

## Load An Array From A JSON File And Print Its Members

An array JSON file isn't much different from an object.

Let's start with a simple JSON array file:

```
[
    "First thing",
    "Second thing",
    "Third thing"
]
```

After that, It's the same steps for loading into a QJsonDocument as it was with the object.

Just check for an array and cast:

```
if(!json_doc.isArray()){
    qDebug() << "JSON doc is not an array.";
    exit();
}

QJsonArray json_array = json_doc.array();

if(json_array.isEmpty()){
    qDebug() << "The array is empty";
    exit();
}

for ( i=0; i< json_array.count(); ++i){
    qDebug() << json_array.at(i).toString();
}
```

## Load An Object With Array and Object Members And Print The Data

Now we will try a deeper JSON file. This one is an object that contains an object and an array:

```
{
    "stats": {
        "str": 34,
        "int": 56,
        "con": 22
    },
    "inventory": [
        "sword",
        "shield",
        "armor"
    ]
}
```

Everything is again the same as loading up the QJsonObject above.

Once you have your `QVariantMap` from the object, you can just pull `QVariant` items out of it and cast to map or list.

```
QJsonObject root_obj = json_doc.object();
QVariantMap root_map = root_obj.toVariantMap();
QVariantMap stat_map = root_map["stats"].toMap();
QVariantList inv_list = root_map["inventory"].toList();
```

Use the `isEmpty()` method on the map and list to validate that they converted correctly.

Unlike the object from the first example, this time we are just going to iterate through all the members of our object and get the keys and values. You can access an object either way.

```
QStringList key_list = stats.keys();
for(int i=0; i < key_list.count(); ++i){
    QString key=key_list.at(i);
    int stat_val = stats[key.toLocal8Bit()].toInt(); 
    qDebug() << key << ": " << stat_val;
}
```

And finally, you can iterate through the inventory list like any other QList.

## Turn A Native Class Object Into JSON and Print It As A String

Saving a JSON object is pretty much the same as loading one, except in reverse.
You can create an empty `QJsonObject` and directly assign data to keys, and the
members are automatically created, as if they were added to a map or dictionary.


Once the object is created, we cast it to a `QJsonDocument` which provides a
method that allows us to cast as a string.
Then we save the string like any other.

```
QJsonObject json_obj;
json_obj["name"] = "Thor";
json_obj["str"] = 34;
json_obj["enemy"] = "Loki";

QJsonDocument json_doc(json_obj);
QString json_string = json_doc.toJSON();

QFile save_file(file_path);
if(!save_file.open(QIODevice::WriteOnly)){
    gDebug() << "failed to open save file");
    exit();
}
save_file.write(json_string.toLocal8Bit());
save_file.close();
```

## Turn An Array Into JSON And Print It As A String

Once you've gotten this far, saving a JSON list is easy.
Just use `QJsonArray` instead of `QJsonObject` and append any new values, just
like any other `QList`.

```
QJsonArray json_array;

json_array.append("Thor");
json_array.append("Odin");
json_array.append("Freya");
```

After that, casting to `QJsonDocument` and saving works exactly like it does
for the `QJsonObject`.

## Turn An Object With Mixed Data (array, object, simple variable) into JSON And Save As A File

Finally, we are going to put all our save JSON knowledge together and save a
multi-level JSON tree.

Start at the leaves and work back to the root when building your JSON from code, it's easier this way.
First we are going to build a `QJsonObject` with data. I tried to use a
brace-enclosed initializer list to to this, as defined in the documentation,
but I could not get it to compile. So here I do it the long way.

```
QJsonObject stats_obj;
stats_obj["name"]="Thor";
stats_obj["str"]=34;
stats_obj["enemy"]="Loki";
```

Then we are going to define a `QJsonArray` to add a list of items. This array
will use the familiar `QList` method of stream initialization.

```
QJsonArray inventory_list;
inventory_list << "sword" << "shield" << "armor";
```

We will then need out root object, which is just an empty `QJsonObject` that I
will use the `insert()` method in order to add our two JSON structures to.

```
QJsonObject root_obj;
root_obj.insert("stats",stats_obj);
root_obj.insert("inventory",inventory_list);
```

Now your JSON is built. Use the usual method to cast as a `QJsonDocument` and
save.


