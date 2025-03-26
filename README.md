# ExpNo 1: Developing AI Agent with PEAS Description

## Name: Sowmya V  
## Register Number: 212222110045  

### AIM:
To find the PEAS description for the given AI problem and develop an AI agent.

### THEORY

#### Medicine Prescribing Agent:
This agent prescribes medicine for fever (greater than 98.5 degrees) which we consider unhealthy. The agent takes the user's temperature as input and operates in a hospital environment with two rooms. The agent considers two factors:  
1. Room location  
2. Presence of an unhealthy patient in a random room  

The agent moves between rooms to check and treat unhealthy patients. Performance is measured by incrementing a score upon treatment and decrementing it upon movement.  

---
## PEAS DESCRIPTION:


| **Agent Type**                  | **Performance**                  | **Environment**    | **Actuators**         | **Sensors**                     |
|---------------------------------|--------------------------------|-------------------|----------------------|--------------------------------|
| Medicine prescribing agent      | Treating unhealthy, agent movement | Rooms, Patient    | Medicine, Treatment  | Location, Temperature of patient |

---

## DESIGN STEPS:

### Step 1: Identifying the Input
- Temperature from patients  
- Location  

### Step 2: Identifying the Output
- Prescribe medicine if the patient in a random room has a fever.  

### Step 3: Developing the PEAS Description
- PEAS description is developed by considering Performance, Environment, Actuators, and Sensors.

### Step 4: Implementing the AI Agent
- Treat unhealthy patients in each room.  
- Check for unhealthy patients in random rooms.  

### Step 5: Measuring the Performance Parameters
- Performance increases for each treatment.  
- Performance decreases for each movement.  
---
## PROGRAM:

