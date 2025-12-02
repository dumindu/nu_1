---
title: Data Structures
slug: "data-structures"
publishDate: 2023-08-28T00:00:00+08:00
---

## Definition

> ⭐️ Data Structures is **a way of organizing and storing data efficiently.**
>
> - They define:
>   - the **relationship between the data**
>   - the **operations that can be performed on the data**
> 
> So that they can be accessed and worked with efficiently.

## Arrays

```text
    APPEND(1)                        APPEND(2)                          APPEND(3)                     
                                                                                                      
    +-----+----+----+                +-----+-----+----+                 +-----+-----+-----+           
    |  1  |    |    |                |  1  |  2  |    |                 |  1  |  2  |  3  |           
    +-----+----+----+                +-----+-----+----+                 +-----+-----+-----+           
       0    1    2    ← INDEXES         0     1     2    ← INDEXES         0     1     2    ← INDEXES  
```

Example: Row of Lockers,
- Imagine a row of **numbered** lockers in a hallway.
- Each locker number is like an **index**: 0, 1, 2, ...
- To get to locker #2, you **go straight to it**—you don’t have to open lockers 0, and 1 first.
- The number of lockers is **fixed**; to add more, you would need a new hallway (a new, bigger array) and move things over.

## Stacks

```text
    PUSH(1)               PUSH(2)               PUSH(3)               POP()                
                                                                                           
    +-------+  ← TOP      +-------+  ← TOP      +-------+  ← TOP      +-------+  ← TOP     
    |   1   |             |   2   |             |   3   |             |   2   |            
    +-------+             +-------+             +-------+             +-------+            
    |       |             |   1   |             |   2   |             |   1   |            
    +-------+             +-------+             +-------+             +-------+            
    |       |             |       |             |   1   |             |       |            
    +-------+             +-------+             +-------+             +-------+            
```

Example: Stack of Plates,
- You put a plate on top  => PUSH
- You take a plate from top => POP
- LIFO: Last In, First Out

## Queues

```text
    ENQUEUE(1)                    ENQUEUE(2)                     ENQUEUE(3)                     DEQUEUE()                
                                                                                                                         
    +-------+-----+-----+         +-------+-------+-----+        +-------+-------+-------+      +-------+-------+-----+  
    |   1   |     |     |         |   1   |   2   |     |        |   1   |   2   |   3   |      |   2   |   3   |     |  
    +-------+-----+-----+         +-------+-------+-----+        +-------+-------+-------+      +-------+-------+-----+  
```

Example: Line (Queue) at a Ticket Counter, 
- You join the line at the back => ENQUEUE 
- The person at the front is served and leaves the line => DEQUEUE
- FIFO: First In, First Out

## Linked Lists

### Singly Linked List

```text
     ADD(2)                   ADD(3)                                ADD(1)                                           
                                                                                                                    
    +-------+                +-------+     +-------+               +-------+     +-------+     +-------+            
    |   2   | --> null       |   2   | --> |   3   | --> null      |   1   | --> |   2   | --> |   3   | --> null   
    +-------+                +-------+     +-------+               +-------+     +-------+     +-------+            
        ↑                        ↑                                     ↑                                            
       head                     head                                  head                                          
```

Example: Treasure Hunt with One-Way Clues,
- Each clue sheet tells you **only where the next clue is**, never the previous one.
- To reach clue #3, you **must follow the chain**: start → clue #1 → clue #2 → clue #3.
- If you want to insert a new clue between #2 and #3, you change clue #2 so it now points to the new clue, and the new clue points to #3.

### Doubly Linked List

```text
              ADD(2)                           ADD(3)                                          ADD(1)                                           
                                                                                                                                               
             +-------+                        +-------+     +-------+                         +-------+     +-------+     +-------+            
    null <-- |   2   | --> null      null <-- |   2   | <-> |   3   | --> null      null <--  |   1   | <-> |   2   | <-> |   3   | --> null   
             +-------+                        +-------+     +-------+                         +-------+     +-------+     +-------+            
                 ↑                                ↑                                               ↑                                            
                head                             head                                            head                                          
```

Example: Music Playlist with Next/Previous,
- Each song knows the **next** song and the **previous** song.
- You can tap **“Next”** to go forward or **“Previous”** to go backward through the playlist.
- You can easily insert a new song between two songs, because each song knows both its **previous** and **next** neighbor.

