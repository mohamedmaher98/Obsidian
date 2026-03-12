# Java Collections

## What is a Collection?

Collections are containers that support various ways of storing and
organizing objects in order to access them efficiently.

They are implementations of abstract data structures that support three
basic operations:

-   Adding a new element to the collection
-   Removing an element from the collection
-   Changing an element in the collection

------------------------------------------------------------------------

# Main Interfaces of Collections and Their Implementations

Three interfaces extend `Collection`:

-   **List**
-   **Set**
-   **Queue**

------------------------------------------------------------------------

## List

Stores **ordered elements** and **can contain duplicates**.

Main implementations:

-   `ArrayList`
-   `LinkedList`
-   `Vector`

### Notes

-   **Vector**
    -   Synchronized
    -   Slower in single-threaded environments
-   **ArrayList**
    -   Faster when navigating the collection
    -   Good for reading data
-   **LinkedList**
    -   Better for inserting and deleting elements

------------------------------------------------------------------------

## Set

A collection that **does not allow duplicate elements**.

Main implementations:

-   `HashSet`
-   `TreeSet`
-   `LinkedHashSet`

### Types

-   **TreeSet**
    -   Stores elements in **sorted order**
-   **HashSet**
    -   Stores elements based on **hash keys**
    -   Order appears random
-   **LinkedHashSet**
    -   Maintains **insertion order**

------------------------------------------------------------------------

## Queue

An interface for implementing **queues in Java**.

Main implementations:

-   `LinkedList`
-   `PriorityQueue`

### Queue Principle

Queues follow **FIFO (First In First Out)**.

------------------------------------------------------------------------

# Map

`Map` is an interface for storing elements as **key-value pairs**.

Main implementations:

-   `HashTable`
-   `HashMap`
-   `TreeMap`
-   `LinkedHashMap`

### Types

-   **HashTable**
    -   Synchronized
    -   Deprecated
-   **HashMap**
    -   Order determined by **hash key**
-   **TreeMap**
    -   Elements stored in **sorted order**
-   **LinkedHashMap**
    -   Elements stored in **insertion order**

⚠️ **Keys in Map cannot be duplicated.**

------------------------------------------------------------------------

## Synchronizing Collections

You can synchronize collections using the `Collections` class:

``` java
Collections.synchronizedMap(myMap);
Collections.synchronizedList(myList);
```

These methods return a **synchronized wrapper** for the collection.

Since **Java 6**, there are special concurrent collections such as:

-   `CopyOnWriteArrayList`
-   `ConcurrentHashMap`

------------------------------------------------------------------------

# ArrayList vs LinkedList

The difference is in **how data is stored**.

-   **ArrayList**
    -   Uses a dynamic array
    -   Faster for reading and accessing elements by index
-   **LinkedList**
    -   Uses a doubly linked list
    -   Better for frequent insert and remove operations

------------------------------------------------------------------------

# HashMap vs HashTable

`HashMap` is very similar to `Hashtable`, but there are differences:

### HashTable

-   Methods are **synchronized**
-   Does **not allow null keys or values**

### HashMap

-   **Not synchronized**
-   Allows **one null key and multiple null values**

Because of synchronization, `Hashtable` has **lower performance**.

Instead, synchronization can be added using:

``` java
Collections.synchronizedMap(map);
Collections.synchronizedList(list);
Collections.synchronizedSet(set);
```

------------------------------------------------------------------------

# ArrayList vs Vector

-   **Vector**
    -   Synchronized
    -   Thread-safe
    -   Slower
-   **ArrayList**
    -   Not synchronized
    -   Faster

------------------------------------------------------------------------

# Why Map Is Not a Collection?

`List` and `Set` are collections of **elements**.

`Map` is a collection of **key-value pairs**.

Because of this structural difference, some `Collection` methods do not
apply to `Map`.

Example:

-   `Collection.remove(Object o)` removes an element.
-   `Map.remove(Object key)` removes an element **by key**.

Therefore, `Map` is **not part of the Collection interface hierarchy**.

------------------------------------------------------------------------

# What is an Iterator?

An **Iterator** is an object used to **iterate over elements in a
collection**.

The `Collection` interface provides:

``` java
Iterator<E> iterator();
```

This method returns an object implementing the `Iterator` interface.

`foreach` internally uses an iterator.

------------------------------------------------------------------------

# How to Get All Keys in a Map?

Use:

``` java
map.keySet();
```

Returns:

    Set<K>

------------------------------------------------------------------------

# How to Get All Values in a Map?

Use:

``` java
map.values();
```

Returns:

    Collection<V>

------------------------------------------------------------------------

# How to Get All Key-Value Pairs in a Map?

Use:

``` java
map.entrySet();
```

Returns:

    Set<Map.Entry<K, V>>
