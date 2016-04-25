---
layout: post
title: How To Replace Type Code With Class in C++
date: 16-04-25 14:20:38 -0800
categories: 
---
# How To Replace Type Code With Class in C++

Martin Fowler's ["Refactoring"](http://refactoring.com/) provides the base example for replacing type codes.

It's an extremely helpful text for me to keep close by while I work, and I tend to end up feeling like I need type code a lot in my own programs.
The example in the book, however is not in C++, and as I've been working in that language lately, I've found the need to translate the ideas to this language.

As a quick recap, a *type code* is where your class is going to be creating several similar objects, but they are differentiated by the value of a member. Say you have a Color class, and each time you create a color object, you pass it a string that gives the code a clue as to what color it is: "white" or "blue" or something like that.

The problem with the string approach is that someone can pass in a color name that you haven't thought of yet, and so you don't handle it. They can pass in any string, for that matter, ensuring that you don't handle it without a bunch of validation code.

Often you see the solution as an enum statement. But again, nothing is to stop someone using your class to pass an integer in place of your constant. You again waste time writing validation code.

The real solution is to control that type value with an object. If a class or interface is expected as a parameter in our Color class constructor, then the compiler does all our validation for us, as the only valid argument is one that has been defined. Our class's business logic handles its one and only business, instead of taking on the additional job of validating.

## The Code

In Java, our type class makes use of `public static final` members. In C++, we don't have that. Instead we must structure our class using `static const`

In *Refactoring*, the example is a `Patient` class that has a `BloodType` type code parameter. We could use a string to give it a blood type, or a constant, but as explained, we would rather let the compiler validate for us, and so we will use a static object.

Contrast this C++ code to the Java code in *Refactoring*. In the header we will have this:

    class BloodGroup {
	public:
		const BloodGroup BloodGroup::O(0);
	    const BloodGroup BloodGroup::A(1);
		
		...
		
	private:
		static const BloodGroup _values[] = {O, A, B, AB}
		
		// Note the private constructor.
		BloodGroup (int code);
	}
	
And in the source file:

	const BloodGroup BloodGroup::O(0);
    const BloodGroup BloodGroup::A(1);
	const BloodGroup BloodGroup::B(2);
	// etc.
	
This gets you the same effect. Now there's no object to create, as you would just use it like the Java `static final` objects. The object gets itself instantiated automatically when you use it.

Your patient class will require a `BloodGroup` object to set its type, and you just pass in something like `BloodGroup::A` to assign the type. There's no danger of someone entering a type that doesn't exist, because it simply won't be possible.

