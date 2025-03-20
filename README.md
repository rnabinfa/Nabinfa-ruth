
import time
import threading
import queue
import random

# Customer class
types = ["VIP", "Corporate", "Normal"]

class Customer:
    def __init__(self, customer_id, priority, service_time):
        self.customer_id = customer_id
        self.priority = priority  # Lower value means higher priority (0: VIP, 1: Corporate, 2: Normal)
        self.service_time = service_time
    
    def __repr__(self):
        return f"Customer-{self.customer_id}({types[self.priority]}, {self.service_time}s)"

# Agent class
class Agent:
    def __init__(self, agent_id, workload_limit):
        self.agent_id = agent_id
        self.workload_limit = workload_limit
        self.current_load = 0
        self.available = True
    
    def assign_task(self, customer):
        self.available = False
        self.current_load += 1
        print(f"Agent-{self.agent_id} serving {customer}")
        time.sleep(customer.service_time)  # Simulate service time
        self.current_load -= 1
        self.available = True
        print(f"Agent-{self.agent_id} finished serving {customer}")
    
    def __repr__(self):
        return f"Agent-{self.agent_id}(Load: {self.current_load}/{self.workload_limit}, {'Free' if self.available else 'Busy'})"

# Task Scheduler
class Scheduler:
    def __init__(self, agents):
        self.agents = agents
        self.queue = queue.PriorityQueue()
    
    def add_customer(self, customer):
        self.queue.put((customer.priority, customer))  # Priority Queue (lower number = higher priority)
    
    def assign_tasks(self):
        while not self.queue.empty():
            _, customer = self.queue.get()
            available_agent = next((agent for agent in self.agents if agent.available), None)
            if available_agent:
                threading.Thread(target=available_agent.assign_task, args=(customer,)).start()
            else:
                self.queue.put((customer.priority, customer))  # Requeue if no agent is available
                time.sleep(1)  # Wait and retry

# Real-time monitoring
def monitor_agents(agents):
    while True:
        print("\nAgent Status:")
        for agent in agents:
            print(agent)
        time.sleep(5)

# Simulation setup
agents = [Agent(i, workload_limit=5) for i in range(3)]
scheduler = Scheduler(agents)
monitor_thread = threading.Thread(target=monitor_agents, args=(agents,), daemon=True)
monitor_thread.start()

# Generate customers
for i in range(10):
    customer = Customer(i, random.randint(0, 2), random.randint(2, 6))
    scheduler.add_customer(customer)

# Start task assignment
scheduler.assign_tasks()
