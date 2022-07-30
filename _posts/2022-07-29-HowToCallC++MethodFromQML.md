---
layout: post
title: How To Call a C++ Method From QML
date: 22-07-29 06:00:00 -08000
categories: C++, Qt, QML
---

# First Create Your C++ Class
This will be the class that has a method that gets called from QML.

- This class must be derived from QObject.
- Decorate your method with `Q_INVOKABLE` .
- Nothing special in the .cpp file.

```
class MyClass : public QObject
{
	Q_OBJECT

public: 
	explicit MyClass(QObject *parent = nullptr);

	Q_INVOKABLE void helloWorld();
	Q_INVOKABLE QString echo(QString value);
}
```

In my CPP file, I just qDebug evidence that these methods are called.

# Set the Context Property
In main, we have to tell Qt that the method will be called.

```
#include <QGuiApplication>
#include <QQmlContext>
#include <QQmlApplicationEngine>
#include "myclass.h"

int main(int argc, char *argv[])
{
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;

	QQmlContext *context = engine.rootContext();
	
    MyClass my_obj;
	conttext->setContextProperty("MyObj", &my_obj);

    engine.load(QUrl(QStringLiteral("qrc:/main.qml")));
    if (engine.rootObjects().isEmpty())
        return -1;

    return app.exec();
}

```

- The "MyObj" value will be the name of the instantiated object we use to call methods from in the QML file.
- `setContextProperty` is the bridge that joins the class with QML/JavaScript.

# Call the Method from QML

You can now call the method as if you were using a JavaScript object method.

```
import QtQuick 2.12
import QtQuick.Window 2.12
import QtQuick.Controls 2.5

Window {
    visible: true
    title: qsTr("Hello World")

    Column {
            Button{
                text : "Hello World"
                onClicked: {
                    MyObj.helloWorld();
                }
            }
            Rectangle {
                width: textToShowId.implicitWidth + 20
                height: 50
                color: "white"
                Text{
                    id : textToShowId
                    text : MyObj.echo("Rectangle text will be the return " +
	                    "value of this method.");
                }
            }
        }
}

```

