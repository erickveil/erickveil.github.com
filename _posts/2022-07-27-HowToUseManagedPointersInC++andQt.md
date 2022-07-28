---
layout: post
title: How To Use Managed Pointers In C++ and Qt
date: 22-07-27 06:00:00 -0800
categories: C++, Qt, pointers
---
This allows a safely shared pointer that can be used on objects that get passed around by reference like in C#. 
- Make sure you declare the objects dynamically.
- Make sure you don't call `delete` on the objects.

If used in this way, you can pass around these references and use them like pointers, and not worry about them being unmanaged because you created them in a factory, and that method is now out of scope, but now someone else is looking at that memory address to use the pointer.

# Declare as class member:

```
QSharedPointer<QString> strPtr;
```

# Define and set in initializer:

```
strPtr = QSharedPointer<QString>(new QString());
```

Notice the dynamic instantiation.

# A List of Pointers

## Declare List

```
QList< QSharedPointer<MyClass> > pList;
```

### Getter for the List:

```
QList< QSharedPointer<MyClass> > &getPList() 
{
	return pList;
}
```

Gets the list as a reference. Use dot notation to call its methods like it was a copy, but you will be affecting the actual list.

## Add a value to the list

From inside the class that owns pList:

```
auto foo = QSharedPointer<MyClass>(new MyClass());
pList.append(a);
```

## Get Values And Modify List Elements Outside the Class

```
QList listRef = container.getPList();
QSharedPointer<MyClass> ptr = listRef.at(index);
MyClass *bar = ptr.data();

// Or all in one line:
MyClass *dog = container.getPList().at(index).data();

// Now just treat it like any other pointer:

// get
qDebug() << "Bar value before modification: " << bar->getter();

// set
bar->setter("new value");
```

## You Don't Need To Go Down To The Pointer
It's much safer this way. Pass around the QSharedPointer and use it like a pointer directly.

```
QList listRef = container.getPList();
QSharedPointer<MyClass> bar = listRef.at(index);

// Or all in one line:
QSharedPointer<MyClass> dog = container.getPList().at(index);

// Now just treat *that* like any other pointer:

// get
qDebug() << "Bar value before modification: " << bar->getter();

// set
bar->setter("new value");

// call a method
bar->theMethod();
```

Now you don't need to worry about who's managing your pointer, because Qt is. No, "Now where did I put that *delete* at?"
