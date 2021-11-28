---
title: 'Java-==-and-equal'
date: 2016-01-01
toc: true
categories:
 - "编程"
tags: 
  - java
  - code
--- 

```java
/**
 * samples for == and equal() 
 * @author hsiung
 *
 */
class TestObj {
	// the class for test == and equal()
}

public class EqualAndCompare {

	public static void main(String[] args) {

		TestObj obj1 = new TestObj();
		TestObj obj2 = new TestObj();
		TestObj obj3 = obj1;
		System.out.println(obj1 == obj2);// false ,
		// == Compares references, not values
		System.out.println(obj1 == obj3);// true

		System.out.println(obj1.equals(obj2));// false,
		// equal() method is derived from java.lang.Object, if not override,nor
		// in superclass,then equal behave as same as ==
		// Always remember to override hashCode if you override equals so as not
		// to "break the contract".
		// As per the API, the result returned from the hashCode() method for
		// two objects must be the same if their equals methods shows that they
		// are equivalent. The converse is not necessarily true.

		String s1 = "haha";// constant pool
		String s2 = new String("haha");// defined in ?heap
		System.out.println(s1 == s2);// false ,== Compares references, not
										// values, there is a exception for
										// static field in class, static String
										// in class == and equal both always
										// return *true*
		// for more infomation，see :
		// http://stackoverflow.com/questions/7520432/what-is-the-difference-between-vs-equals-in-java

		System.out.println(s1.equals(s2)); // true compare the
											// value

		String s3 = s2.intern();// find the same value String in constant pool
		System.out.println(s1 == s3);// true

		int i1 = 2;// primitive type has no equal() method
		Integer i3 = Integer.valueOf(2);
		System.out.println(i1 == i3);// true, i3 automatic unboxing into int;
		System.out.println(i3.equals(i1));// auto boxing into Integer

		Integer i2 = 2;
		System.out.println(i3.compareTo(i2));

	}

	/*
	 * Comparable interface, a.compareTo(b) return -1：less,0:equal,1:greater. 0
	 * should always be returned for objects when the .equals() comparisons
	 * return true. All Java classes that have a natural ordering implement this
	 * (String, Double, BigInteger, ...).
	 * 
	 *
	 * Comparator interface: is a util for compare two instance,then you can use
	 * the comparator to sort array and other things
	 * 
	 */

}
```
