<h1>ExpNo 1 :Developing AI Agent with PEAS Description</h1>
<h3>Name: Saravanan N</h3>
<h3>Register Number/Staff Id: TSML006</h3>


<h3>AIM:</h3>
<br>
<p>To find the PEAS description for the given AI problem and develop an AI agent.</p>
<br>
<h3>Theory</h3>
<h3>Medicine prescribing agent:</h3>
<p>Such this agent prescribes medicine for fever (greater than 98.5 degrees) which we consider here as unhealthy, by the user temperature input, and another environment is rooms in the hospital (two rooms). This agent has to consider two factors one is room location and an unhealthy patient in a random room, the agent has to move from one room to another to check and treat the unhealthy person. The performance of the agent is calculated by incrementing performance and each time after treating in one room again it has to check another room so that the movement causes the agent to reduce its performance. Hence, agents prescribe medicine to unhealthy.</p>
<hr>
<h3>PEAS DESCRIPTION:</h3>
<table>
  <tr>
    <td><strong>Agent Type</strong></td>
    <td><strong>Performance</strong></td>
     <td><strong>Environment</strong></td>
    <td><strong>Actuators</strong></td>
    <td><strong>Sensors</strong></td>
  </tr>
    <tr>
    <td><strong>Medicine prescribing agent</strong></td>
    <td><strong>Treating unhealthy, agent movement</strong></td>
     <td><strong>Rooms, Patient</strong></td>
    <td><strong>Medicine, Treatment</strong></td>
    <td><strong>Location, Temperature of patient</strong></td>
  </tr>
</table>
<hr>
<H3>DESIGN STEPS</H3>
<h3>STEP 1:Identifying the input:</h3>
<p>Temperature from patients, Location.</p>
<h3>STEP 2:Identifying the output:</h3>
<p>Prescribe medicine if the patient in a random has a fever.</p>
<h3>STEP 3:Developing the PEAS description:</h3>
<p>PEAS description is developed by the performance, environment, actuators, and sensors in an agent.</p>
<h3>STEP 4:Implementing the AI agent:</h3>
<p>Treat unhealthy patients in each room. And check for the unhealthy patients in random room</p>
<h3>STEP 5:</h3>
<p>Measure the performance parameters: For each treatment performance incremented, for each movement performance decremented</p>
<h3>Program:</h3>

```python
import random
import time

# --- Base Classes ---
class Thing:
    def is_alive(self):
        return hasattr(self, "alive") and self.alive

    def show_state(self):
        print("I don't know how to show_state.")


class Agent(Thing):
    def __init__(self, program=None):
        self.alive = True
        self.performance = 0
        self.program = program

    def can_grab(self, thing):
        return False


def TableDrivenAgentProgram(table):
    percepts = []

    def program(percept):
        percepts.append(percept)
        return table.get(tuple(percepts))
    return program


# --- Environment & Locations ---
room_A, room_B = (0, 0), (1, 0)

def TableDrivenDoctorAgent():
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


class Environment:
    def __init__(self):
        self.things = []
        self.agents = []

    def percept(self, agent):
        raise NotImplementedError

    def execute_action(self, agent, action):
        raise NotImplementedError

    def default_location(self, thing):
        return None

    def is_done(self):
        return not any(agent.is_alive() for agent in self.agents)

    def step(self):
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
        for step in range(steps):
            if self.is_done():
                return
            self.step()

    def add_thing(self, thing, location=None):
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
        try:
            self.things.remove(thing)
        except ValueError as e:
            print(e)
        if thing in self.agents:
            self.agents.remove(thing)


# --- Doctor Environment ---
class TrivialDoctorEnvironment(Environment):
    def __init__(self):
        super().__init__()
        self.status = {
            room_A: random.choice(["healthy", "unhealthy"]),
            room_B: random.choice(["healthy", "unhealthy"]),
        }

    def percept(self, agent):
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        if action == "Right":
            agent.location = room_B
            agent.performance -= 1
        elif action == "Left":
            agent.location = room_A
            agent.performance -= 1
        elif action == "treat":
            try:
                temp = float(input("Enter patient's temperature in °F: "))
                if temp > 98.5:
                    print("Diagnosis: Unhealthy (Fever)")
                    print("Prescribed: Paracetamol and low-dose antibiotic.")
                    agent.performance += 10
                else:
                    print("Diagnosis: Healthy")
                self.status[agent.location] = "healthy"
            except ValueError:
                print("Invalid input. Skipping treatment.")

    def default_location(self, thing):
        return random.choice([room_A, room_B])


# --- Main Execution ---
if __name__ == "__main__":
    agent = TableDrivenDoctorAgent()
    environment = TrivialDoctorEnvironment()
    environment.add_thing(agent)

    print("\n\t Status of rooms before treatment")
    print(environment.status)
    print(f"Agent Location : {agent.location}")
    print(f"Performance : {agent.performance}")
    time.sleep(2)

    for step in range(2):
        print(f"\n--- Step {step+1} ---")
        environment.run(steps=1)
        print("\n\t Status after treatment")
        print(environment.status)
        print(f"Agent Location : {agent.location}")
        print(f"Performance : {agent.performance}")
        time.sleep(2)
```
<h3>OUTPUT:</h3>

![image](https://github.com/user-attachments/assets/7e0dc1be-c818-4a35-9169-0c80c41dec03)


<h3>Result:</h3>
Thus, an AI agent is developed.
