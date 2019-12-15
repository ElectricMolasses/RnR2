# The Linked List

Linked lists are an incredibly versatile data structure.  The easiest structure to compare them to would be an Array.  They key differences are that Linked Lists are not indexed like Arrays are, but in exchange, you're able to dynamically resize a Linked List without re-allocating memory for the entire thing.

Many languages, like javascript, will abstract away memory management, but when you create an array, you're telling the computer, "Hey, I need this much space.", and it blocks out that place in memory to be used exclusively by your array.
If you need to increase the amount of space allocated to an array, you would have to create a new one, copy the contents of the old array into the new one, and the de-allocate the old one.

Knowing this, we can see the value a Linked List might bring to the table, as an indefinitely growing data structure.

An brief glance at the linked lists structure might look something like this.

```js
const LinkedList = {
  value: 8,
  next: null
}
```

We have an object that contains a value property, and a next property.  If we want to grow our list, we could assign the next property a new list item.

```js
LinkedList.next = {
  value: 'Second Item',
  next: null
}
```

So we have a list item that can grow indefinitely, but if we were to add many items, we would need to navigate through the next property as many times as the depth of that item.
First, we'll make our structure a little easier to navigate.  Namely, we'll keep track of the item we're currently on, so the list will sit on a value for us instead of immediately returning to the first one.

```js
const LinkedList = () => {
  let root = null;
  let tail = root;

  return {
    add: (value) => {
      if (root === null) {
        root = {
          value: value,
          next: null
        };
        tail = root;
      } else {
        tail.next = {
          value: value,
          next: null
        };
        tail = tail.next;
      }
    },
    peekFront: () => {
      return root;
    },
    peekBack: () => {
      return tail;
    }
  }
}
```

We added peekFront and peekBack immediately just to test our functions.  Effectively, calling LinkedList will act as calling a constructor for the whole structure, and it will provide us with, you guessed it, an empty list.
The first time we add to it, it will see that root is empty, assign a root 'Node' to it, and then assign tail to root.  Tail will not automatically update for the first item addition, as null was passed to it by VALUE rather than by REFERENCE.

So now we're able to add items, but only look to the front and the back.  How do we actually traverse through the list?  First, let's add a pointer to the variables inside our list, to keep track of our current position.
We'll need to make sure we assign it to root the first time we create a root node as well.

```js
const LinkedList = () => {
  let pointer = root;

  ...

  return {
    add (value) => {
      if (root === null) {
        root = {
          value: value,
          next: null
        };
        tail = root;
        pointer = root;
    }
    ...
  }
}
```

Now we'll have to add a next function, and a value function to allow the user to traverse to the next value of the list, as well as see it.  For convenience, we'll have the next function also return the next value it reaches.

```js
...
next: () => {
  if (pointer.next) pointer = pointer.next;
  else pointer = root;

  return pointer.value;
}
...
```

This will allow us to traverse through the list, returning the values it encounters as it goes, and looping back to the root node if we reach the end of the list.

```js
...
value: () => {
  return pointer.value;
}
...
```

This snippet will let us pull the value we're currently sitting on.

```js
...
  let size = 0;
...
  return {
    add: (value) => {
      if (root === null) {
        root = {
          value: value,
          next: null
        };
        tail = root;
        pointer = root;
        size++;
    },
...
    size: () => {
      return size;
    }
```

And this will let us efficiently track the size of the list, without needing to recursively scan through the entire list to count it each time.

The last core feature to add, is removing a node from the list.  We have a few options on how we can support addition and deletion, but we'll add that particular feature after an upgrade.

In order to perform the upgrade, all we need to do is create a backwards connection in addition to the forward connections we already have.  We can implement that fairly simply with the following changes:

```js
...
  return {
    add: (value) => {
      if (root === null) {
        root = {
          value: value,
          next: null,
          last: null
        };
        tail = root;
        pointer = root;
      } else {
        tail.next = {
          value: value,
          next: null,
          last: tail
        };
        tail = tail.next;
      }
    },
...
```

We'll add a new function to allow backwards traversal through the list as well, as so:

```js
...
back: () => {
    if (pointer.last) pointer = pointer.last;
    else pointer = tail;

    return pointer.value;
  }
...
```

At this point, we have a mostly functional doubly linked list.  This also allows us to add a delete function that can easily remove any element of the list, without creating an impassable breach in the middle.  We could do this with a singly linked list as well, but when we iterate or recurse through the list to find the value we're removing, we would also have to maintain a second pointer for the previous value, or search through it twice.

We can implement the delete function as shown below:

```js
...
delete: (value) => {
  let current = root;

  while (current !== null) {
    if (current.value === value) {
      if (current.last) {
        current.last.next = current.next;
      } else root = current.next;
      if (current.next) {
        current.next.last = current.last;
      } else tail = current.last;
      current = null;
      return true;
    }
    current = current.next;
  }
  return false;
}
...
```

There's quite a bit going on here.  First, we need to iterate through each item on the list until we find the one matching the value we would like to remove.  We could implement a counter and remove from index, but by nature, if we need index's we would avoid using a linked list to begin with.

Once we find that value, we reach out to the list items before and after the one we're removing, and cross the connections over, so the item before the current, becomes connected to the item after the current, and vice versa.

This effectively removes the current item from the list, but just to help the garbage collector, we set the current item to null as well.

In the case of removing the first, or the last element, we also move the root or the tail, so we don't end up returning to that item when we overflow from one side of the list to the other.

## But what is it good for?

The Linked List takes up a lot more space than an array, is slower to navigate, and can't quickly tell you whether or not a value is inside.  Sure, they have dynamic memory allocation, but quite often that problem is solved by overallocating arrays.  So here are a couple examples.

You can quickly and easily implement a queue with a linked list, that allows it to grow arbitrarily large.  By adding to the back and deleting the item you just used from the front, it just works!

The structure is the basis of dynamic trees.  If each item is a node, but instead of containing a next value, it contains a next list, you can add and remove children to any tree node, or even cut out a huge segment of nodes and connect the remaining nodes to the cut off point.

Effectively, the less you know about your data, and the more you're going to modify it, the more you may want a linked list.  Our example is not a terribly efficient Linked List, but it still maintains the advantage of flexibility over Arrays.

Hopefully this left you with a better understanding of a Linked List, and a few related data structures by comparison.