## Trees

```text
   ADD(5)      ADD(3)      ADD(6)           ADD(1)             ADD(2)            ADD(8)       
                                                                                              
    (5)         (5)         (5)               (5)               (5)               (5)        
               /           /   \             /   \             /   \             /   \       
             (3)         (3)   (6)         (3)   (6)         (3)   (6)         (3)   (6)     
                                          /                 /   \             /   \     \    
                                        (1)               (1)   (2)         (1)   (2)   (8)  
```

Example: Company Organization Chart,
- The CEO is the **root**.
- Each manager reports to exactly **one** person above (their **parent**).
- Managers can have **multiple** people reporting to them (their **children**).
- There are **no cycles**—you never loop back up to the CEO in a circle.

## Heaps

```text
  INSERT(5)         INSERT(3)         INSERT(8)           INSERT(1)  
                                                                     
    (5)                (3)               (3)                 (1)     
                     /                  /   \               /   \    
                   (5)                (5)   (8)           (3)   (8)  
                                                          /          
                                                        (5)          
```

Example: Emergency Room Queue,
- Each patient has a **priority** (ex, how urgent their case is).
- The **highest priority** patient is treated first, no matter when they arrived.
- A heap keeps the **highest (or lowest)** priority element at the **top** so it can be found and removed efficiently.

## Tries/ Prefix Trees

```text
     INSERT("to")          INSERT("tea")           INSERT("ten")             INSERT("in")                 INSERT("inn")   
                                                                                                                         
        (root)                 (root)                 (root)                    (root)                       (root)      
       /                      /                      /                         /      \                     /      \     
     t                      t                      t                         t          i                 t          i   
     |                      |                      |                         |          |                 |          |   
     o                  +---+---+              +---+---+                 +---+---+      n             +---+---+      n   
    [*]                 |       |              |       |                 |       |     [*]            |       |     [*]  
                        e       o              e       o                 e       o                    e       o      |   
                        |      [*]            / \     [*]               / \     [*]                  / \     [*]     n   
                        a                    a   n                     a   n                        a   n           [*]  
                       [*]                  [*]  [*]                  [*]  [*]                     [*]  [*]              
```

Example: Auto-Complete in a Search Bar,
- As you type characters (t → te → tea), the system walks down the trie.
- It can quickly find **all words** that share the same **prefix** (ex: “te” → “tea”, “ten”).

## Graphs

```text
            +------------------------+  
            |                        |  
  (1) ---- (2) ---------+            |  
   |          \         |            |  
   |           \        |            |  
   |            v       v            |  
  (6)         (3) ---- (4)           |  
                \      ^             ^  
                 \    /              |  
                  v  /              /   
                  (5) -------------+    
```

Example: City Map with Roads,
- Intersections are **nodes**, roads are **edges**.
- You can have **multiple paths** between places and **loops** (cycles).
- Some intersections may have many roads; others only one or two.
- Unlike trees, you can often start at one place, travel around, and come **back to where you started**.

## Hash Tables

```text
    INSERT(foo, 1)               INSERT(bar, 2)                 INSERT(buzz, 3)     
                                                                                       
    +---------+--------+         +---------+--------+           +---------+--------+
    |   key   | value  |         |   key   | value  |           |   key   | value  |
    +---------+--------+         +---------+--------+           +---------+--------+
    |   foo   |   1    |         |   foo   |   1    |           |   foo   |   1    |
    +---------+--------+         +---------+--------+           +---------+--------+
                                 |   bar   |   2    |           |   bar   |   2    |
                                 +---------+--------+           +---------+--------+
                                                                |   bazz  |   3    |
                                                                +---------+--------+
```

Example: Like a Dictionary,
- The word is the **key** (ex: buzz).
- The definition is the **value** (ex: 3 for bazz).
- You look up the word to get its definition, not by scanning everything from the start.

## Matrices

```text
           1    2    3    ← COLUMNS  
         +----+----+----+            
      1  |  1 |  4 |  3 |            
         +----+----+----+            
      2  |  5 |  2 |  6 |            
         +----+----+----+            
      ↑                              
     ROWS                             
```

Example: Seating Chart in a Theater,
- Each seat is at a **row** and **column**: (row 3, seat 5).
- The whole seating layout is a **grid** of values (ex. 0 = empty, 1 = booked).
- You can refer to or update a specific seat using its **(row, column)** position.