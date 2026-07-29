# Encapsulation

Encapsulation is to put data and methods of operating data together, hide internal implementation details, and expose only necessary interfaces to the outside world.

For example, set member variables to private, and only access and modify the state through public methods. Its purpose is to protect the invariants of the class.

---

# The core of encapsulation is invariant

The most important thing about a class is not just "what member variables it has", but what invariants it maintains:
- `std::vector` → pointer / size / capacity must be consistent
- `std::unique_ptr` → There is only one owner at the same time
- `MutexGuard` → The lock must be held while the object is alive
- `Fraction` → The denominator cannot be 0, and may have to remain in the reduced canonical form

This means:

- The responsibility of the constructor is to create an invariant
- The responsibility of member functions is to maintain invariant
- The destructor's responsibility is to end the life cycle safely
- The responsibility of API design is to prevent users from easily destroying invariants

---

# The essence of encapsulation is to protect invariant

Encapsulation does not mean making everything private, but only exposing those operations that do not break the class invariant.

- Users can only modify state in a controlled manner
- Classes can centrally check rules
- invariant is not easily broken by external code

So **private is just a means, and protecting invariant is the purpose**.