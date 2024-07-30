### chapter 1 hexagonal architecture ###

- You want to decouple the complexity of your software to its technology. Hexagonal architecture enables you to abstract the technolgyout.
- Is suitable for distributed paradigm more than for a monolithic application. 
- As computing resources have become more affordable, microservices and Service Oriented Architecture became more prominent. Hexagonal architecture suits it best.

#### Quick overview of Hexagonal architecture ####

- the architecture is split between different *hexagone*, each owning a different piece of concern.

- There is the Domain hexagon, the Application Hexagon and the Framework Hexagon. Each is englobed by the next.

Domain hexagon = contains the entity and value type. entity = representation of a unit; value type = values that can be taken by the entity.

Application hexagon = Contains the use cases for the entity, the input for the use case (input port) and its output (output port).

Framework hexagon = How the software is interacted with. It has two components: the drivee and the driven. the drivee are the input gateway to the solution (API, REST, Client, etc) and the driven is what is changed by the solution (DB, Documents, etc)