```
import random
import time


class Thing: 
    """
    This represents any physical object that can appear in an Environment. """

    def is_alive(self):
        """Things that are 'alive' should return true."""
        return hasattr(self, "alive") and self.alive

    def show_state(self):
        """Display the agent's internal state. Subclasses should override."""
        print("I don't know how to show_state.")
```
```
class Agent(Thing):
    
    """
        An Agent is a subclass of Thing """

    def __init__(self, program=None):
        self.alive = True
        self.performance = 0 
        self.program = program

    def can_grab(self, thing):
        """Return True if this agent can grab this thing. Override for appropriate subclasses of Agent and Thing.""" 
        return False
```
```
def TableDrivenAgentProgram(table): 
    """
    [Figure 2.7]
    This agent selects an action based on the percept sequence. It is practical only for tiny domains.
    To customize it, provide as table a dictionary of all
    {percept_sequence:action} pairs. """
    percepts = []
    

    def program(percept):
        action = None
        percepts.append(percept)
        action = table.get(tuple(percepts))
        return action 
    return program
```
```
room_A, room_B = (0,0), (1,0)
```
```
def TableDrivenDoctorAgent():
    """
    Tabular approach towards hospital function. 
    """
        
    table = {
    ((room_A, "healthy"),): "Right",
    ((room_A, "unhealthy"),): "treat",
    ((room_B, "healthy"),): "Left",
    ((room_B, "unhealthy"),): "treat",
    ((room_A, "unhealthy"), (room_A, "healthy")): "Right",
    ((room_A, "healthy"), (room_B, "unhealthy")): "treat",
    ((room_B, "healthy"), (room_A, "unhealthy")): "treat",
    ((room_B, "unhealthy"), (room_B, "healthy")): "Left",
    ((room_A, "unhealthy"), (room_A, "healthy"), (room_B, "unhealthy")): "treat",
    ((room_B, "unhealthy"), (room_B, "healthy"), (room_A, "unhealthy")): "treat",
    }
    return Agent(TableDrivenAgentProgram(table))
```
```
TableDrivenDoctorAgent()
```
```
class Environment:
    """Abstract class representing an Environment. 'Real' Environment classes inherit from this. Your Environment will typically need to implement:
    percept:	Define the percept that an agent sees. execute_action:	Define the effects of executing an action.
    Also update the agent.performance slot.
    The environment keeps a list of .things and .agents (which is a subset of .things). Each agent has a .performance slot, initialized to 0.
    Each thing has a .location slot, even though some environments may not need this."""

    def __init__(self):
        self.things = [] 
        self.agents = []
        #room_A, room_B = (0,0), (1,0) # The two locations for the Doctor to treat

    def percept(self, agent):
        """Return the percept that the agent sees at this point. (Implement this.)"""
        raise NotImplementedError

    def execute_action(self, agent, action):
        """Change the world to reflect this action. (Implement this.)""" 
        raise NotImplementedError

    def default_location(self, thing):
        """Default location to place a new thing with unspecified location."""
        return None

    def is_done(self):
        """By default, we're done when we can't find a live agent.""" 
        return not any(agent.is_alive() for agent in self.agents)

    def step(self):
        """Run the environment for one time step. If the
        actions and exogenous changes are independent, this method will do. If there are interactions between them, you'll need to override this method."""
        if not self.is_done(): 
            actions = []
            for agent in self.agents:
                if agent.alive:
                    actions.append(agent.program(self.percept(agent))) 
                else:
                    actions.append("")
            for (agent, action) in zip(self.agents, actions): 
                self.execute_action(agent, action)

    def run(self, steps=1000):
        """Run the Environment for given number of time steps."""
        for step in range(steps):
            if self.is_done():
                return 
            self.step()

    def add_thing(self, thing, location=None):
        """Add a thing to the environment, setting its location. For convenience, if thing is an agent program we make a new agent for it. (Shouldn't need to override this.)"""
        if not isinstance(thing, Thing):
            thing = Agent(thing)
        if thing in self.things:
            print("Can't add the same thing twice") 
        else:
            thing.location = (location if location is not None else self.default_location(thing))
            self.things.append(thing) 
            if isinstance(thing, Agent):
                thing.performance = 0 
                self.agents.append(thing)

    def delete_thing(self, thing):
        """Remove a thing from the environment."""
        try:
            
            self.things.remove(thing) 
        except ValueError as e:
            print(e)
            print(" in Environment delete_thing")
            print(" Thing to be removed: {} at {}".format(thing, thing.location))
            print(" from list: {}".format([(thing, thing.location) for thing in self.things]))
        if thing in self.agents: 
            self.agents.remove(thing)
```
```
class TrivialDoctorEnvironment(Environment):
    """This environment has two locations, A and B. Each can be unhealthy or healthy. The agent perceives its location and the location's status. This serves as an example of how to implement a simple Environment."""

    def __init__(self):
        super().__init__()
        #room_A, room_B = (0,0), (1,0) # The two locations for the Doctor to treat
        self.status = {room_A: random.choice(["healthy", "unhealthy"]), room_B: random.choice(["healthy", "unhealthy"]),}

    def thing_classes(self):
        return [TableDrivenDocterAgent]

    def percept(self, agent):
        """Returns the agent's location, and the location status (unhealthy/healthy)."""
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        """Change agent's location and/or location's status; track performance. Score 10 for each treatment; -1 for each move."""
        if action == "Right":
            agent.location = room_B
            agent.performance -= 1
        elif action == "Left":
            agent.location = room_A
            agent.performance -= 1
        elif action == "treat":
            tem=float(input("Enter your temperature")) 
            if tem>=98.5:
                self.status[agent.location] == "unhealthy"
                print("medicine prescribed: paracetamol and anti-biotic(low dose)")
                agent.performance += 10
            else:
                self.status[agent.location] = "healthy" 
            self.status[agent.location] = "healthy"


    def default_location(self, thing):
           
        return random.choice([room_A, room_B])
```
```
if   __name__ == "__main__":
    
    agent = TableDrivenDoctorAgent() 
    environment = TrivialDoctorEnvironment() 
    #print(environment)
    environment.add_thing(agent)
    print("\tStatus of patients in rooms before treatment")
    print(environment.status)
    print("AgentLocation : {0}".format(agent.location)) 
    print("Performance : {0}".format(agent.performance))
    time.sleep(3)
    for i in range(2):
        environment.run(steps=1)
        print("\n\tStatus of patient in room after the treatment") 
        print(environment.status)
        print("AgentLocation : {0}".format(agent.location)) 
        print("Performance : {0}".format(agent.performance)) 
        time.sleep(3)
```
---
## OUTPUT:
![image](https://github.com/user-attachments/assets/913e0302-bb50-4af5-8384-e5ba4d402311)

## RESULT:
Thus, the Developing AI Agent with PEAS Description was successfully implemented using Python programming.
