---
layout: post
title: How To Compose a Singleton in C++ Qt
date: 20-10-22 06:00:00 -0800
categories: C++, singleton, Qt, design patterns
---
![Post Banner](https://i.imgur.com/MFVpkYA.jpg)

Lately I've been preferring my configuration files for programs I write to be
[singletons.](https://en.wikipedia.org/wiki/Singleton_pattern) It's one of the
few times I think it's okay to use such a pattern. I can use the config object
all over my program without passing it explicitly - which can get messy when
*every* object wants a reference. The singleton also lets me load the config
file once when it's created, then not worry about whether or not I'm working
with an object that's had it's values loaded and initialized or not.

Normally we don't like singletons. They include code that manages the singleton,
and that violates the "Do One Thing" rule. But in this case, we would otherwise
need management code to make sure things are initialized before use *anyway*.
There's no way around it. Well. There's probably some design pattern that makes
reading the code infinitely more difficult, as they tend to do, just to work
around this simple solution and satisfy the same goons who advocate 100% test
coverage.

Let's opt for simple, so we can get on with our day.

Just recently, I was writing in C++ for work, when I realized I hadn't written
this pattern in this language before! I've been using it a bunch in C#
with great results. So now we get an article we can refer to so I don't have
to reinvent this every time.

## The Header File

Our header file is familiar if you've written a singleton before in other
languages, with just a couple minor differences.

- The private `_instance` should be a pointer.
- We don't have the luxury of built in getters. Either make the values public (yuck?)
or include an explicit getter for each.

We can't initialize the static `_instance` here. That we will take care of in
main.

```
#ifndef CONFIGURATION_H
#define CONFIGURATION_H

class Configuration
{
    static Configuration *_instance;

    // Config values go here
    int _savedValue = 0;

public:
    static Configuration* Instance();

    // Getters for all values
    int getSavedValue();

private:
    Configuration();
};

#endif // CONFIGURATION_H
```

## The Source File

Notice in the static `Instance` method, we test for that initialized value of
'nullptr'. Stay tuned, as we set that in the main file.

The rest of this is bog-standard singleton stuff.

Now notice we're violating another maxim: "Don't do any work in the constructor."
In our example, it's no problem: we're just setting values. But in real life,
those variable values are likely coming from us loading a config file and parsing it.

I'd prefer to do this after the instantiation. But in this case that happens
inside the static method `Instance`. We won't be allowed to assign to our non-static
variables. We could static all the things, maybe. We can also just drink Tabasco sauce.

We're already in dangerous territory by choosing to write a singleton. We have to
pick and choose which rules we're going to break. In this case, I'm loading the
config file in the constructor. It's a slippery slope!
We can consider ourselves outlaws.

```
#include "configuration.h"

Configuration *Configuration::Instance()
{
    if (_instance == nullptr) {
        _instance = new Configuration;
    }

    return _instance;
}

int Configuration::getSavedValue()
{
    return _savedValue;

}

Configuration::Configuration()
{
    // Load your config file here and set all the values
    _savedValue = 1;

}
```

### Main

Finally we have main. You instantiate the singleton as you would expect.
But here in main is the clunky part necessitated by C++ semantics. We have to
globally initialize `Configuration::_instance`. We can't do it globally in
the header file, because Qt's make process counts up and detects that as
duplication of the initialization wherever it's included.

So it has to be global, and it has to be here. There's a reason we've created
other languages since C++.

```
#include <QCoreApplication>

#include <QDebug>
#include "configuration.h"

Configuration *Configuration::_instance = nullptr;

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    // Get or create our singleton object
    auto cfg = Configuration::Instance();

    // Get a value from the singleton
    int cfgValue = cfg->getSavedValue();

    qDebug() << "Config value: " << cfgValue;

    return a.exec();
}
```
