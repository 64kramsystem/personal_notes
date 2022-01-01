# C++

- [C++](#c)
  - [Basics](#basics)
  - [Data types](#data-types)

## Basics

Strings, and printing to stdout:

```cpp
#include <iostream>
#include <string>

using namespace std;

string name = "Saverio";
cout << name << " slaps " << name << "\n";
```

Class basics:

```cpp

class Actor {
public:
  string name;

  // Constructor, with fields initialization.
  //
  Actor(string name) : slapped_(false), name(name) {}

  virtual ~Actor() {}        // Destructor
  virtual void update() = 0; // Abstract ("pure virtual") function; makes the class abstract
private:
  bool slapped_;
};

// Subclass.
//
class Comedian : public Actor {
public:
  // Invoke superclass' constructor.
  //
  Comedian(string name) : Actor(name) {}

  void face(Actor* actor) { /* blahblah */ }

  // Overridden abstract function.
  //
  virtual void update() {
    // blahblah
  }
};
```

## Data types

- `unsigned char`: (at least) 0-255 (see https://stackoverflow.com/a/87648)
