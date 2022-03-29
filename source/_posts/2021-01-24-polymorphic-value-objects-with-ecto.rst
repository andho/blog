
- Creating a table structure is too finicky and hard to maintain.
- Trying wrangle embedded schemas.
- 
- Creatind an ecto type that will save the structure as a map.
  - Now it's time to design the structure.
    - First I went with a Union Type struct as explained in this article. But
      the conversion code in the Ecto type ended being too cryptic, like if I
      looked at it after a few months I wouldn't recognize what the hell is
      happening.
    - So I ended up using completely different structs to represent the
      variants and letting the Ecto type handle the conversion back and forth.

- why not use mongo db or any other non structured db. Actually there is no
  reason not to as I probably wouldn't couple this very tightly with any other
  modules.
